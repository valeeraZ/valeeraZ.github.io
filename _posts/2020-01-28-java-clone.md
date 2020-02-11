---
title: Java面向对象-拷贝
layout: post
subtitle: 浅复制和深复制
date:       2020-01-28
author:     "Zhao"
header-img: "img/java.png"
tags: 
    - Java

---

对于新手而言，复制一个对象可能很简单：

```java
class Student {  
  private int number;  
  public int getNumber() {  
    return number;  
  }  
  public void setNumber(int number) {  
    this.number = number;  
  }        
}  
public class Test {  
  public static void main(String args[]) {    
    Student stu1 = new Student();  
    stu1.setNumber(12345);  
    Student stu2 = stu1;  
    System.out.println("学生1:" + stu1.getNumber());  
    System.out.println("学生2:" + stu2.getNumber());  
  }  
}  
```

   输出：

```
学生1:12345  
学生2:12345  
```

我们试着改变stu2实例的number字段，再打印结果看看

```java
stu2.setNumber(54321);   
System.out.println("学生1:" + stu1.getNumber());  
System.out.println("学生2:" + stu2.getNumber());  
```

```
学生1:12345  
学生2:12345  
```

为什么我改变了stu2的值，stu1也跟着改变了呢？
原因出在(stu2 = stu1) 这一句。该语句的作用是将stu1的**引用**赋值给stu2。其实，stu1和stu2在堆内存中指向的是同一个对象。

![obj-reference.png](https://i.loli.net/2020/01/29/6HZtLxDFKyEj9Pe.png)

那么，怎样才能达到复制一个对象呢？深度拷贝与浅拷贝


# 浅复制（浅克隆）

被复制对象的所有变量都含有与原来的对象相同的值，而所有的对其他对象的引用仍然指向原来的对象。

换言之，浅复制仅仅复制所考虑的对象，而不复制它所引用的对象。

```java
class Student implements Cloneable { 
	String name; 
	int age; 
	Student(String name,int age) { 
		this.name=name; this.age=age; 
	} 
	public Object clone() { 
		Object o=null; 
		try { 
			o=(Student)super.clone();
			//Object 中的clone()识别出你要复制的是哪一个对象。 
		} catch(CloneNotSupportedException e) {
    	System.out.println(e.toString()); 
		} return o; 
	} 
}     

```

# 深复制（深克隆）

被复制对象的所有变量都含有与原来的对象相同的值，除去那些引用其他对象的变量。

那些引用其他对象的变量将指向被复制过的新对象，而不再是原有的那些被引用的对象。

换言之，深复制把要复制的对象所引用的对象都复制了一遍。

## java clone 方法

- clone()

clone方法将对象复制了一份并返回给调用者。一般而言，clone() 方法满足：

① 对任何的对象x，都有x.clone() !=x//克隆对象与原对象不是同一个对象

② 对任何的对象x，都有x.clone().getClass()==x.getClass()//克隆对象与原对象的类型一样

③ 如果对象x的equals()方法定义恰当，那么x.clone().equals(x)应该成立。

- Java中对象的克隆

① 为了获取对象的一份拷贝，我们可以利用Object类的clone()方法。

② 在派生类中覆盖基类的clone()方法，并声明为public。

③ 在派生类的clone()方法中，调用super.clone()。

④ 在派生类中实现Cloneable接口。

```java
static class Body implements Cloneable{
		public Head head;
		
		public Body() {}
 
		public Body(Head head) {this.head = head;}
 
		@Override
		protected Object clone() throws CloneNotSupportedException {
			return super.clone();
		}
		
}
static class Head /*implements Cloneable*/{
  public  Face face;

  public Head() {}
  public Head(Face face){this.face = face;}

} 
public static void main(String[] args) throws CloneNotSupportedException {

  Body body = new Body(new Head());

  Body body1 = (Body) body.clone();

  System.out.println("body == body1 : " + (body == body1) );

  System.out.println("body.head == body1.head : " +  (body.head == body1.head));


}

```
在以上代码中， 有两个主要的类， 分别为Body和Face， 在Body类中， 组合了一个Face对象。当对Body对象进行clone时， 它组合的Face对象只进行浅拷贝。打印结果可以验证该结论：
body == body1 : false
body.head == body1.head : true

如果要使Body对象在clone时进行深拷贝， 那么就要在Body的clone方法中，将源对象引用的Head对象也clone一份。

```java
static class Body implements Cloneable{
  public Head head;
  public Body() {}
  public Body(Head head) {this.head = head;}

  @Override
  protected Object clone() throws CloneNotSupportedException {
    Body newBody =  (Body) super.clone();
    newBody.head = (Head) head.clone();
    return newBody;
  }

}
static class Head implements Cloneable{
  public  Face face;

  public Head() {}
  public Head(Face face){this.face = face;}
  @Override
  protected Object clone() throws CloneNotSupportedException {
    return super.clone();
  }
} 
public static void main(String[] args) throws CloneNotSupportedException {

  Body body = new Body(new Head());

  Body body1 = (Body) body.clone();

  System.out.println("body == body1 : " + (body == body1) );

  System.out.println("body.head == body1.head : " +  (body.head == body1.head));

}
```
打印结果为：
body == body1 : false
body.head == body1.head : false

由此可见， body和body1内的head引用指向了不同的Head对象， 也就是说在clone Body对象的同时， 也拷贝了它所引用的Head对象， 进行了深拷贝。


### 真的是深拷贝吗

由上一节的内容可以得出如下结论：如果想要深拷贝一个对象， 这个对象必须要实现Cloneable接口，实现clone方法，并且在clone方法内部，把该对象引用的其他对象也要clone一份 ， 这就要求这个被引用的对象必须也要实现Cloneable接口并且实现clone方法。

那么，按照上面的结论， Body类组合了Head类， 而Head类组合了Face类，要想深拷贝Body类，必须在Body类的clone方法中将Head类也要拷贝一份，但是在拷贝Head类时，默认执行的是浅拷贝，也就是说Head中组合的Face对象并不会被拷贝。验证代码如下：（这里本来只给出Face类的代码就可以了， 但是为了阅读起来具有连贯性，避免丢失上下文信息， 还是给出整个程序，整个程序也非常简短）

```java
static class Body implements Cloneable{
  public Head head;
  public Body() {}
  public Body(Head head) {this.head = head;}

  @Override
  protected Object clone() throws CloneNotSupportedException {
    Body newBody =  (Body) super.clone();
    newBody.head = (Head) head.clone();
    return newBody;
  }

}

static class Head implements Cloneable{
  public  Face face;

  public Head() {}
  public Head(Face face){this.face = face;}
  @Override
  protected Object clone() throws CloneNotSupportedException {
    return super.clone();
  }
} 

static class Face{}

public static void main(String[] args) throws CloneNotSupportedException {

  Body body = new Body(new Head(new Face()));

  Body body1 = (Body) body.clone();

  System.out.println("body == body1 : " + (body == body1) );

  System.out.println("body.head == body1.head : " +  (body.head == body1.head));

  System.out.println("body.head.face == body1.head.face : " +  (body.head.face == body1.head.face));

}
```
打印结果为：
body == body1 : false
body.head == body1.head : false
body.head.face == body1.head.face : true

![obj-reference-profond1.png](https://i.loli.net/2020/01/29/9yhVxNTJKGlkF4r.png)

那么，对Body对象来说，算是这算是深拷贝吗？其实应该算是深拷贝，因为对Body对象内所引用的其他对象（目前只有Head）都进行了拷贝，也就是说两个独立的Body对象内的head引用已经指向了独立的两个Head对象。但是，这对于两个Head对象来说，他们指向了同一个Face对象，这就说明，两个Body对象还是有一定的联系，并没有完全的独立。这应该说是一种**不彻底的深拷贝**。

### 如何进行彻底的深拷贝

对于上面的例子来说，怎样才能保证两个Body对象完全独立呢？只要在拷贝Head对象的时候，也将Face对象拷贝一份就可以了。这需要让Face类也实现Cloneable接口，实现clone方法，并且在在Head对象的clone方法中，拷贝它所引用的Face对象。修改的部分代码如下：

```java
static class Head implements Cloneable{
  public  Face face;

  public Head() {}
  public Head(Face face){this.face = face;}
  @Override
  protected Object clone() throws CloneNotSupportedException {
    //return super.clone();
    Head newHead = (Head) super.clone();
    newHead.face = (Face) this.face.clone();
    return newHead;
  }
} 

static class Face implements Cloneable{
  @Override
  protected Object clone() throws CloneNotSupportedException {
    return super.clone();
  }
}
```
再次运行上面的示例，得到的运行结果如下：
body == body1 : false
body.head == body1.head : false
body.head.face == body1.head.face : false

这说明两个Body已经完全独立了，他们间接引用的face对象已经被拷贝，也就是引用了独立的Face对象。内存结构图如下：

![obj-reference-profond2.png](https://i.loli.net/2020/01/29/HFKraU3Vq7hZvDE.png)

依此类推，如果Face对象还引用了其他的对象， 比如说Mouth，如果不经过处理，Body对象拷贝之后还是会通过一级一级的引用，引用到同一个Mouth对象。同理， 如果要让Body在引用链上完全独立， 只能显式的让Mouth对象也被拷贝。

到此，可以得到如下结论：如果在拷贝一个对象时，要想让这个拷贝的对象和源对象完全彼此独立，那么在引用链上的每一级对象都要被显式的拷贝。所以创建彻底的深拷贝是**非常麻烦**的，尤其是在引用关系非常复杂的情况下， 或者在引用链的某一级上引用了一个第三方的对象， 而这个对象没有实现clone方法， 那么在它之后的所有引用的对象都是被共享的。 举例来说，如果被Head引用的Face类是第三方库中的类，并且没有实现Cloneable接口，那么在Face之后的所有对象都会被拷贝前后的两个Body对象共同引用。

---
版权：  

[详解Java中的clone方法 -- 原型模式](https://blog.csdn.net/zhangjg_blog/article/details/18369201)  

[java 浅拷贝，深度拷贝与属性复制](https://houbb.github.io/2019/01/09/java-deep-copy)  

[Java如何完全复制一个对象](https://www.jianshu.com/p/c3fbb9ea855f)