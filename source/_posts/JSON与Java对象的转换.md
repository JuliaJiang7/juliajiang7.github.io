---
title: JSON与Java对象的转换
date: 2020-07-23 14:36:31
tags:
  - Java
  - json
categories: Java
visible: hide
typora-copy-images-to: ..\pictures
---

本文简要介绍了 JSON 的定义、语法规则，同时总结了两个把 JSON 字符串转为 JavaBean 的库分别是 json-lib、fastjson，另外还有 json 数据的传输与接收的简单使用。

## 1. JSON

### 1.1 什么是 JSON？

**JSON（JavaScript Object Notation，JavaScript对象表示法）：**（from Wikipedia）是一种由道格拉斯·克罗克福特构想和设计、*轻量级的数据交换语言*，该语言以易于让人阅读的文字为基础，用来传输由属性值或者序列性的值组成的数据对象。尽管JSON是JavaScript的一个子集，但JSON是*独立于语言的文本格式*，并且采用了类似于C语言家族的一些习惯。

JSON 数据格式与语言无关。即便它源自JavaScript，但当前很多编程语言都支持 JSON 格式数据的生成和解析。JSON 的官方 MIME 类型是 application/json，文件扩展名是 .json。

<!--more-->

### 1.2 JSON 语法规则

- 数据格式为 键/值 对（一个名称对应一个值）

```json
"name":"julia"
```

- 数据由逗号分隔
- 大括号保存对象（对象可以保存多个键值对）

```json
{"name":"julia", "url":"juliajiang7.github.io/"}
```

- 方括号保存数组，数组可以包含对象

```json
"sites":[
    {"name":"julia", "url":"juliajiang7.github.io"},
    {"name":"Google", "url":"www.google.com"}
]
```

详细请参考 [JSON教程](https://www.runoob.com/json/json-tutorial.html)

## 2. 使用 json-lib

http://json-lib.sourceforge.net/

JSON-lib is a java library for transforming beans, maps, collections, java arrays and XML to JSON and back again to beans and DynaBeans.
It is based on the work by Douglas Crockford in http://www.json.org/java.

### 2.1 引入 maven

```xml
<dependency>
    <groupId>net.sf.json-lib</groupId>
    <artifactId>json-lib</artifactId>
    <version>2.2.3</version>
    <classifier>jdk15</classifier>
</dependency>
```

### 2.2 json 对象转为 Java 对象

#### 2.2.1 创建实体 Student 

```java
public class Student {
    private String name;
    private Integer age;
    private Boolean isValid;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public Boolean getIsValid() {
        return isValid;
    }

    public void setIsValid(Boolean valid) {
        isValid = valid;
    }
}
```

#### 2.2.2 转换对象

```java
@Test
public void test(){
    String jsonStr = "{\"name\":\"julia\", \"age\":16, \"isValid\":true}";
    // 将json字符串转为JSONObject对象
    JSONObject jsonObject = JSONObject.fromObject(jsonStr);
    // 将JSONObject对象转为Student对象
    Student student = (Student) JSONObject.toBean(jsonObject, Student.class);
    System.out.println(student);
}
// 输出：
// Student{name='julia', age=16, isValid=true}
```

### 2.3 json 数组转为 Java 的 List\<T>

```java
@Test
public void test2(){
    String jsonStr = "[{\"name\":\"julia\", \"age\":16, \"isValid\":true}, " +
        "{\"name\":\"fan\", \"age\":17, \"isValid\":false}, " +
        "{\"name\":\"jiang\", \"age\":18, \"isValid\":true}]";
    // 将json数组转为JSONArray对象
    JSONArray jsonArray = JSONArray.fromObject(jsonStr);
    // 将JSONArray对象转为List
    List<Student> students = (List<Student>) JSONArray.toCollection(jsonArray, Student.class);
    System.out.println(students);
}
// 输出：
// [Student{name='julia', age=16, isValid=true}, Student{name='fan', age=17, isValid=false}, Student{name='jiang', age=18, isValid=true}]
```

### 2.4 json 复杂数据转为 JavaBean 对象

如果 json 对象中包含数组，这个数组中包含 json 对象，如下所示：

![image-20200514143731277](/pictures/image-20200514143731277-1595486755603.png)

要将这个 json 字符串转为 JavaBean 对象，实体类需要有 List<Student\> 属性。定义实体来 Teacher 如下：

#### 2.4.1 创建 Teacher 对象

```java
public class Teacher {
    private String name;
    private Integer age;
    List<Student> students;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public List<Student> getStudents() {
        return students;
    }

    public void setStudents(List<Student> students) {
        this.students = students;
    }
}
```

#### 2.4.2 转换对象

我们还是采用前面的方式进行转换，发现在获取 ``student.getName()`` 时报错**net.sf.ezmorph.bean.MorphDynaBean cannot be cast to** 如下所示：

```java
@Test
public void test3(){
    String jsonStr = "{\"name\":\"teacher\", \"age\":30, \"students\":" +
        "[{\"name\":\"julia\",\"age\":16,\"isValid\":true}," +
        "{\"name\":\"fan\",\"age\":17,\"isValid\":false}]}";
    // 将json数组转为JSONArray对象
    JSONObject jsonObject = JSONObject.fromObject(jsonStr);
    Teacher teacher = (Teacher) JSONObject.toBean(jsonObject, Teacher.class);
    System.out.println(teacher);
    for (Student student : teacher.getStudents()) {
        System.out.println(student.getName());
    }
}
// 输出：
/* 
Teacher{name='teacher', age=30, students=[net.sf.ezmorph.bean.MorphDynaBean@3c6aa04a[
  {isValid=true, name=julia, age=16}
], net.sf.ezmorph.bean.MorphDynaBean@2257fadf[
  {isValid=false, name=fan, age=17}
]]}

java.lang.ClassCastException: net.sf.ezmorph.bean.MorphDynaBean cannot be cast to com.juliajiang.blogtest.entity.Student
...
*/
```

这是因为：在操作 json 数据时，如果没有指明数据类型，那么只能是基本类型（比如上述Integer、Boolean等）或者String类型，不能出现复杂数据类型。

应该采用如下方式转换：

```java
@Test
public void test3(){
    String jsonStr = "{\"name\":\"teacher\", \"age\":30, \"students\":" +
        "[{\"name\":\"julia\",\"age\":16,\"isValid\":true}," +
        "{\"name\":\"fan\",\"age\":17,\"isValid\":false}]}";
    // 将json数组转为JSONArray对象
    JSONObject jsonObject = JSONObject.fromObject(jsonStr);
    Map<String, Class> map = new HashMap<>();
    map.put("students", Student.class);
    // 添加map
    Teacher teacher = (Teacher) JSONObject.toBean(jsonObject, Teacher.class, map);
    System.out.println(teacher);
    for (Student student : teacher.getStudents()) {
        System.out.println(student.getName());
    }
}
// 输出：
/*
Teacher{name='teacher', age=30, students=[Student{name='julia', age=16, isValid=true}, Student{name='fan', age=17, isValid=false}]}
julia
fan
*/
```

其中 map 对象是 Teacher 中各个属性的类型，map 的 key 是属性的名，value 是属性的类型。

## 3. 使用 fastjson

fastjson 中文WiKi：https://github.com/alibaba/fastjson/wiki/Quick-Start-CN

### 3.1 什么是 fastjson?

fastjson是阿里巴巴的开源JSON解析库，它可以解析JSON格式的字符串，支持将Java Bean序列化为JSON字符串，也可以从JSON字符串反序列化到JavaBean。

### 3.2 引入 maven 

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.28</version>
</dependency>
```

### 3.3 json 字符串与 Java 对象互转

```java
@Test
public void test4(){
    Student student = new Student("julia", 16, true);
    // 将Java对象转为json字符串
    String jsonString = JSON.toJSONString(student);
    System.out.println(jsonString);
    // 将json字符串转为Java对象
    Student student1 = JSON.parseObject(jsonString, Student.class);
    System.out.println(student1);
}
/* 输出：
{"age":16,"isValid":true,"name":"julia"}
Student{name='julia', age=16, isValid=true}
*/
```

### 3.4 json 数组转为 List\<T> 

```java
@Test
public void test(){
    String jsonStr = "[{\"name\":\"julia\", \"age\":16, \"isValid\":true}, " +
        "{\"name\":\"fan\", \"age\":17, \"isValid\":false}, " +
        "{\"name\":\"jiang\", \"age\":18, \"isValid\":true}]";
    List<Student> students = JSONArray.parseArray(jsonStr, Student.class);
    System.out.println(students);
}
/*
输出：
[Student{name='julia', age=16, isValid=true}, Student{name='fan', age=17, isValid=false}, Student{name='jiang', age=18, isValid=true}]
*/
```

### 3.5 json 复杂数据转为 Java 对象

```java 
@Test
public void test2(){
    String jsonStr = "{\"name\":\"teacher\", \"age\":30, \"students\":" +
        "[{\"name\":\"julia\",\"age\":16,\"isValid\":true}," +
        "{\"name\":\"fan\",\"age\":17,\"isValid\":false}]}";
    Teacher teacher = JSON.parseObject(jsonStr, Teacher.class);
    System.out.println(teacher);
    for (Student student : teacher.getStudents()) {
        System.out.println(student.getName());
    }
}
/*输出：
Teacher{name='teacher', age=30, students=[Student{name='julia', age=16, isValid=true}, Student{name='fan', age=17, isValid=false}]}
julia
fan
*/
```

## 4. fastjson 和 json-lib 对比

就以上三种使用途径来看，fastjson 确实更加方便。

## 5. json 数据的传输与接收

JSON 通常用于与服务端交换数据，在向服务器发送数据时一般是字符串。我们可以使用 `` JSON.stringify() `` 方法将 JavaScript 对象转换为字符串。

向服务器发送请求：

```javascript
var param = JSON.stringify(searchList);
$.post(url, {
            searchList:param
        },function (data) {
            /*...*/
        });
```

springboot 后台 controller 接收：

```java
@RequestMapping("/search")
@ResponseBody
public Map search(String searchList){
    // 将前台接收的 json 数组转化为实体SearchList的列表
    List<SearchList> lists = JSONArray.parseArray(searchList, SearchList.class);
    /*...*/
}
```

## 6. 参考文献

1. https://www.runoob.com/json/json-tutorial.html