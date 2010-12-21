---
layout: post
title: Compile JSPs On Application Startup
---

The first request to a JSP has a processing delay as the application server
converts the JSP to Java code and compiles the Java code to a class file.  You
can prevent your users from experiencing this delay by compiling the JSPs
before they are requested.

The [Tomcat manual](http://tomcat.apache.org/tomcat-6.0-doc/jasper-howto.html#Web_Application_Compilation)
describes how to compile JSPs at build time.  An Ant target runs the Japser JSP
Engine to convert JSPs to Java servlet code.  Another Ant target compiles the
Java servlet code to class files.  Servlet declarations and mappings for the
generated servlets must be merged into the `web.xml` file.  I'm going to present
a sleeker solution.

The JSP specification says a JSP container must accept the request parameter
`jsp_precompile` on a request to a JSP.  The parameter can have values `true`
or `false`, or no value at all.  If the parameter has value `true` or no value,
then it is a suggestion to the container to compile the JSP.  Even if container
chooses to ignore this suggestion, it must not let the page process the
request.

A
[custom servlet context listener](http://github.com/pukkaone/jsptools/blob/master/src/com/github/pukkaone/jsp/JspCompileListener.java)
implementation compiles all JSPs in the Web application when
the application starts.  To use it, define a listener in the `web.xml` file:

{% highlight xml %}
<listener>
  <listener-class>com.github.pukkaone.jsp.JspCompileListener</listener-class>
</listener> 
{% endhighlight %}


### Details

The `JspCompileListener` class implements the
[ServletContextListener](http://download.oracle.com/javaee/6/api/javax/servlet/ServletContextListener.html)
interface so it receives a notification when the Web application starts.

{% highlight java %}
public class JspCompileListener implements ServletContextListener {
    private ServletContext servletContext;
    private HttpServletRequest request = createHttpServletRequest();
    private HttpServletResponse response = createHttpServletResponse();
{% endhighlight %}

The `contextInitialized` method receives a notification when the application
starts.

{% highlight java %}
    public void contextInitialized(ServletContextEvent event) {
        servletContext = event.getServletContext();
        
        compileJspsInDirectory("/");
    }
{% endhighlight %}

The `compileJspsInDirectory` method finds and compiles all JSPs in a directory.
It recursively descends into subdirectories and processes JSPs found there.

{% highlight java %}
    @SuppressWarnings("unchecked")
    private void compileJspsInDirectory(String dirPath) {
        Set<String> paths = servletContext.getResourcePaths(dirPath);
        for (String path : paths) {
            if (path.endsWith(".jsp")) {
                RequestDispatcher requestDispatcher =
                        servletContext.getRequestDispatcher(path);
                if (requestDispatcher == null) {
                    // Should have gotten a RequestDispatcher for the path
                    // because the path came from the getResourcePaths() method.
                    throw new Error(path + " not found");
                }

                try {
                    servletContext.log("Compiling " + path);
                    requestDispatcher.include(request, response);
                } catch (Exception e) {
                    servletContext.log("include", e);
                }
            } else if (path.endsWith("/")) {
                // Recursively process subdirectories.
                compileJspsInDirectory(path);
            }
        }
    }
{% endhighlight %}

The RequestDispatcher sends a request to a JSP.  The
[ServletRequest](http://download.oracle.com/javaee/6/api/javax/servlet/ServletRequest.html)
argument passed to the
[RequestDispatcher.include](http://download.oracle.com/javaee/6/api/javax/servlet/RequestDispatcher.html#include(javax.servlet.ServletRequest, javax.servlet.ServletResponse)
sets the query string to `jsp_precompile`.  The ServletRequest interface defines
many methods.  Instead of writing a class providing empty implementions for
those methods, creating a JDK dynamic proxy that implements the interface is
more concise.
Another JDK dynamic proxy implements the
[ServletResponse](http://download.oracle.com/javaee/6/api/javax/servlet/ServletResponse.html)  
interface in a similar fashion.

{% highlight java %}
    private HttpServletRequest createHttpServletRequest() {
        InvocationHandler handler = new InvocationHandler() {
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                if (method.getName().equals("getQueryString")) {
                    return "jsp_precompile";
                }
                return null;
            }
        };
        
        return (HttpServletRequest) Proxy.newProxyInstance(
                getClass().getClassLoader(),
                new Class<?>[] { HttpServletRequest.class },
                handler);
    }
{% endhighlight %}
