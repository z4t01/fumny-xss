# Reflected XSS

## 1) Reflected XSS with event handlers and href attributes blocked

* Ref: https://portswigger.net/web-security/cross-site-scripting/contexts/lab-event-handlers-and-href-attributes-blocked


Note that this lab has **whitelisted tags**, but all events and anchor href attributes are blocked.

Like the first example, we are going to test a search functionality:

<img src="img/refl/1/search.png"><br><br><br>

The first step we can to is to inject different tags like:
* \<body>
* \<h1>
* \<img>
* and so on...

When the tag is not allowed, the web application responds with a *"Tag is not allowed"* error message.
By abusing this behaviour, we are able to retrieve all the whitelisted HTML tags (I suggest you to use the [PortSwigger XSS Cheatsheet page](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)).

Intruder attack results:
* Target + position:  `GET /?search=<§XXX§> `
* Whitelisted tags:
    * \<svg>
        * \<animate>
    * \<image>
    * \<a>
* All the events seem to be blacklisted

So, right now, we have to create a working payload by using the listed tags above only.

We all agree about the fact that the `<a>` element will be the basic to construct our malicious href. 

However, as explained by [mozilla developer page](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/a), it is possible to use a kind of `<a>` element into an `<svg>` one.

>>>The \<a> SVG element creates a hyperlink to other web pages, files, locations in the same page, email addresses, or any other URL. It is very similar to HTML’s \<a> element. SVG's \<a> element is a container, which means you can create a link around text (like in HTML) but also around any shape 
>>>

For example, the following code will draw a black circle which, if clicked, will display an alert popup:

```
<svg viewBox="0 0 100 100">
  <a href="javascript:alert()">
    <circle cx="50" cy="40" r="35"/>
  </a>
</svg>
```

Instead of a circle, we can simply use a [text](https://www.w3schools.com/graphics/svg_text.asp):

```
<svg viewBox="0 0 100 100">
  <a href="javascript:alert()">
    <text x=20 y=20>Click Here</text>
  </a>
</svg>
```
However, remember that the *href* attribute is blacklisted, so we are not able to use this simple payload. 
We need to use the whitelisted `<animate>` element.

The `<animate>` element provides a way to animate an attribute (like href LOL) of an element over time. The syntax is very simple:
* `<animate attributeName=TARGETATTRIBUTE values=VALUES`

If we can alter the *href* attribute, we can insert it into the vulnerable page bypassing the WAF whitelist:
* `<animate attributeName=href values=javascript:alert(1)></animate>`

So, we are now able to create our custom paylaod:

```
targetlab.com/?search=<svg><a><animate+attributeName=href+values=javascript:alert(1)+/><text+x=20+y=20>Click me</text></a>
```

By injecting it, we were able to alter the `<svg><a>` *href* attribute's value to the classic *javascript:alert(1)*.

Vulnerable page snipped code result:

```
<h1>
   <svg>
      <a>
         <animate attributeName=href values=javascript:alert(1)></animate>
         <text x=20 y=20>Click me</text>
      </a>
   </svg>
</h1>
```
By clicking the "Click me" text, the standard alert(1) popup will be arised.