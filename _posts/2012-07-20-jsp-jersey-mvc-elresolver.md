---
layout: post
title: Read Model Data In Jersey MVC JSP Templates Without "it."
---

Jersey, the JAX-RS reference implementation, includes an
[MVC framework](https://blogs.oracle.com/sandoz/entry/mvcj)
supporting the rendering of views by JSP templates.  The application passes a
model object to the framework, where the model object implements properties
providing the data to render in the JSP template.  Jersey exposes the model
object to the JSP template as a request attribute named "`it`".  So to read a
model property, a JSP template must evaluate an EL expression reading a
property of this object, for example, `${it.propertyName}`.

In Spring MVC, you don't have to write a specific prefix at the start of an EL
expression to read model data.  The model is a key-value map, and the framework
exposes every entry of the map as a request attribute.

To avoid cluttering JSP templates with "`it.`", I wrote a
[custom ELResolver which exposes model properties as implicit objects](https://github.com/pukkaone/webappenhance/blob/master/src/main/java/com/github/pukkaone/jsp/ViewableModelELResolver.java),
allowing a JSP template to read a model property with an EL expression like
`${propertyName}`.

A
[custom servlet context listener](https://github.com/pukkaone/webappenhance/blob/master/src/main/java/com/github/pukkaone/jsp/ViewableModelELResolverListener.java)
registers the custom ELResolver when the application starts.  To use
it, define a listener in the `web.xml` file:

    <listener>
      <listener-class>com.github.pukkaone.jsp.ViewableModelELResolverListener</listener-class>
    </listener> 


### Details

The custom ELResolver gets the model object from the request attribute named
"`it`", then uses reflection to resolve a name to a property of the object.
Assuming the requested property name is _key_, the following steps are tried:

  * If the object is a
    [Map](http://docs.oracle.com/javase/7/docs/api/java/util/Map.html),
    then lookup the value from the Map using _key_ as the key.
  * If the object has a method named _key_ with non-void return type, then use
    the return value from calling the method.
  * If the object has a method named `get`_key_ with the first letter of _key_
    capitalized and non-void return type, then use the return value from
    calling the method.
  * If the object has a field named _key_, then get the field value.

Finally, if the value acquired from the previous steps is an object implementing
[Callable](http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Callable.html),
then use the return value from invoking it.
