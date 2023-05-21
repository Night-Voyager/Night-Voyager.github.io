---
layout: post
title: 【Spring Boot】关于@ManyToMany的使用
# author: 焜_8899
date: 2021-05-05 22:00:07 +0800
last_modified_at: 2021-05-17 17:05:53 +0800
tags: [Spring Boot]
toc:  true
---

## 1. 写在前面

### 1.1 需求背景

学校作业让开发一个“数字图书馆”，具体要求可以理解为一个论文评审管理系统。

一篇论文会有多个关键词。而搜索的时候，一个关键词会对应多个论文。

所以，最初设计数据库时的想法是，论文一个表，关键字一个表，<u>两个表之间是**多对多**的关系</u>。（其实后来发现可能有别的方案，但没有尝试，就按照当时的思路做下去了。）

这之后遇到了一些问题，于是在这篇文章整理下过程与解决方法。[体现这一过程的代码已经发布到了GitHub上](https://github.com/Night-Voyager/AManyToManyTestDemo)^[1]^，亦是本文中的内容。有需要的可以下载，也可以尝试注释掉或取消注释不同的部分，看看效果。（~￣︶￣~）

### 1.2 补充说明

这个作业是本人第一次接触`Spring Boot`。在这之前，本人学习的是用`python`语言开发的`Django`和`Flask`框架。因此，作为本文示例的项目中的目录结构保留了之前的习惯。当然，用于交作业的项目的目录结构采取的是`Spring Boot`的习惯。（~￣▽￣~）

因为本课程教师是外教，所以本文提供了[英文版本]({% link _posts/2021-05-06-【Spring Boot】About Using @ManyToMany Annotation.md %})。

## 2. 代码部分

### 2.1 模型层（Model）

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

    @ManyToMany(cascade = CascadeType.PERSIST) // 给当前设置的实体操作另一个实体的权限。
    private Set<Keyword> keywords;
}

```

其中使用了`@ManyToMany`注解，表示两表之间是多对多关系。`Paper.java`的`@ManyToMany`注解中加入了`cascade`属性，其值为`CascadeType.PERSIST`，表示

> 给当前设置的实体操作另一个实体的权限。^[2]^

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

`Keyword.java`的`@ManyToMany`注解中加入了`mappedBy`属性，表示

>  其所在的一方是被拥有方，而其指向的是拥有方。^[3][4]^

运行程序后，会自动生成`paper`表和`keyword`表，以及一个`paper_keywords`中间表。

假如没有在`@ManyToMany`注解中加入`mappedBy`属性的话，会生成两张中间表，没有必要也不够简洁。

在以上代码中可以看到，为了简化代码，使用了`@Data`注解。这为后续的开发埋下了隐患。

### 2.2 控制层（Controller）

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

这样就实现了两个简单的接口，分别响应GET请求和POST请求。

其中，`PaperRepository`根据[官方文档](https://spring.io/guides/gs/accessing-data-mysql/)^[5]^拓展了`CrudRepository`接口，在此不做详细展开。

## 3. POST请求数据格式

### 3.1 JSON

在控制层中，POST请求被映射到的函数接收的参数是个Paper对象。这样，传入接口的JSON数据可以被反序列化为Paper对象。然而，只有当传入的JSON格式正确时，才能进行反序列化。

一个格式正确的例子：

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

假如在Windows的命令行下使用cURL发送JSON格式的数据，那么所传的JSON数据本身需要用双引号引起来，而JSON数据里面的双引号需要加反斜杠进行转义。下面是一个例子（本文项目端口号被设为8081，下同）：

```cmd
curl -X POST -H "content-type: application/json" -d "{\"title\": \"from_curl\", \"keywords\": [{\"keyword\": \"BS\"}]}" http://localhost:8081/paper/add
```

其中：

- `-X`参数用于指定HTTP请求的方法；
- `-H`参数用于添加HTTP标头，对大小写和空格不敏感，这里指定了发送的内容类型；
- `-d`参数用于添加发送POST请求的数据体，使用后将自动使用POST方法发送请求，因此可省略`-X POST`；
- 最后是请求的地址。

更多关于cURL的用法可以参考[这篇文章](http://www.ruanyifeng.com/blog/2019/09/curl-reference.html)^[6]^。

### 3.3 Python

Python中的`requests`包可以用来发送HTTP请求。本人用于测试本项目的代码如下：

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

### 3.4 补充

个人认为，3.1中所给的数据格式其实不是很理想。更加简单且便于前端操作的格式应当是：

```json
{
    "title": "pure json",
    "keywords": ["CS", "DS", "MIS"]
}
```

然而并没有实现。若是有人能告知解决方法，感激不尽！

## 4. GET请求无限递归报错

### 4.1 报错信息

将以上程序编译运行，可以正常插入数据。但当读取数据时，出现类似以下报错：

```java
*** java.lang.instrument ASSERTION FAILED ***: "!errorOutstanding" with message transform method call failed at JPLISAgent.c line: 844
*** java.lang.instrument ASSERTION FAILED ***: "!errorOutstanding" with message transform method call failed at JPLISAgent.c line: 844
*** java.lang.instrument ASSERTION FAILED ***: "!errorOutstanding" with message transform method call failed at JPLISAgent.c line: 844
2021-04-28 17:38:01.558  WARN 9476 --- [nio-8081-exec-1] .w.s.m.s.DefaultHandlerExceptionResolver : Resolved [org.springframework.http.converter.HttpMessageNotWritableException: Could not write JSON: Infinite recursion (StackOverflowError); nested exception is com.fasterxml.jackson.databind.JsonMappingException: Infinite recursion (StackOverflowError) (through reference chain: java.util.ArrayList[0]->com.example.demo.paper.Paper["keywords"])]
2021-04-28 17:38:01.560  WARN 9476 --- [nio-8081-exec-1] o.h.e.loading.internal.LoadContexts      : HHH000100: Fail-safe cleanup (collections) : org.hibernate.engine.loading.internal.CollectionLoadContext@2c52ae69<rs=HikariProxyResultSet@1263489024 wrapping Result set representing update count of -1>
2021-04-28 17:38:01.560  WARN 9476 --- [nio-8081-exec-1] o.h.e.loading.internal.LoadContexts      : HHH000100: Fail-safe cleanup (collections) : org.hibernate.engine.loading.internal.CollectionLoadContext@7af84376<rs=HikariProxyResultSet@1287874177 wrapping Result set representing update count of -1>
2021-04-28 17:38:01.561  WARN 9476 --- [nio-8081-exec-1] o.h.e.loading.internal.LoadContexts      : HHH000100: Fail-safe cleanup (collections) : org.hibernate.engine.loading.internal.CollectionLoadContext@59b5f55c<rs=HikariProxyResultSet@2018225027 wrapping Result set representing update count of -1>
```

其中前三行的内容可能会出现不止三次，第5行以后的内容会重复多次，重复的内容省略。

### 4.2 报错原因

出现这种情况，原因是存在无限递归（Infinite recursion）导致的栈溢出错误（StackOverflowError）。

而之所以会出现无限递归，原因如下：

首先，在模型层，`Paper`中的`keywords`字段关联向了`Keyword`，`Keyword`中的`papers`字段又关联回了`Paper`。假设现在要读取所有`Paper`的数据。这种情况下，在进行序列化的时候，一个`paper`对象的数据会被加载。然后，其所关联的所有`keyword`对象的数据也会被加载。这时，与这些`keyword`相关联的`paper`对象的数据也都会被加载。而这些`paper`中就包含刚才已经被加载过的那一个`paper`。于是，上述过程会一直持续循环，直到栈空间不足。

### 4.3 处理方法

要解决这一问题，关键点在于终止无限递归。当然，此时需要加载完足够的数据。

上述控制层的代码不需要更改。

模型层的代码，须将`@Data`注解删除。同时，为了保持对代码的简化，可使用`@Setter`和`@Getter`注解。然后，在类的声明之前加上`@JsonIdentityInfo(generator = ObjectIdGenerators.PropertyGenerator.class, property = "id")`。这样，一个被序列化过的对象将不会再次被序列化，从而既终止了无限递归又保证加载了足够的数据。

最终代码如下：

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

    @ManyToMany(cascade = CascadeType.PERSIST) // 给当前设置的实体操作另一个实体的权限。
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

### 4.4 补充

这一段可能会看着像个闲笔。

从[本文的项目](https://github.com/Night-Voyager/AManyToManyTestDemo)^[1]^可看出，我已经尝试过很多网上建议的方法，但当时全部都没有得到相应文章所说的效果。我一度感到疑惑。我试过使用`@JsonIgnore`、`@JsonIgnoreProperties`、`@JsonManagedReference`、`@JsonBackReference`等注解，也试过在`controller`中手动删除会导致无限递归的部分，更试过自己写一个序列化器。然而，结果都只是报错的信息略有变化，但报错的情况始终存在。

最终，我偶然试出，在删除`@Data`注解后，再使用网上的方法，可以实现相应的效果，才意识到可能是`@Data`注解的问题。

发现这个问题后，再经过对比，我才最终采用本文4.3小节中说明的方法。

我也想过试着找一下`@Data`注解会导致无限递归的原因。但目前只了解到`@Data`可以作为另外几个注解的简化的等效替代。受限于作业提交的时间，我只得把注意力放到相对更要紧的地方，短时间内应该没有时间专研这一块的内容。若是有人有所了解，愿意交流，亦感激不尽！

## 5. 参考文献

本文大多参考文献已以链接的形式标注在了相应位置。以下是完整参考文献列表：

[1] Night-Voyager. AManyToManyTestDemo[EB/OL]. (2021-04-20) [2021-04-26]. https://github.com/Night-Voyager/AManyToManyTestDemo.

[2] Osheep. 【简单易懂】JPA概念解析：CascadeType（各种级联操作）详解。 - 架构修炼[EB/OL]. (2017-08-25) [2021-04-26]. https://www.osheep.cn/3680.html.

[3] 笙歌会停. @ManyToMany中的mappedy - SegmentFault 思否[EB/OL]. (2019-10-26) [2021-04-26]. https://segmentfault.com/a/1190000020806546. (这篇文章中有拼写错误)

[4] NimChimpsky 和 JB Nizet. java - @ManyToMany(mappedBy = &quot;foo&quot;) - Stack Overflow[EB/OL]. (2013-01-01) [2021-04-26]. https://stackoverflow.com/questions/14111607/manytomanymappedby-foo.

[5] VMware, Inc. or its affiliates. Getting Started \| Accessing data with MySQL[EB/OL]. (2021-03-11) [2021-03-19]. https://spring.io/guides/gs/accessing-data-mysql/.

[6] 阮一峰. curl 的用法指南 - 阮一峰的网络日志[EB/OL]. (2019-09-05) [2021-04-06]. http://www.ruanyifeng.com/blog/2019/09/curl-reference.html.
