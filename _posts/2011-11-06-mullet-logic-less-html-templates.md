---
layout: post
title: "Introducing Mullet logic-less HTML templates"
---

If you take the
variables
in [Mustache](http://mustache.github.com/)
{% literal %} {{ }} {% endliteral %}
tags and move them to HTML attributes, you wind up with
[Mullet templates](http://pukkaone.github.com/mullet/).
The templates are clean HTML and can be a static HTML prototype for your user
interface.  This prototype can display sample content which the application
will replace with dynamic content at run time.


## Example

Given the template:

{% highlight html %}
<ul>
  <li data-for="repos">
    <a data-href="url" data-text="description">Sample project</a>
  </li>
</ul>
{% endhighlight %}

Given the data in the hash:

{% highlight javascript %}
{
  repos: [
    { url: "hello", description: "Hello project" },
    { url: "world", description: "World project" }
  ]
}
{% endhighlight %}

Will render the output:

{% highlight html %}
<ul>
  <li>
    <a href="hello">Hello project</a>
  </li>
  <li>
    <a href="world">World project</a>
  </li>
</ul>
{% endhighlight %}

By coding the commands in
[HTML5 custom data attributes](http://www.w3.org/TR/html5/elements.html#embedding-custom-non-visible-data-with-the-data-attributes),
the template is still valid HTML.


## Rationale

Why write another template engine?

One goal is the ability to write templates as clean HTML.
[Amrita](http://amrita.sourceforge.jp/),
[Pure](http://beebole.com/pure/) and
[Weld](https://github.com/hij1nx/weld)
templates are pure HTML because they completely separate the commands from the
templates.  The separation is so complete that it's not obvious from looking
only at the template where the dynamic data is going to be inserted in the
template.  In Mullet templates, the command in attributes point out the dynamic
content.

[Cambridge](http://code.google.com/p/cambridge/) and
[Thymeleaf](http://www.thymeleaf.org/) have the same idea of coding the
commands in HTML attributes, but they each implement a rich expression language,
and I want to see have far I get with an extremely simple variable syntax.
