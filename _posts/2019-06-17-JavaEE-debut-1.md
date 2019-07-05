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

![structure.jpg](https://i.loli.net/2019/07/01/5d198c4fc1bf290242.jpg)

## Part 1 : servlet

### doGet and doPost
You should use doGet() when you want to intercept on HTTP GET requests. You should use doPost() when you want to intercept on HTTP POST requests. Do not port the one to the other or vice versa which makes no utter sense.

For more information, please read [HTTP Protocol](https://developer.mozilla.org/en-US/docs/Web/HTTP).

**GET**

```java
protected void doGet(HttpServletRequest req, HttpServletResponse resp)
```

Usually, HTTP GET requests are idempotent. I.e. you get exactly the same result everytime you execute the request (leaving authorization/authentication and the time-sensitive nature of the page —search results, last news, etc— outside consideration). We can talk about a bookmarkable request. Clicking a link, clicking a bookmark, entering raw URL in browser address bar, etc will all fire a HTTP GET request. If a Servlet is listening on the URL in question, then its doGet() method will be called. It's usually used to preprocess a request. 

**POST**

```java
protected void doPost(HttpServletRequest req, HttpServletResponse resp)
```

HTTP POST requests are not idempotent. If the enduser has submitted a POST form on an URL beforehand, which hasn't performed a redirect, then the URL is not necessarily bookmarkable. The submitted form data is not reflected in the URL. Copypasting the URL into a new browser window/tab may not necessarily yield exactly the same result as after the form submit. Such an URL is then not bookmarkable. If a Servlet is listening on the URL in question, then its doPost() will be called. It's usually used to postprocess a request. I.e. gathering data from a submitted HTML form and doing some business stuff with it (conversion, validation, saving in DB, etc). 

### service

```java
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

```java
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
### Methods
So now we have `service` to redirect the request to the methods which we will define below.

**Connection to data base**

Of course all the operation concerned with data will have to establish a connection with data base to get or alter data. 

A user might have serveral functions: he can register, login and modify his profile. To ease the pressure of server, we use a global data base connection. Once a servlet is created, a connection to data base will be established automatically in `context listener`  and all the CRUD operations will use this connection. 

Here the code of connection to db.
```java
@WebListener //important for his role as a listener
public class DatabaseContextListener implements ServletContextListener {
    private ServletContext context = null;
    private Connection conn = null;

    public DatabaseContextListener() {

    }
    public static Connection getConnection(){
        try {
            //com.mysql.jdbc.Driver
            Class.forName("com.mysql.jdbc.Driver");
            System.out.println("Ready for the driver JDBC.");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        Connection c = null;
        try {
            c = DriverManager.getConnection(
                    "jdbc:mysql://127.0.0.1:3306/paris2024",
                    "root", "");//config of db
        } catch (SQLException e) {
            e.printStackTrace();
        }
        System.out.println("Connected to data base.");
        return c;
    }

    //will be called when the servlet is created
    public void contextInitialized(ServletContextEvent event)  {
        this.context = event.getServletContext();
        conn = getConnection();
        // a db connection in context
        context.setAttribute("dbConn",conn);
        System.out.println("Connection initialized in Context.");
    }

    //will be called when the servlet is closed
    public void contextDestroyed(ServletContextEvent event){
        this.context = null;
        try {
            this.conn.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
        this.conn = null;
        System.out.println("Connection closed in Context.");
    }
}

```

Now we have the connection that's to say we can do the CRUD. I will pass the connection by invoking the functions in service(will be mentioned in the next article, **not** the service in this article) to DAO.

Here the code of a complete servlet.

```java
@WebServlet(name = "userServlet")
public class userServlet extends HttpServlet {
    static Connection conn ;
    private user user = new user();

    public void init(){
        conn = (Connection) getServletContext().getAttribute("dbConn");
        System.out.println("userServlet initialized.实例化.");
    }
    protected void service(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException{
        //see the last bloc
    }

    protected void doRegister(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException{
        response.setContentType("charset=utf-8");
        PrintWriter out = response.getWriter();    //获取writer方法，用于将数据返回给ajax
        userService us = new userServiceImplement(conn);//调用service
        String email = request.getParameter("email");//获取输入的用户名
        String password = request.getParameter("password");//获取输入的密码
        String name = request.getParameter("name");//获取输入的密码

        user.setUser_email(email);
        user.setUser_password(password);
        user.setUser_name(name);
        if(us.register(user)) { //成功注册
            out.print("true");//返回true，表示注册成功
            System.out.println("New user registered:"+user.getUser_name());
        }else{
            out.print("false");//返回true，表示注册成功
            System.out.println("Fail to register user");
        }
    }
}
```


















