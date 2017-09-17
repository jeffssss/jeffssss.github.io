---
published: true
title: 了解Language Server
layout: post
author: Jeffssss 
comments: true
category: Java
tags:
- Java
---

本周(具体是9月12号)，Atom官网宣布GitHub与Facebook合作联合推出Atom IDE，将IDE风格的功能带到了Atom上。初始版本的Atom IDE支持TypeScript、Flow、JavaScript、Java、C#和PHP，它利用强大的语言服务器为代码和项目提供深度的语法分析。
作为一个好奇宝宝，我立马就去了解了一下语言服务器(嗯？为啥不是去调研Atom？)

## Language Server(语言服务器)

什么是Language Server？ [VScode的使用文档](https://code.visualstudio.com/docs/extensions/example-language-server)里有简单介绍(注:本文Language均指编程语言).

> Language servers allow you to add your own validation logic to files open in VS Code. Typically you just validate programming languages. However validating other file types is useful as well. A language server could, for example, check files for inappropriate language.

Language Server翻译为“语言服务器”，并不是说它真的是一个服务器，而是它把语言相关的特性和功能从IDE中解耦出来，作为一个独立的程序单独运行，提供了例如引用查询(Find All References)等功能的具体实现，Client是编辑器或IDE，例如Atom、VScode等。

更加确切的解释是，Language Server是某语言的Language Server Protocol具体实现。

## Language Server Protocol

### 简介

什么是Language Server Protocol? 它的[官方Wiki](https://github.com/Microsoft/language-server-protocol)如此写到:

> The Language Server protocol is used between a tool (the client) and a language smartness provider (the server) to integrate features like auto complete, goto definition, find all references and alike into the tool. 

Language Server Protocol(LSP)，语言服务端协议，是由微软提出，并与Redhat、Codenvy、Sourcegraph等公司联合推出的开源协议(使用Creative Commons Attribution以及MIT License)，用于智能语言服务程序向编辑器等工具提供诸如自动补全(auto complete)、跳转到定义(go to definition)等功能的场景。LSP根据编辑器与编程语言分析器的交互制定了一套公共的流程。每个具体的语言实现LSP协议后，编辑器通过调用不同语言的Language Server，从而实现支持不同语言特性的需求。

LSP的目的在于解决市场上多语言与多编辑器的问题。如果m个编辑器，每个编辑器支持n个语言，需要开发m * n次。当编辑器和语言均支持LSP时，仅需要开发m + n次即可。

目前LSP版本为3.0，Client与Server通信基于[JSON RPC v2.0协议](http://www.jsonrpc.org/specification) [(→中文版文档)](http://wiki.geekdream.com/Specification/json-rpc_2.0.html)，支持TCP以及Stdin/Stdout传输,这意味着，你的编辑器既可以从本地运行的Language Server获取语法信息，也可以连接远程的服务器获取语法信息。

下图解释了编辑器与Language Server交互的简单流程(图来自LSP Wiki)

![communication ](https://raw.githubusercontent.com/Microsoft/language-server-protocol/master/images/interaction-diagram.png)

1. 当用户在编辑器中打开文件，编辑器会发送`didOpen`通知，服务器会将对应文件读入到内存中。
2. 当用户编辑文件时，编辑器会发送`didChange`通知，服务器会根据修改的内容立即更新对应的语法信息
3. 当用户修改文件且服务器更新语法信息时监测到Warning或者Error后，会通过`publishDiagostics`通知将异常同步给编辑器。
4. 当用户想要查询某个变量的声明，编辑器会发送`definition`请求，服务器会返回对应文件的URI以及文件内的定位信息Range，编辑器会跳转到RI指定的文件以及Range指定的位置。
5. 当用户关闭文件，编辑器会发送`didClose`通知，服务器会吧文件保存在文件系统并且释放掉内存里的相关内容。

### 协议内容简介

查看了一下LSP的Git信息，LSP Star数为1805，Fork 135，Watch 151。LSP的协议内容的上一次修改是在4天前(2017-09-14)，目前版本是3.0，协议中的方法共计40个。LSP 2.x版本的最后一次提交日期是2016-10-21，协议中的方法共计33个。个人感觉：1. LSP在开源社区中发展良好，2. LSP逐渐稳定，协议内容不会有大的变化(不会像Python或者Swift)

LSP按消息的类型共分为：

1. General
	
	通用类型
	
	* initialize
	* initialized
	* shutdown
	* exit
	* $/cancelRequest
	
2. Window
	
	弹窗类型，用于Serve请求Client，其特点是通过弹窗展示消息或者记录日志。
	
	* window/showMessage
	* window/showMessageRequest
	* window/logMessage
	* telemetry/event
	
3. Client

	客户端类型，用于Server请求Client，用以实现动态功能注册的功能。
	
	* client/registerCapability
	* client/unregisterCapability
	
4. Workspace

	工作空间类型，用于Client请求Serve，其相关方法均与工作空间的配置信息或者行为相关。
	
	* workspace/didChangeConfiguration
	* workspace/didChangeWatchedFiles
	* workspace/symbol
	* workspace/executeCommand
	* workspace/applyEdit
	
5. Document

	文档类型，例如didOpen，大部分用于Client请求Server，是方法最多的类型。其方法均与文档的操作相关。
	
	* textDocument/publishDiagnostics
	* textDocument/didOpen
	* textDocument/didChange
	* textDocument/willSave
	* textDocument/willSaveWaitUntil
	* textDocument/didSave
	* textDocument/didClose
	* textDocument/completion
	* completionItem/resolve
	* textDocument/hover
	* textDocument/signatureHelp
	* textDocument/references
	* textDocument/documentHighlight
	* textDocument/documentSymbol
	* textDocument/formatting
	* textDocument/rangeFormatting
	* textDocument/onTypeFormatting
	* textDocument/definition
	* textDocument/codeAction
	* textDocument/codeLens
	* codeLens/resolve
	* textDocument/documentLink
	* documentLink/resolve
	* textDocument/rename
	
详细的协议内容参见 [Language Server Protocol](https://github.com/Microsoft/language-server-protocol/blob/master/protocol.md)，文档详细阐明了每个方法的功能，并提供JSON RPC版本的请求例子。

### 目前LSP的实现情况

LSP各个语言的实现请求可以在LSP开源社区[langserver.org](http://langserver.org/)中查询到

截止目前，大部分主流语言均有对应的Language Server。部分语言，例如C#、Java有多个不同厂商提供的Language Server。同时，部分主流编辑器和IDE也纷纷支持LSP，包括Eclipse、VScode、Sublime Text & Sublime Text 3、Atom、Emacs、Vim等等。

这样看来，LSP现在的发展形势可以说是很不错，已经在IDE改革的道路上稳步前行。

## Language Server体验

为了了解LSP的效果，我决定亲自体验一番。因为我工作中使用Java语言开发，所以也决定体验一下Java的Language Server。

使用工具为

* 编辑器：VScode(据说体验很不错)
* 插件 [Language support for Java ™ for Visual Studio Code](https://github.com/redhat-developer/vscode-java)。(VScode默认不提供Java的Language Server，所以需要下载Java插件，这个插件由RedHat提供，底层Language Server采用了Eclipse Foundation以及RedHat联合出品的[eclipse.jdt.ls](https://github.com/eclipse/eclipse.jdt.ls))

我试验了如下功能：

1. 第一次打开工程时，会自动加载Project下的内容，构件完整的语法树(花费一定的时间)

2. 打开文件时，会检查项目的classpath的相关信息。下图是没有查询到classpath信息，会弹出警告框。

	![classpath incomplete](/img/classpath incomplete.png)

3. 单击某个变量，能自动高亮文件中所有此变量的引用
	
	![varialbe hightlight](/img/variable highlight.png)

4. 查询某个变量的定义，能够自动跳转到变量的定义或者声明位置
	
5. 点击某个方法，出现方法的定义信息

	![function info](/img/function info.png)
	
6. 查询某个方法的引用，能否查询到该方法被使用的地方。

	![find usages](/img/find usages.png)
	
通过设置`java.trace.server`变量，我们能在控制台看到VScode与Language Server交互的具体信息。

```
[Trace - 02:47:09] Sending request 'textDocument/hover - (110)'.
Params: {
    "textDocument": {
        "uri": "file:///Users/jifeng/git/eclipse.jdt.ls/org.eclipse.jdt.ls.core/src/org/eclipse/jdt/ls/core/internal/JDTUtils.java"
    },
    "position": {
        "line": 222,
        "character": 34
    }
}

[Trace - 02:47:09] Sending request 'textDocument/documentHighlight - (111)'.
Params: {
    "textDocument": {
        "uri": "file:///Users/jifeng/git/eclipse.jdt.ls/org.eclipse.jdt.ls.core/src/org/eclipse/jdt/ls/core/internal/JDTUtils.java"
    },
    "position": {
        "line": 222,
        "character": 34
    }
}
```

仅从体验来说，和IDEA的差距不大，能够满足日常开发中，大部分的coding工作。查询是否有BUG、查询速度快慢等等更细节的问题，需要去细心测试才能发现，但是从第一印象来说，VScode + Language Server的体验给我眼前一亮的感觉。

## 最后

对于使用JavaScript、Python等脚本语言的同学来说，LSP会给你开发环境的新选择，你可以通过选择自己喜欢或者熟悉的编辑器并搭配口碑优秀的Language Server来提高开发效率，再也不用等待臃肿的IDE响应你的操作了。

对于写Java的同学，目前还是用IDEA吧。虽然现在LSP的出现能够让编辑器支持Java语法以及代码分析的功能，但是有很多Java开发中必要的功能没有得到支持，比如运行程序，调试程序，就算有相应的Java插件能够支持，我想也没有直接使用IDE方便吧。