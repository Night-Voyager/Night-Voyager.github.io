---
layout: post
title: 【Spring Boot】About Using @ManyToMany Annotation
# author: 焜_8899
date: 2021-05-06 21:51:07 +0800
tags: [Spring Boot]
toc:  true
---

## 1. Preface

### 1.1 Requirements & Background

The coursework is to develop a Digital Library, which can be considered as a Paper Review Management System.

A paper may have several keywords. While searching, a keyword will relate to many papers.

Therefore, the initial idea is to design two tables in the database, one for papers, one for keywords. <u>The relationship between these two tables is **many-to-many**.</u> (In fact, another method was found later. However, such method was not been taken advantage. The development process followed the initial idea.)

Some problems occurred. Consequently, this article is written to summarize the process of solving the problems. The code shown in this article reflects this process, and [has been published on GitHub](https://github.com/Night-Voyager/AManyToManyTestDemo). You can clone it, and then comment or uncomment different parts to see how it works.

### 1.2 Supplementary Explanation

This is the first time for me to use `Spring Boot` framework in order to implement the coursework. Previously, I used to work with `Django` and `Flask`, which are written in `python`. Thus, the project shown in this article keeps the previous habits. However, the project submitted for the coursework certainly takes the habits of `Spring Boot`.

There is also [a Chinese version of this article]({% link _posts/2021-05-05-【Spring Boot】关于@ManyToMany的使用.md %}).

## 2. The Code

### 2.1 Model Layer

###### Paper.java

```java
package com.example.demo.paper;

import com.example.demo.keyword.Keyword;
import lombok.Data;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import javax.persistence.*;
import java.util.Set;

@Data
@Entity
@EntityListeners(AuditingEntityListener.class)
public class Paper {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Integer id;

    private String title;

    @ManyToMany(cascade = CascadeType.PERSIST) // Give the currently set entity the authority to operate another entity.
    private Set<Keyword> keywords;
}

```

`@ManyToMany` annotation is used to specify the many-to-many relationship between two tables. The `cascade` attribute is added in the `@ManyToMany` annotation in `Paper.java`, whose value is `CascadeType.PERSIST`, which means

> Give the currently set entity the authority to operate another entity.^[2]^

###### Keyword.java

```java
package com.example.demo.keyword;

import com.example.demo.paper.Paper;
import lombok.Data;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import javax.persistence.*;
import java.util.Set;

@Data
@Entity
@EntityListeners(AuditingEntityListener.class)
public class Keyword {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Integer id;

    private String keyword;

    @ManyToMany(mappedBy = "keywords")
    private Set<Paper> papers;
}

```

The `mappedBy` attribute is added in the `@ManyToMany` annotation in `Keyword.java`, which means

>  the side where it is located is the owned side, and the side it points to is the owner.^[3][4]^

Once runs, the program will automatically create a  `paper` table, a `keyword` table, and a `paper_keywords` middle table to describe the relationship.

If the `mappedBy` attribute is not used, two middle tables will be created, which is not necessary and not concise.

It can be told from the above code that, in aiming of simplify the code, `@Data` annotation is used, which led to a hidden trouble for the subsequent development.

### 2.2 Controller Layer

###### PaperController.java

```java
package com.example.demo.paper;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

@Controller
@RequestMapping(path = "/paper")
public class PaperController {
    @Autowired
    private PaperRepository paperRepository;

    @GetMapping(path = "/all")
    public @ResponseBody Iterable<Paper> getAllPapers() {
        return paperRepository.findAll();
    }

    @PostMapping(path = "/add")
    @ResponseBody
    public ResponseEntity<?> addNewPaper(@RequestBody Paper paper) {
        paperRepository.save(paper);
        return ResponseEntity.status(HttpStatus.CREATED).build();
    }
}

```

Two simple APIs are implemented to response to GET requests and POST requests respectively.

According to [the official document](https://spring.io/guides/gs/accessing-data-mysql/)^[5]^, `PaperRepository` extends `CrudRepository` interface, which will not discuss in detail here.

## 3. Data Format of POST Requests

### 3.1 JSON

In the Controller, the method where POST requests mapped to takes a Paper object as parameter. As a consequence, the input JSON data can be deserialized to a Paper object. However, the JSON data can only be deserialized if it is in the right format.

An example of the right format:

```json
{
    "title": "pure json",
    "keywords": [
        {"keyword": "CS"},
        {"keyword": "DS"},
        {"keyword": "MIS"}
    ]
}
```

### 3.2 cURL

While using cURL under Windows cmd to send JSON data, the data itself should be marked by double quotation marks, and the double quotation marks in the JSON data should be labeled with backslash escapes. The following is an example (the port number of this project is set to be 8081, the same below):

```cmd
curl -X POST -H "content-type: application/json" -d "{\"title\": \"from_curl\", \"keywords\": [{\"keyword\": \"BS\"}]}" http://localhost:8081/paper/add
```

In the example, 

- `-X` parameter is used to specify the method of HTTP request,
- `-H` parameter is used to add HTTP header, which is not sensitive to case and space, and specifies the content type to be sent,
- `-d` parameter is used to add the data body of a POST request, which will automatically send a POST request if used, and thus `-X POST` can be omitted,
- at last is the url where the request to be sent to.

For more information on the usage of cURL, please refer to [this article (in Chinese)](http://www.ruanyifeng.com/blog/2019/09/curl-reference.html)^[6]^.

### 3.3 Python

The `requests` package of Python can be taken advantage of to send HTTP requests. The code that I use to test this project is shown as follows:

```python
import requests

data = {
    "title": "from_python",
    "keywords": [
        {"keyword": "CS"},
        {"keyword": "DS"},
        {"keyword": "MIS"}
    ]
}

print(
    requests.post(
        url='http://127.0.0.1:8081/paper/add',
        json=data
    ).text
)

```

### 3.4 Supplementary

Personally speaking, the format of data given in 3.1 is not ideal. A simpler format that is also easier for frontend to operate could be like:

```json
{
    "title": "pure json",
    "keywords": ["CS", "DS", "MIS"]
}
```

However, it is not realized. I sincerely appreciate it if someone could tell a way to achieve it!

## 4. Error Occurs When Sending GET Requests

### 4.1 Error Information

Build and run the previously given program, data can be successfully received by the backend. But while trying to get data, the following error messages will be shown:

```java
*** java.lang.instrument ASSERTION FAILED ***: "!errorOutstanding" with message transform method call failed at JPLISAgent.c line: 844
*** java.lang.instrument ASSERTION FAILED ***: "!errorOutstanding" with message transform method call failed at JPLISAgent.c line: 844
*** java.lang.instrument ASSERTION FAILED ***: "!errorOutstanding" with message transform method call failed at JPLISAgent.c line: 844
2021-04-28 17:38:01.558  WARN 9476 --- [nio-8081-exec-1] .w.s.m.s.DefaultHandlerExceptionResolver : Resolved [org.springframework.http.converter.HttpMessageNotWritableException: Could not write JSON: Infinite recursion (StackOverflowError); nested exception is com.fasterxml.jackson.databind.JsonMappingException: Infinite recursion (StackOverflowError) (through reference chain: java.util.ArrayList[0]->com.example.demo.paper.Paper["keywords"])]
2021-04-28 17:38:01.560  WARN 9476 --- [nio-8081-exec-1] o.h.e.loading.internal.LoadContexts      : HHH000100: Fail-safe cleanup (collections) : org.hibernate.engine.loading.internal.CollectionLoadContext@2c52ae69<rs=HikariProxyResultSet@1263489024 wrapping Result set representing update count of -1>
2021-04-28 17:38:01.560  WARN 9476 --- [nio-8081-exec-1] o.h.e.loading.internal.LoadContexts      : HHH000100: Fail-safe cleanup (collections) : org.hibernate.engine.loading.internal.CollectionLoadContext@7af84376<rs=HikariProxyResultSet@1287874177 wrapping Result set representing update count of -1>
2021-04-28 17:38:01.561  WARN 9476 --- [nio-8081-exec-1] o.h.e.loading.internal.LoadContexts      : HHH000100: Fail-safe cleanup (collections) : org.hibernate.engine.loading.internal.CollectionLoadContext@59b5f55c<rs=HikariProxyResultSet@2018225027 wrapping Result set representing update count of -1>
```

The contents of the beginning three lines may appear more than three times. The contents after the fifth line will also show up for many times. The repeated contents are omitted.

### 4.2 Reason of Error

Such situation is due to the `StackOverflowError` caused by `infinite recursion`. The reason why infinite recursion occurs is stated below.

Primarily, in the model layer, the `keywords` field in `Paper` is related to `Keyword`, and the `papers` field in `Keyword` is related to `Paper`. Assume that the data of all `Paper` is requested. In this case, when serializing, the data of a `paper` object will be loaded. Then, all `keyword` objects relate to that `paper` object will consequently be loaded. After that, all `paper` objects relate to those `keyword` objects will also be loaded. However, these `paper` objects include the `paper` object loaded at the beginning. As a result, The above process will continue to loop until the stack space is insufficient.

### 4.3 Solution

The key of solving this issue is to stop the infinite recursion. Additionally, enough data should have been loaded.

The code in the controller layer does not have to be changed.

The `@Data` annotation must be deleted in the model layer. Meanwhile, in order to keep the code concise, `@Setter` and `@Getter` annotations can be used. Then, the annotation `@JsonIdentityInfo(generator = ObjectIdGenerators.PropertyGenerator.class, property = "id")` should be added before the declaration of class. Consequently, a serialized object will not be serialized again. Because of this, the infinite recursion is interrupted and enough data is loaded.

The final code is shown below:

###### Paper.java

```java
package com.example.demo.paper;

import com.example.demo.keyword.Keyword;
import com.fasterxml.jackson.annotation.*;
import lombok.Getter;
import lombok.Setter;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import javax.persistence.*;
import java.util.Set;

@Entity
@EntityListeners(AuditingEntityListener.class)
@JsonIdentityInfo(generator = ObjectIdGenerators.PropertyGenerator.class, property = "id")
@Setter
@Getter
public class Paper {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Integer id;

    private String title;

    @ManyToMany(cascade = CascadeType.PERSIST) // Give the currently set entity the authority to operate another entity.
    private Set<Keyword> keywords;
}

```

###### Keyword.java

```java
package com.example.demo.keyword;

import com.example.demo.paper.Paper;
import com.fasterxml.jackson.annotation.*;
import lombok.Getter;
import lombok.Setter;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import javax.persistence.*;
import java.util.Set;

@Entity
@EntityListeners(AuditingEntityListener.class)
@JsonIdentityInfo(generator = ObjectIdGenerators.PropertyGenerator.class, property = "id")
@Setter
@Getter
public class Keyword {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Integer id;

    private String keyword;

    @ManyToMany(mappedBy = "keywords")
    private Set<Paper> papers;
}

```

### 4.4 Supplementary

This session may look like a free pen.

It can be told from [the project of this article](https://github.com/Night-Voyager/AManyToManyTestDemo)^[1]^ that, considerable number of methods on the Internet have been tried to use. However, none of them got the effect shown in the corresponding articles, which really confused me. I have tried to use annotations such as `@JsonIgnore`, `@JsonIgnoreProperties`, `@JsonManagedReference`, `@JsonBackReference`, etc. Additionally, I have tried to manually delete the part that causes infinite recursion. Moreover, I have also tried to write a serializer on my own. However, none of these methods works. The error information may change a little bit, but it always exists.

Finally, I accidentally found that, using methods on the Internet after deleting `@Data` annotation would work. And then I realized that it is because of `@Data` annotation.

Understood the cause, I adopted the method described in 4.3 after comparing.

I have also considered to figure out the reason why `@Data` annotation would lead to infinite recursion. But at present, I only know that `@Data` can be used as a simplified equivalent substitute for some other annotations. Limited by the deadline of the coursework submission, I have to focus on more necessary parts. And there may be no time for me to study this part in a period of time. I also sincerely appreciate it if someone knows it and is willing to tell something about it!

## 5. Reference

Most of the references in this article have been marked in the corresponding position in the form of links. The following is a complete list of references:

[1] Night-Voyager. (2021, Apr. 20). AManyToManyTestDemo. Retrieved Apr. 26, 2021, from https://github.com/Night-Voyager/AManyToManyTestDemo.

[2] Osheep. (2017, Aug. 25). 【简单易懂】JPA概念解析：CascadeType（各种级联操作）详解。. Retrieved Apr. 26, 2021, from 架构修炼: https://www.osheep.cn/3680.html. (in Chinese)

[3] 笙歌会停. (2019, Oct. 26). @ManyToMany中的mappedy. Retrieved Apr. 26, 2021, from SegmentFault 思否: https://segmentfault.com/a/1190000020806546. (in Chinese, contains typos)

[4] NimChimpsky, & JB Nizet. (2013, Jan. 1). @ManyToMany(mappedBy = "foo"). Retrieved Apr. 26, 2021, from Stack Overflow: https://stackoverflow.com/questions/14111607/manytomanymappedby-foo.

[5] VMware, Inc. or its affiliates. (2021, Mar. 11). Getting Started \| Accessing data with MySQL. Retrieved Mar. 19, 2021, from Spring: https://spring.io/guides/gs/accessing-data-mysql/.

[6] 阮一峰. (2019, Sept. 5). curl 的用法指南. Retrieved Apr. 6, 2021, from 阮一峰的网络日志: http://www.ruanyifeng.com/blog/2019/09/curl-reference.html. (in Chinese)
