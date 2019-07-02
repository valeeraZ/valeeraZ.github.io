---
title: JavaEE Debut--JavaServlet(1)
layout: post
subtitle: "JavaServlet"
date:       2019-06-30
author:     "Zhao"
header-img: "img/javaee.png"
tags: 
    - Java
---
# Introduction
Servelt is the base of JavaEE but I didn't learn that in school until my second year. In this article, I will talk about the basic technics about JavaEE. However, my way is different with other tutorial: I don't use JSP to load a page instead I use AJAX.

## Preparement
- IntelliJ IDEA or eclipse
- TomCat
- libraries: 
    - servlet-api.jar (which is given in tomcat/lib)
    - all the jars about JSON
    - mysql connector
![lib.jpg](https://i.loli.net/2019/06/30/5d18d1b1ca3f366600.jpg)
- Some basic knowledge about AJAX and Java

# Let's go: user systeme
## simple structure
![structure.jpg](https://i.loli.net/2019/07/01/5d198c4fc1bf290242.jpg)

## Part 1 : servlet

### doGet and doPost
You should use doGet() when you want to intercept on HTTP GET requests. You should use doPost() when you want to intercept on HTTP POST requests. Do not port the one to the other or vice versa which makes no utter sense.

For more information, please read [HTTP Protocol](https://developer.mozilla.org/en-US/docs/Web/HTTP).

**GET**

```Java
protected void doGet(HttpServletRequest req, HttpServletResponse resp)
```

Usually, HTTP GET requests are idempotent. I.e. you get exactly the same result everytime you execute the request (leaving authorization/authentication and the time-sensitive nature of the page —search results, last news, etc— outside consideration). We can talk about a bookmarkable request. Clicking a link, clicking a bookmark, entering raw URL in browser address bar, etc will all fire a HTTP GET request. If a Servlet is listening on the URL in question, then its doGet() method will be called. It's usually used to preprocess a request. 

**POST**

```Java
protected void doPost(HttpServletRequest req, HttpServletResponse resp)
```

HTTP POST requests are not idempotent. If the enduser has submitted a POST form on an URL beforehand, which hasn't performed a redirect, then the URL is not necessarily bookmarkable. The submitted form data is not reflected in the URL. Copypasting the URL into a new browser window/tab may not necessarily yield exactly the same result as after the form submit. Such an URL is then not bookmarkable. If a Servlet is listening on the URL in question, then its doPost() will be called. It's usually used to postprocess a request. I.e. gathering data from a submitted HTML form and doing some business stuff with it (conversion, validation, saving in DB, etc). 

### service

```Java
protected void service(HttpServletRequest req, HttpServletResponse resp)
```
If you override the service function in your servlet, the servlet will handle all the request methods 
Use it when you know what you're doing, the default implementation calls doGet()or doPost() so if you overwrite it, you won't get the other method called.

To override the service, I prefer using reflex to call the methods below which I will define.

Name | Type | Function
:-: | :-: | :-: 
doLogin | POST | get information(email and password) of login 
doRegister | POST | create an account
getEmail | GET | verify that an address email is duplicated or not

Here the code of service in `public class userServlet extends HttpServlet`

```Java
protected void service(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        request.setCharacterEncoding("utf-8");
        String methodname = request.getParameter("method");//get the name of method in request
        try {
            Class clazz = Class.forName("userServlet");//in the class of userServlet
            Method[] methods = clazz.getDeclaredMethods();//
            for (Method method : methods) {
                String name = method.getName();//search for the method
                if (name.equals(methodname)) {
                    method.invoke(clazz.newInstance(), request, response);
                    break;
                }
            }
        } catch (ClassNotFoundException | IllegalAccessException | IllegalArgumentException | InvocationTargetException | InstantiationException e) {
            e.printStackTrace();
        }
    }
```














