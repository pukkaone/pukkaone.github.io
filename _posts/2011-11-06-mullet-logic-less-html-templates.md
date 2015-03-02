---
layout: post
title: "Introducing Mullet logic-less HTML templates"
---

If you take the variables in [Mustache](http://mustache.github.com/)
{% raw %} {{ }} {% endraw %}
tags and move them into HTML attributes, you wind up with
[Mullet templates](http://pukkaone.github.com/mullet/).


## Example

Given the template:

    <ul>
      <li data-for="repos">
        <a data-href="url" data-text="description">Sample project</a>
      </li>
    </ul>

Given the data in the hash:

    {
      repos: [
        { url: "hello", description: "Hello project" },
        { url: "world", description: "World project" }
      ]
    }

Will render the output:

    <ul>
      <li>
        <a href="hello">Hello project</a>
      </li>
      <li>
        <a href="world">World project</a>
      </li>
    </ul>


## Static HTML prototype

By coding the commands in
[HTML5 custom data attributes](http://www.w3.org/TR/html5/elements.html#embedding-custom-non-visible-data-with-the-data-attributes),
the templates are still valid HTML.  The templates are clean HTML which your
HTML authoring tool and browser can display correctly.  You can use the
templates as a static HTML prototype for your user interface.  Fill the
prototype with sample content and the application will replace it with dynamic
content at run time.


## Rationale

Why write another template engine?

One goal is the ability to write templates as clean HTML.
[Amrita](http://amrita.sourceforge.jp/),
[Pure](http://beebole.com/pure/) and
[Weld](https://github.com/hij1nx/weld)
templates are pure HTML because they completely separate the commands from the
templates.  The separation is so complete that it's not obvious from looking
only at the template where the dynamic data is going to be inserted in the
template.  In Mullet templates, the presence of a special attribute indicates
dynamic content is going to be inserted into the element.

[Cambridge](http://code.google.com/p/cambridge/) and
[Thymeleaf](http://www.thymeleaf.org/) have the same idea of coding the
commands in HTML attributes, but they each implement a rich expression language,
and I want to see have far I get with an extremely simple variable syntax.


## Implementations

The project has implementations in these programming languages:

  * [Java](http://pukkaone.github.com/mullet/java.html), including Spring MVC
    integration
  * [Ruby](http://pukkaone.github.com/mullet/ruby.html), including Sinatra
    integration
