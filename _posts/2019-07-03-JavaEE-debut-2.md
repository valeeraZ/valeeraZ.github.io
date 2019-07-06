---
title: JavaEE Debut--JavaServlet(2)
layout: post
subtitle: "service, DAO and bean"
date:       2019-06-30
author:     "Zhao"
header-img: "img/javaee.png"
tags: 
    - Java
---
## Part2 : Service
In fact, we can use DAO directly after getting called from servlet. But I personnally like adding a service layer between DAO and servlet. On the one hand, using a service can make the code and logic easier to understand; on the other hand, a same method DAO can be used in different services.

To better manage, here I use interface and implement the interface.

```java
public interface userService {
    public user login(user user);
    public boolean register(user user);
    public boolean findEmail(String email);
}
/*-----Implement-----*/
public class userServiceImplement implements userService {
    Connection conn;
    public userServiceImplement(Connection conn){
        this.conn = conn;//get the db conn
    }
    @Override
    public user login(user user)  {
        userDAO userDAO = new userDAOImplement(conn);//use DAO
        user userToValid = userDAO.getUser(user.getUser_email(),user.getUser_password());
        if(userToValid != null){
            return userToValid;
        }
        return null;
    }
    @Override
    public boolean register(user user){
        userDAO userDAO = new userDAOImplement(conn);//Use DAO
        user userToValid = userDAO.setUser(user.getUser_email(),user.getUser_password(),user.getUser_name());
        if(userToValid != null){
            return true;
        }
        return false;
    }
    @Override
    public boolean findEmail(String email){
        userDAO userDAO = new userDAOImplement(conn);//Use DAO
        Boolean result = userDAO.getEmail(email);
        return result;
    }
    //output an user to a JSON Object
    public static JSONObject userToJSON(String error, user user){
        JSONObject JSONobj = new JSONObject();
        JSONobj.put("errorMessage",error);
        if(error != "true"){
            JSONobj.put("userID",String.valueOf(user.getUser_id()));
            JSONobj.put("userName",user.getUser_name());
            JSONobj.put("userEmail",user.getUser_email());
        }
        return JSONobj;
    }
}
```

## Part3 : DAO
Now you have seen that I use DAO in class service. So what's DAO ?

What is DATA ACCESS OBJECT (DAO) -

> It is a *object/interface*, which is used to access data from database of data storage. 

WHY WE USE DAO:

it abstracts the retrieval of data from a data resource such as a database. The concept is to "separate a data resource's client interface from its data access mechanism."

The problem with accessing data directly is that the source of the data can change. Consider, for example, that your application is deployed in an environment that accesses an Oracle database. Then it is subsequently deployed to an environment that uses Microsoft SQL Server. If your application uses stored procedures and database-specific code (such as generating a number sequence), how do you handle that in your application? You have two options:

Rewrite your application to use SQL Server instead of Oracle (or add conditional code to handle the differences), or
Create a layer inbetween your application logic and the data access

Its in all referred as **DAO Pattern**, It consist of following:

**Data Access Object Interface** - This interface defines the standard operations to be performed on a model object(s).

**Data Access Object concrete class** -This class implements above interface. This class is responsible to get data from a datasource which can be database / xml or any other storage mechanism.

**Model Object or Value Object** - This object is simple POJO containing get/set methods to store data retrieved using DAO class.

For more information, read [Core J2EE Patterns - Data Access Object](https://www.oracle.com/technetwork/java/dataaccessobject-138824.html).

Here the code of interface of userDAO and his implementation.

```java
public interface userDAO {
    public user getUser(String user_email, String user_password);
    public user setUser(String user_email, String user_password, String user_name );
    public boolean getEmail(String user_email);
}

/*-----Implement-----*/

public class userDAOImplement implements userDAO {
    Connection conn;
    PreparedStatement ps;
    ResultSet rs;
    JSONObject data;

    /**
     * Constructor for connection to db
     */
    public userDAOImplement(Connection conn) {
        this.conn = conn;
        ps = null;
        rs = null;
        data = null;
    }


    public user getUser(String user_email, String user_password) {
        user user = new user();
        String password_encode = encode(user_password);
        //encode/decode is a method which you have to define. Simply I use Base64
        try {
            String sql = "select * from user where user_email = ? and user_password = ?";
            ps = conn.prepareStatement(sql);
            ps.setString(1, user_email);
            ps.setString(2, password_encode);
            ps.execute();
            ResultSet rs = ps.executeQuery();
            if (rs.next()) {
                user.setUser_id(rs.getInt("user_id"));
                user.setUser_email(rs.getString("user_email"));
                user.setUser_password(rs.getString("user_password"));
                user.setUser_name(rs.getString("user_name"));
                return user;
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return null;
    }
    public user setUser(String user_email, String user_password,String user_name){
        user user = new user();
        String password_encode = encode(user_password);
        try {
            String sql = "INSERT INTO user(user_email, user_password, user_name) VALUES (?,?,?)";
            ps = conn.prepareStatement(sql);
            ps.setString(1, user_email);
            ps.setString(2, password_encode);
            ps.setString(3, user_name);
            ps.executeUpdate();
            user.setUser_email(user_email);
            user.setUser_password(password_encode);
            user.setUser_name(user_name);
            return user;
        } catch (SQLException e) {
            e.printStackTrace();
        }
        return null;
    }
    public boolean getEmail(String user_email){
        try{
            String sql = "SELECT user_email FROM user WHERE user_email = ?";
            ps = conn.prepareStatement(sql);
            ps.setString(1, user_email);
            ps.execute();
            ResultSet rs = ps.executeQuery();
            if(rs.next()){
                return true;
            }
        }catch (SQLException e) {
            e.printStackTrace();
        }
        return false;
    }
}
```

## Part4 : bean
A JavaBean is a Java class that should follow the following conventions:

- It should have a no-arg constructor.
- It should be Serializable.
- It should provide methods to set and get the values of the properties, known as getter and setter methods.

Why use JavaBean?

According to Java white paper, it is a reusable software component. A bean encapsulates many objects into one object so that we can access this object from multiple places. Moreover, it provides easy maintenance.

In our project, the user bean is very simple.
```java
public class user {
    private int user_id;
    private String user_email;
    private String user_password;
    private String user_name;

    public int getUser_id() {
        return user_id;
    }

    public void setUser_id(int user_id) {
        this.user_id = user_id;
    }

    public String getUser_email() {
        return user_email;
    }

    public void setUser_email(String user_email) {
        this.user_email = user_email;
    }

    public String getUser_password() {
        return user_password;
    }

    public void setUser_password(String user_password) {
        this.user_password = user_password;
    }

    public String getUser_name() {
        return user_name;
    }

    public void setUser_name(String user_name) {
        this.user_name = user_name;
    }
}
```