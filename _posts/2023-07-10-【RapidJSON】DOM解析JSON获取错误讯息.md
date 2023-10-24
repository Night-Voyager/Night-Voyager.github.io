---
layout: post
title: 【RapidJSON】DOM解析JSON获取错误讯息
# author: 焜_8899
date: 2023-07-10 16:49:13 +0800
tags: [RapidJSON, json]
toc:  true
---

当使用 RapidJSON 的 DOM 方式解析 JSON 时，若解析过程出现错误，可以使用 `GetParseError_En()` 和 `GetParseError()` 获取相关的错误信息。

需注意的是，关于解析错误的处理方式，RapidJSON 的[中文文档](http://rapidjson.org/zh-cn/md_doc_dom_8zh-cn.html#ParseError)和[英文文档](https://rapidjson.org/md_doc_dom.html#ParseError)给出的示例代码略有不同。

[中文文档](http://rapidjson.org/zh-cn/md_doc_dom_8zh-cn.html#ParseError)使用了 `GetParseErrorCode()`：
```c++
#include "rapidjson/document.h"
#include "rapidjson/error/en.h"
 
// ...
Document d;
if (d.Parse(json).HasParseError()) {
    fprintf(stderr, "\nError(offset %u): %s\n", 
        (unsigned)d.GetErrorOffset(),
        GetParseError_En(d.GetParseErrorCode()));
    // ...
}
```

而[英文文档](https://rapidjson.org/md_doc_dom.html#ParseError)使用的是 `GetParseError()`：
```c++
#include "rapidjson/document.h"
#include "rapidjson/error/en.h"
 
// ...
Document d;
if (d.Parse(json).HasParseError()) {
    fprintf(stderr, "\nError(offset %u): %s\n", 
        (unsigned)d.GetErrorOffset(),
        GetParseError_En(d.GetParseError()));
    // ...
}
```

经试验，应使用  `GetParseError()`：
```shell
error: no member named 'GetParseErrorCode' in 'rapidjson::GenericDocument<rapidjson::UTF8<>>'; did you mean 'GetParseError'?
        std::cout << rapidjson::GetParseError_En(document.GetParseErrorCode()) << std::endl;
                                                          ^~~~~~~~~~~~~~~~~
                                                          GetParseError
```

以下是我的代码：
```c++
#include "rapidjson/document.h"
#include "rapidjson/encodedstream.h"
#include "rapidjson/error/en.h"
#include "rapidjson/istreamwrapper.h"
#include "rapidjson/reader.h"

// ...

std::ifstream inputFileStream("sample.json");
if (!inputFileStream.is_open()) {
    // ...
}

rapidjson::IStreamWrapper inputStreamWrapper(inputFileStream);  // 用 inputStreamWrapper 包装 inputFileStream

// AutoUTFInputStream 会先使用 BOM 来检测编码。
// 若 BOM 不存在，它便会使用合法 JSON 的特性来检测。
// 若两种方法都失败，它就会倒退至构造函数提供的 UTF 类型。
rapidjson::AutoUTFInputStream<unsigned, rapidjson::IStreamWrapper> encodedInputStream(inputStreamWrapper);  // 用 encodedInputStream 包装 inputStreamWrapper

rapidjson::Document document;  // Document 为 GenericDocument< UTF8<> >
document.ParseStream<0, rapidjson::AutoUTF<unsigned>>(encodedInputStream);  // 把任何 UTF 编码的文件解析至内存中的 UTF-8

if (document.HasParseError()) {
    std::cout << "------" << std::endl;
    std::cout << rapidjson::GetParseError_En(document.GetParseErrorCode()) << std::endl;  // 该行会报错，应使用 document.GetParseError()
}

// ...
```

另外，无论是[中文文档](http://rapidjson.org/zh-cn/document_8h_source.html)还是[英文文档](https://rapidjson.org/document_8h_source.html)，在 `rapidjson/document.h` 头文件中，也只有 `GetParseError()`：
```c++
 template <typename Encoding, typename Allocator = RAPIDJSON_DEFAULT_ALLOCATOR, typename StackAllocator = RAPIDJSON_DEFAULT_STACK_ALLOCATOR >
 class GenericDocument : public GenericValue<Encoding, Allocator> {
 public:
 // ...
  
     //! Get the \ref ParseErrorCode of last parsing.
     ParseErrorCode GetParseError() const { return parseResult_.Code(); }
  
 // ...
 };
```

而 `GetParseErrorCode()` 则出现在 `rapidjson/reader.h` 头文件中，也是[中](http://rapidjson.org/zh-cn/reader_8h_source.html)[英](https://rapidjson.org/reader_8h_source.html)一致，但尚未了解如何使用：
```c++
 template <typename SourceEncoding, typename TargetEncoding, typename StackAllocator = CrtAllocator>
 class GenericReader {
 public:
 // ...
  
     //! Get the \ref ParseErrorCode of last parsing.
     ParseErrorCode GetParseErrorCode() const { return parseResult_.Code(); }
  
 // ...
 }; // class GenericReader
```
