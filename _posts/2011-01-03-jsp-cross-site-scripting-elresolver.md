---
layout: post
title: ELResolver To Prevent Cross-Site Scripting in JSPs
---

[Cross-site scripting](http://en.wikipedia.org/wiki/Cross-site_scripting) is a
computer security vulnerability enabling an attacker to inject malicious code
into a Web page that will be executed by the Web browser when other users view
the page.  If your Web application accepts data from the user and then outputs
that data unaltered in HTML, then it is vulnerable because user-controlled
data might contain executable code.

Since JSP 2.0, EL expressions can appear in the template text:

{% highlight jsp %}
<div id="greeting">Hello, ${user.name}</div>
{% endhighlight %}

Unfortunately, the JSP container does not escape expression values, so if the
expression contains user-controlled data, then cross-site scripting is
possible.  JSTL provides a couple of ways to sanitize the output.  The `c:out`
tag escapes XML characters by default:

{% highlight jsp %}
<c:out value="${user.name}"/>
{% endhighlight %}

Alternatively, the EL function `fn:escapeXml` also escapes XML characters:

{% highlight jsp %}
${fn:escapeXml(user.name)}
{% endhighlight %}

If you have a lot of JSPs, fixing them using these solutions is tedious.
You must also remember to include these solutions in any new code you write.

Starting with
JSP 2.1, a Web application can register a custom
[ELResolver](http://download.oracle.com/javaee/6/api/javax/el/ELResolver.html).
I'm going to present a custom ELResolver that escapes EL expression values,
allowing you to use EL expressions while preventing cross-site scripting.

A
[custom servlet context listener](http://github.com/pukkaone/jsptools/blob/master/src/com/github/pukkaone/jsp/EscapeXmlELResolverListener.java)
registers the custom ELResolver when the application starts.  To use
it, define a listener in the `web.xml` file:

{% highlight xml %}
<listener>
  <listener-class>com.github.pukkaone.jsp.EscapeXmlELResolverListener</listener-class>
</listener> 
{% endhighlight %}


### Details

A servlet context listener's `contextInitialized` method calls the
[JspApplicationContext.addELResolver](http://download.oracle.com/javaee/6/api/javax/servlet/jsp/JspApplicationContext.html#addELResolver(javax.el.ELResolver\))
method to register the custom ELResolver.

{% highlight java %}
    public void contextInitialized(ServletContextEvent event) {
        ServletContext servletContext = event.getServletContext(); 
        JspApplicationContext jspContext = 
                JspFactory.getDefaultFactory().getJspApplicationContext(servletContext); 
        jspContext.addELResolver(new EscapeXmlELResolver()); 
    }
{% endhighlight %}

The `addELResolver` method inserts the custom ELResolver into a chain of
standard resolvers.  When evaluating an expression, the JSP container consults
the chain of resolvers in the order:

 * ImplicitObjectELResolver
 * *ELResolvers registered by the addELResolver method.*
 * MapELResolver
 * ListELResolver
 * ArrayELResolver
 * BeanELResolver
 * ScopedAttributeELResolver

This is a problem because the custom ELResolver wants to escape the value that
results from consulting the entire chain.  In a bit of tricky programming,
the custom ELResolver saves a reference to the chain of resolvers, which is
actually implemented by a
[CompositeELResolver](http://download.oracle.com/javaee/6/api/javax/el/ELResolver.html):

{% highlight java %}
    private ELResolver originalResolver;
    
    private ELResolver getOriginalResolver(ELContext context) {
        if (originalResolver == null) {
            originalResolver = context.getELResolver();
        }
        return originalResolver;
    }
{% endhighlight %}

When asked for a value, the custom ELResolver invokes the chain of resolvers
to get the value.  The custom ELResolver is itself in the chain of resolvers,
so before invoking the chain, it sets a flag telling itself to do nothing when
its turn in the chain comes around.

{% highlight java %}
    private boolean gettingValue;

    @Override
    public Object getValue(ELContext context, Object base, Object property)
        throws NullPointerException, PropertyNotFoundException, ELException
    {
        if (gettingValue) {
            return null;
        }
        
        gettingValue = true;
        Object value =
                getOriginalResolver(context).getValue(context, base, property);
        gettingValue = false;

        if (value instanceof String) {
            value = EscapeXml.escape((String) value);
        }
        return value;
    }
{% endhighlight %}

After getting the value from the chain, if the value is a String, then the
custom ELResolver escapes the String value.