---
layout: post
title: "ObjectParser版本1.0.0发布"
description: ""
category: 
tags: [ObjectParser]
---
{% include JB/setup %}

* TOC
{:toc}

### 概述
对于平台性业务，或者多业务对接的团队，经常碰到的一个问题是需要对接其他业务接口提供的信息，然后封装成一个标准的平台对象。而这中间很繁琐的一点就是，需要从不同业务的接口里拼接数据。

常见的业务接口有两种，一种是rest接口，返回json信息，一种是rmi接口，返回的是业务方的对象。

对于返回json的接口，常见的办法是先转成java对象，然后依次去取需要的信息，这样就的方式存在的很大的一个弊端就是，如果对方的json是一个复合对象，你就得构造相对应的复合对象，这样下来，你的业务代码里会有很多用来取字段的对象。而且，这样操作也很繁琐。

对于返回java对象的接口，你也需要一个字段一个字段去取，代码中有大量的get和set，看起来很恶心。

ObjectParser就是为了解决上面的问题而出现的。

### ObjectParser
ObjectParser是用来解析Java/Json对象的，主要有两种:

- JsonObjectParser: 负责解析Json对象
- ObjectParser: 负责解析Java对象

字段路径（fieldPath）：字段路径是ObjectParser中很重要的一个概念，它表示需要提取的字段在目标对象里的路径

这里简单拿javascript来做个对比，在javascript中，如果拿到一个json串，提取其中字段的方式比较简单，就是`var dest = JSON.parse("{\"a\":1}")`然后，通过`dest.a`就可以获取到`a`的值了，这个是比较简单的场景，对于复杂的对象，javascript的这种操作就非常方便了。

但是java是静态语言，对于这种动态取字段的方式不支持，所以这里就是ObjectParser起作用的时候了，对于json对象，ObjectParser提供了类似于javascript的操作。对于上面的例子，只需要通过`JsonObjectParser dest = new JsonObjectParser("{\"a\":1}")`来解析json对象，然后通过`dest.getInt("a")`就可以获取到`a`的值了。

#### JsonObjectParser

JsonObjectParser就是针对json对象的parser，上面已经简单讲解过。

#### ObjectParser

ObjectParser是针对于Java对象的parser，提供的是操作和上面类似，只不过是直接解析java对象。

### 使用方式
ObjectParser支持两种方式进行解析：

- Annotation解析
- Java API解析

#### Annotation解析

对于字段比较多的复杂对象，用Annotation解析比较方便。例如，假设接口返回的是一个复杂的对象`TestObj_1`，这个对象由`TestObject_In`和`TestObjComplex_origin`以及两个原生数据类型组成:
<pre><code>
public class TestObj_1 {
    private int a;
    private Long b;
    private TestObject_In in;
    private TestObjComplex_origin data;

}

public class TestObject_In {
   private int c;
}

public class TestObjComplex_origin {
    private int o_a;
    private String o_b;
}
</code></pre>

现在需要从上面的对象里提取信息，组成一个新的对象`TestObject_dst`，其中`dA` `dB` `dC` `data`字段分别来自于TestObject_In的`a` `b` `in.c`和`data`字段：
<pre><code>
//dest class
public class TestObject_dst {
    @FieldPath("a")
    private int dA;
    @FieldPath("b")
    private long dB;
    @FieldPath("in.c")
    private int dC;
    @ClassFieldPath("data")
    private TestObjComplex_dst data;
}

public class TestObjComplex_dst {
    @FieldPath("o_a")
    private int a;
    @FieldPath("o_b")
    private String b;
}
</code></pre>

通过`@FieldPath` `@ClassFieldPath`可以指定该字段需要的字段路径。对于复合对象，需要在构成复合对象的类里面同样用Annotation标注字段的路径。

对于列表对象可以通过`@ListFieldPath`来指定列表对象的路径，同时也需要指定该列表对象的元素类型，并且元素类型也需要进行字段路径标注，各个Annotation的处理是递归的。

FieldPath通过`.`来区分字段，对于列表字段，可以通过`[index]`来指定索引，例如`data.a[1].b`。

#### JavaAPI

Java API就相对简单了，对于那些只需要取很少字段，或者通过代码动态获取的方式比较合适。这种方式使用比较简单。例如：
<pre><code>
JsonObjectParser dest = new JsonObjectParser("{\"a\":1}");
int a = dest.getInt("a");
</code></pre>


### 总结

总体来说，对于java这种静态语言，ObjectParser提供了一种类似于静态语言获取字段信息的方式，在很多业务场景下还是非常适用的。


[V.0.0.1](https://github.com/kenvifire/ObjectParser/releases)

[GitHub链接](https://github.com/kenvifire/ObjectParser)









