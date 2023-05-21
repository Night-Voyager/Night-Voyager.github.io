---
layout: post
title: 关于servlet解析HTTP请求的一点笔记
# author: 焜_8899
date: 2021-11-10 22:51:09 +0800
last_modified_at: 2021-11-18 18:59:57 +0800
tags: [servlet]
toc:  true
---

## 0. 说明

笔记内容关于以下方法：
 - String getContextPath()
 - String getQueryString()
 - String getRequestURI()

[菜鸟教程](https://www.runoob.com/servlet/servlet-client-request.html)的描述如下：
>下面的方法可用在 Servlet 程序中读取 HTTP 头。这些方法通过 HttpServletRequest 对象可用。
>
>|序号|方法 & 描述|
>| --- | --- |
>|13|String getContentType()<br>返回请求主体的 MIME 类型，如果不知道类型则返回 null。|
>|19|String getQueryString()<br>返回包含在路径后的请求 URL 中的查询字符串。|
>|23|String getRequestURI()<br>从协议名称直到 HTTP 请求的第一行的查询字符串中，返回该请求的 URL 的一部分。|

## 1. 示例代码

```
import java.io.IOException;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet("/NewServlet/*")
public class NewServlet extends HttpServlet {

    /**
     * Handles the HTTP <code>GET</code> method.
     *
     * @param request servlet request
     * @param response servlet response
     * @throws ServletException if a servlet-specific error occurs
     * @throws IOException if an I/O error occurs
     */
    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        String contextPath = request.getContextPath();
        String queryString = request.getQueryString();
        String requestURI = request.getRequestURI();
        
        response.getWriter().println(
                "ContextPath: " + contextPath + "\n" +
                "QueryString: " + queryString + "\n" +
                "RequestURI: " + requestURI
        );
    }

}

```

## 2. 请求结果

```
C:\Users\asus>curl http://localhost:8080/WebApplication1/NewServlet
ContextPath: /WebApplication1
QueryString: null
RequestURI: /WebApplication1/NewServlet

C:\Users\asus>curl http://localhost:8080/WebApplication1/NewServlet/
ContextPath: /WebApplication1
QueryString: null
RequestURI: /WebApplication1/NewServlet/

C:\Users\asus>curl http://localhost:8080/WebApplication1/NewServlet/123
ContextPath: /WebApplication1
QueryString: null
RequestURI: /WebApplication1/NewServlet/123

C:\Users\asus>curl http://localhost:8080/WebApplication1/NewServlet?id=123
ContextPath: /WebApplication1
QueryString: id=123
RequestURI: /WebApplication1/NewServlet

C:\Users\asus>curl http://localhost:8080/WebApplication1/NewServlet?id=123?num=123
ContextPath: /WebApplication1
QueryString: id=123?num=123
RequestURI: /WebApplication1/NewServlet

C:\Users\asus>curl http://localhost:8080/WebApplication1/NewServlet?id=123/123
ContextPath: /WebApplication1
QueryString: id=123/123
RequestURI: /WebApplication1/NewServlet

C:\Users\asus>curl -G -d 'id=123' -d 'tag=tag' http://localhost:8080/WebApplication1/NewServlet
ContextPath: /WebApplication1
QueryString: 'id=123'&'tag=tag'
RequestURI: /WebApplication1/NewServlet

C:\Users\asus>curl -G -d 'id=123' -d 'tag=tag' http://localhost:8080/WebApplication1/NewServlet/
ContextPath: /WebApplication1
QueryString: 'id=123'&'tag=tag'
RequestURI: /WebApplication1/NewServlet/
```
