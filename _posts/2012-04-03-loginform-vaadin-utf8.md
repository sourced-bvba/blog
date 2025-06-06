---
layout: post
category : java
title: "Vaadin’s LoginForm UTF8-encoding"
tags : [vaadin, java]
---

For those using Vaadin and have its security handled through the LoginForm class, there is something you need to know. When you’re deploying on a standard servlet container, the encoding is set by default to ISO-8859-1 in accordance to the servlet specification. Not a big deal, you would say, as you’ve set the accept-charset in the form tag to UTF-8 and as such you would be able to handle UTF-8 characters.

Well, I found out this is not the case. If you would enter I£oveVaadin as a field value in the LoginForm, it sends IÄ£oveVaadin to the backing class handling the login. Which off course would break the security. Apparently, not all browsers (I tested FireFox and Chrome) send the charset to the server, in which case the server falls back to the default specification and converting the String back to ISO-8859-1, resulting in a somewhat different value.

Luckily, the solution is simple: either convert the String to UTF-8 yourself in your handler code (easily done through a simple static method, getBytes() and converting those bytes to a new UTF-8 String) or if you’re using Spring and want a more robust solution, you can add the CharacterEncodingFilter to your web.xml, configure it for UTF-8 and map it to your Vaadin servlet. This will force UTF-8 encoding on any HTTP request to the Vaadin servlet. If you ask me, you should do this by default anyways. I know I will.

Happy coding!