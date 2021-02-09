---
title: Jackson基本应用
date: 2021-02-09 13:33:23
tags: [Java Jackson]
---

# 一 Jackson使用示例



### 第1步：创建ObjectMapper对象。

创建ObjectMapper对象。它是一个可重复使用的对象。

```java
ObjectMapper mapper = new ObjectMapper();
```

### 第2步：反序列化JSON到对象。

从JSON对象使用readValue()方法来获取。通过JSON字符串和对象类型作为参数JSON字符串/来源。

```java
//Object to JSON Conversion
Student student = mapper.readValue(jsonString, Student.class);
```

### 第3步：序列化对象到JSON。

使用writeValueAsString()方法来获取对象的JSON字符串表示。

```java
//Object to JSON Conversion		
jsonString = mapper.writeValueAsString(student);
```

在这个例子中，我们创建一个Student类。将创建一个JSON字符串学生的详细信息，并将其反序列化到学生的对象，然后将其序列化到JSON字符串。



```java
    public static void main(String args[]) throws JsonParseException, JsonMappingException, IOException {
        // 第1步：创建ObjectMapper对象。
        // 创建ObjectMapper对象。它是一个可重复使用的对象。
        ObjectMapper mapper = new ObjectMapper();
        String jsonString = "{\"name\":\"Mahesh\", \"age\":21}";

        /** 第2步：反序列化JSON到对象。
        * 从JSON对象使用readValue()方法来获取。
        * 通过JSON字符串和对象类型作为参数JSON字符串/来源。
        * map json to student
       	*/
        Student student = mapper.readValue(jsonString, Student.class);
        System.out.println(student);

        // 第3步：序列化对象到JSON。
        // 使用writeValueAsString()方法来获取对象的JSON字符串表示。
        jsonString = mapper.writeValueAsString(student);
        System.out.println(jsonString);
    }
```



 

```java
public class Student {

    private String name;
    private int age;

    public Student() {
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String toString() {
        return "Student [ name: " + name + ", age: " + age + " ]";
    }
}
```



 

# 二 Jackson数据绑定

数据绑定API用于JSON转换和使用属性访问或使用注解POJO(普通Java对象)。以下是它的两个类型。

- **简单数据绑定** - 转换JSON，从Java Maps, Lists, Strings, Numbers, Booleans 和 null 对象。
- **完整数据绑定** - 转换JSON到任何JAVA类型。

ObjectMapper读/写JSON两种类型的数据绑定。数据绑定是最方便的方式是类似XML的JAXB解析器。

#### 简单的数据绑定

简单的数据绑定是指JSON映射到Java核心数据类型。下表列出了JSON类型和Java类型之间的关系。

 

| Sr. No. | JSON 类型         | Java 类型                    |
| ------- | ----------------- | ---------------------------- |
| 1       | object            | LinkedHashMap<String,Object> |
| 2       | array             | ArrayList<Object>            |
| 3       | string            | String                       |
| 4       | complete number   | Integer, Long or BigInteger  |
| 5       | fractional number | Double / BigDecimal          |
| 6       | true \| false     | Boolean                      |
| 7       | null              | null                         |

```
public static void main(String args[]) throws JsonGenerationException, JsonMappingException, IOException {
        ObjectMapper mapper = new ObjectMapper();

        Map<String, Object> studentDataMap = new HashMap<String, Object>();
        int[] marks = { 1, 2, 3 };

        Student student = new Student();
        student.setAge(10);
        student.setName("Mahesh");
        // JAVA Object
        studentDataMap.put("student", student);
        // JAVA String
        studentDataMap.put("name", "Mahesh Kumar");
        // JAVA Boolean
        studentDataMap.put("verified", Boolean.FALSE);
        // Array
        studentDataMap.put("marks", marks);

        mapper.writeValue(new File("student3.json"), studentDataMap);
        //result student.json
        //{ 
        //   "student":{"name":"Mahesh","age":10},
        //   "marks":[1,2,3],
        //   "verified":false,
        //   "name":"Mahesh Kumar"
        //}
        studentDataMap = mapper.readValue(new File("student3.json"), Map.class);

        System.out.println(studentDataMap.get("student"));
        System.out.println(studentDataMap.get("name"));
        System.out.println(studentDataMap.get("verified"));
        System.out.println(studentDataMap.get("marks"));
    }
```

 

#### 完整数据绑定

如最上面的例子的第2步

```
        // 第2步：反序列化JSON到对象。
        // 从JSON对象使用readValue()方法来获取。通过JSON字符串和对象类型作为参数JSON字符串/来源。
        //map json to student
        Student student = mapper.readValue(jsonString, Student.class);
        System.out.println(student);
```

 

 

# 三 Jackson树模型

**树模型准备JSON文件的内存树表示。 ObjectMapper构建JsonNode节点树。这是最灵活的方法。它类似于DOM解析器的XML。**

#### 从JSON创建树

```java
public static void main(String args[]) throws JsonProcessingException, IOException {
        ObjectMapper mapper = new ObjectMapper();
        String jsonString = "{\"name\":\"Mahesh Kumar\", \"age\":21,\"verified\":false,\"marks\": [100,90,85]}";
    
        // 从JSON创建树
        JsonNode rootNode = mapper.readTree(jsonString);

        // 使用相对路径来根节点在遍历树，并处理该数据得到的每个节点。考虑下面的代码片段遍历提供的根节点的树。
        JsonNode nameNode = rootNode.path("name");
        System.out.println("Name: " + nameNode.textValue());

        JsonNode ageNode = rootNode.path("age");
        System.out.println("Age: " + ageNode.intValue());

        JsonNode verifiedNode = rootNode.path("verified");
        System.out.println("Verified: " + (verifiedNode.booleanValue() ? "Yes" : "No"));

        JsonNode marksNode = rootNode.path("marks");
        Iterator<JsonNode> iterator = marksNode.elements();
        System.out.print("Marks: [ ");
        while (iterator.hasNext()) {
            JsonNode marks = iterator.next();
            System.out.print(marks.intValue() + " ");
        }
        System.out.println("]");
    }
```

 

#### 树到Java对象转换



```java
    public static void main(String args[]) throws JsonGenerationException, JsonMappingException, IOException {
        ObjectMapper mapper = new ObjectMapper();

        String jsonString = "{\"name\":\"Mahesh\", \"age\":21}";
        // 从JSON创建树
        JsonNode rootNode = mapper.readTree(jsonString);

        Student student = mapper.treeToValue(rootNode, Student.class);
        System.out.println(student);

    }
```

 

# 四 Jackson流式API

流式API读取和写入JSON内容离散事件。 JsonParser读取数据，而JsonGenerator写入数据。它是三者中最有效的方法，是最低开销和最快的读/写操作。它类似于XML的Stax解析器。

- JsonGenerator - 写入JSON字符串。
- JsonParser - 解析JSON字符串。

## 使用JsonGenerator写入JSON



使用JsonGenerator是非常简单的。首先使用JsonFactory.createJsonGenerator()方法创建一个JsonGenerator，并用write***()方法来写每一个JSON值。

```java
JsonFactory jasonFactory = new JsonFactory();
JsonGenerator jsonGenerator = jasonFactory.createJsonGenerator(new File("student.json"), JsonEncoding.UTF8);
jsonGenerator.writeStartObject();
// "name" : "Mahesh Kumar"
jsonGenerator.writeStringField("name", "Mahesh Kumar");

```

## 使用JsonParser 读取JSON

使用JsonParser是非常简单的。首先使用JsonFactory.createJsonParser()方法创建JsonParser，并使用它的nextToken()方法来读取每个JSON字符串作为标记。检查每个令牌和相应的过程。



```java
JsonFactory jasonFactory = new JsonFactory();
JJsonParser jsonParser = jasonFactory.createJsonParser(new File("student.json"));
while (jsonParser.nextToken() != JsonToken.END_OBJECT) {
   //get the current token
   String fieldname = jsonParser.getCurrentName();
   if ("name".equals(fieldname)) {
      //move to next token
      jsonParser.nextToken();
      System.out.println(jsonParser.getText());             
   }
}
```

 

### 　　反序列化JSON到list

```java
  ObjectMapper mapper = new ObjectMapper();
  List<Budget> budgets = mapper.readValue(jsonString, new TypeReference<List<Budget>>() { });
```

 



```java
package com.xc.json.JackSon;

import java.util.Iterator;
import java.util.List;
import java.util.Map;

import com.fasterxml.jackson.databind.JavaType;
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

public class JackSon {
    public static void main(String[] args) throws Exception {
        String json = "{\"username\":\"zhangsan\",\"性别\":\"男\",\"company\":{\"companyName\":\"中华\",\"address\":\"北京\"},\"cars\":[\"蔚蓝\",\"特斯拉\"],\"logList\":[{\"id\":123,\"name\":\"abc\"},{\"id\":234,\"name\":\"efff\"}]}";
        System.out.println(json);
        ObjectMapper mapper = new ObjectMapper();
        JsonNode rootNode = mapper.readTree(json);

        // String的处理
        JsonNode usernameNode = rootNode.path("username");
        System.out.println("==username==");
        String textValue = usernameNode.textValue();
        System.out.println("textValue:" + textValue);
        String asText = usernameNode.asText();
        System.out.println("asText:" + asText);
        String username = mapper.writeValueAsString(usernameNode);
        System.out.println("username:" + username);

        // 转换成类型,Map类型，同样Integer,String,List，也可以同样处理
        JsonNode companyNode = rootNode.path("company");
        String company = mapper.writeValueAsString(companyNode);
        Map<String, Object> companyMap = mapper.readValue(company, Map.class);
        System.out.println("==company==");
        System.out.println(companyMap);

        // 数组的处理
        JsonNode carsNode = rootNode.path("cars");
        System.out.println("==cars==");
        Iterator<JsonNode> iterator = carsNode.elements();
        System.out.print("cars: [ ");
        while (iterator.hasNext()) {
            JsonNode marks = iterator.next();
            System.out.print(marks.textValue() + " ");
        }
        System.out.println("]");

        // 转换成List<Log>，类型
        JsonNode logListNode = rootNode.path("logList");
        String logJson = mapper.writeValueAsString(logListNode);
        JavaType logType = mapper.getTypeFactory().constructParametricType(List.class, Log.class);
        List<Log> logList = mapper.readValue(logJson, logType);
        System.out.println("==logList==");
        System.out.println(logList);

    }
}
```



 

```java
package com.xc.json.JackSon;

class Log {
    private int id;
    private String name;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "log [id=" + id + ", name=" + name + "]";
    }

}
```



 ![img](https://img2018.cnblogs.com/blog/1003856/201901/1003856-20190131172919594-2095130567.png)

 

jackson-api:

http://tool.oschina.net/apidocs/apidoc?api=jackson-1.9.9