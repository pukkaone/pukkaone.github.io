---
layout: post
title: Compile JSPs On Application Startup
---

The first request to a JSP typically has a processing delay as the JSP
container compiles the JSP.  Specifically, the container generates Java source
code for a servlet and compiles the source code to a class file.  You can
prevent your users from experiencing this delay by compiling the JSPs before
they are requested.

The [Tomcat manual](http://tomcat.apache.org/tomcat-6.0-doc/jasper-howto.html#Web_Application_Compilation)
describes how to compile JSPs at build time.  An Ant target runs the Japser JSP
Engine to convert JSPs to servlet code.  Another Ant target compiles the
servlet code to class files.  Servlet declarations and mappings for the
generated servlets must be merged into the `web.xml` file.  I'm going to
present another solution that compiles JSPs when the application starts.

The JSP specification says a container must accept the request parameter
`jsp_precompile` on a request to a JSP.  The parameter can have values `true`
or `false`, or no value at all.  If the parameter has value `true` or no value,
then it is a suggestion to the container to compile the JSP.  Even if container
chooses to ignore this suggestion, it must not let the page process the
request.

A
[custom servlet context listener](http://github.com/pukkaone/webappenhance/blob/master/src/com/github/pukkaone/jsp/JspCompileListener.java)
compiles all JSPs in the Web application when the application starts.  To use
it, define a listener in the `web.xml` file:

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

The call to the
[RequestDispatcher.include](http://download.oracle.com/javaee/6/api/javax/servlet/RequestDispatcher.html#include(javax.servlet.ServletRequest, javax.servlet.ServletResponse)
method sends a request to a JSP.  One of the arguments must be a
[HttpServletRequest](http://download.oracle.com/javaee/6/api/javax/servlet/http/HttpServletRequest.html)
implementation that returns `jsp_precompile` for the query string.  I could
have written a class providing stub implementations for the HttpServletRequest
methods, but the interface has so many methods that the code to create a
[JDK dynamic proxy](http://download.oracle.com/javase/6/docs/api/java/lang/reflect/Proxy.html)
implementing the interface is more concise.  Another JDK dynamic proxy
implements the
[HttpServletResponse](http://download.oracle.com/javaee/6/api/javax/servlet/http/HttpServletResponse.html)
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
