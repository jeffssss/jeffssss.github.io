---
published: true
title: iOS发送邮件及其中文乱码解决方法
layout: post
author: Jeffssss 
category: iOS
tags:
- iOS
- mail

---


* [引](#yin)
* [skpsmtpmessage介绍](#skpsmtpmessage)
* [基本使用](#jibenshiyong)
	* [发送邮件](#fasongyoujian)
	* [发送之后](#fasongzhihou)
* [标题中文乱码解决方法](#biaoti)
* [附件文件名中文乱码解决方法](#fujian)


---

<h2 id="yin">引</h2>

要我说呀，你如果不曾碰到奇奇怪怪的需求都不好意思说你是个程序猿。最近我碰到了，所以才有这篇文章，记录一下过程中遇到的问题。

总而言之，这篇文章是记录一下我是如何在iOS端实现自动发送邮件功能的。某个应用场景为：

* 用户点击iOS端的"发送邮件"按钮，其功能是将某些数据以邮件的形式`自动发送`到用户的邮箱。
* iOS端向特定SMTP服务器发起请求，发送邮件。

经过一段时间的百度(好孩子不翻qiang)，我了解到，iOS发送邮件目前有[3种实现方式](http://blog.csdn.net/zhibudefeng/article/details/12952203)：

1. 使用openURL来实现发邮件的功能
2. 使用MFMailComposeViewController来实现发邮件的功能
3. 使用比较有名的开源SMTP协议来实现，例如SKPSMTPMessage

前两者会跳出当前页面，与我们"自动发送"的要求不符，所以我们使用开源的[SKPSMTPMessage](https://github.com/jetseven/skpsmtpmessage)实现功能。

如果你只是想知道怎么解决乱码问题，请转到最后。

<h2 id="skpsmtpmessage">skpsmtpmessage介绍</h2>

github地址：[https://github.com/jetseven/skpsmtpmessage](https://github.com/jetseven/skpsmtpmessage)

skpsmtpmessage 简单的封装了SMTP协议，让你的app可以发送邮件。它是手动封装的，有很大的自由度，可以在后台发送请求，从而`自动发送`邮件。

需要注意的一点是，你最好
>To use this in your app, add the files in the SMTPLibrary directory to your project.

原因是，Cocoapod里的skpsmtpmessage版本较老，会引起标题中文乱码的问题。

不得不吐槽一句，iOS的SMTP封装的第三方开源库真的好少，我找来找去就找到这一个陈年老库，而Java的类似库印象中比比皆是。

不得不再吐槽一句，让移动端自动发送邮件的需求也是挺少见的。我没文化所以觉得发送邮件应该是放在后台来实现的。

<h2 id="jibenshiyong">基本使用</h2>

<h3 id="fasongyoujian">发送邮件</h3>

直接放代码吧，使用上很方便直观。

{% highlight objectivec linenos %}
SKPSMTPMessage *testMsg = [[SKPSMTPMessage alloc] init];
testMsg.fromEmail = @"your email";
testMsg.toEmail = @"target email";
testMsg.relayHost = @"smtp server";
testMsg.requiresAuth = YES;//是否需要认证(登陆)
testMsg.login = @"登陆账号";
testMsg.pass = @"password";
testMsg.delegate = sender;//指定delegate
//testMsg.bccEmail = [defaults objectForKey:@"bccEmal"];//这个不知道干嘛的。。。找不到资料
testMsg.wantsSecure = NO;//是否用ssl
testMsg.subject = @"SMTPMessage Test Message，测测试试";
NSDictionary *plainPart = [NSDictionary dictionaryWithObjectsAndKeys:@"text/plain; charset=UTF-8",kSKPSMTPPartContentTypeKey,
                          @"邮件由系统自动发送，请勿回复。",kSKPSMTPPartMessageKey,@"8bit",kSKPSMTPPartContentTransferEncodingKey,nil];
testMsg.parts = [NSArray arrayWithObjects:plainPart,nil]; 
dispatch_async(dispatch_get_main_queue(), ^{
   [testMsg send];
});
{% endhighlight %}

你也可以添加附件，除了以上代码，你需要做的是

首先，添加附件块

{% highlight objectivec linenos %}
NSString *vcfPath = [[NSBundle mainBundle] pathForResource:@"test" ofType:@"vcf"];
NSData *vcfData = [NSData dataWithContentsOfFile:vcfPath]; 
NSDictionary *vcfPart = [NSDictionary dictionaryWithObjectsAndKeys:@"text/directory;\r\n\tx-unix-mode=0644;\r\n\tname=\"test.vcf\"",kSKPSMTPPartContentTypeKey,@"attachment;\r\n\tfilename=\"test.vcf\"",kSKPSMTPPartContentDispositionKey,[vcfData encodeBase64ForData],kSKPSMTPPartMessageKey,@"base64",kSKPSMTPPartContentTransferEncodingKey,nil]; 
{% endhighlight %}

然后，在test.parts中加上刚才的附件块
	
{% highlight objectivec %}
testMsg.parts = [NSArray arrayWithObjects:plainPart,vcfPart,nil];
{% endhighlight %}
	
以上代码大部分来自skpsmtpmessage提供的Demo，需要注意的是，demo中使用多线程发送邮件，而我刚才的代码是在主线程发送邮件，原因是，在多线程环境，邮件无法发送成功。这与其内部机制有关，我暂时没看出个所以然。看以后会不会有兴趣研究研究。

<h3 id="fasongzhihou">发送之后</h3>

发送之后，会调用delegate方法。

成功，调用：

{% highlight objectivec  %}
- (void)messageSent:(SKPSMTPMessage *)message;
{% endhighlight %}

失败，调用：

{% highlight objectivec %}
- (void)messageFailed:(SKPSMTPMessage *)message error:(NSError *)error;
{% endhighlight %}


<h2 id="biaoti">标题中文乱码解决方法</h2>

这个问题解决方法就一个，使用github上的skpsmtpmessage库，不要用cocoapods，因为cocoapods上的代码是老代码，没有修复标题乱码的问题。

具体的改动是：

SKPSMTPMessage.m的sendParts里

{% highlight objectivec %}
NSData *messageData = [message dataUsingEncoding:NSASCIIStringEncoding allowLossyConversion:YES];
{% endhighlight %}
改为

{% highlight objectivec %}
NSData *messageData = [message dataUsingEncoding:NSUTF8StringEncoding allowLossyConversion:YES];
{% endhighlight %}

之前用ASCII的编码，所以会乱码。现在改为utf8就好了。

<h2 id="fujian">附件文件名中文乱码解决方法</h2>

当我在邮箱的页面点击下载附件时，附件的文件名是乱码的形式。百度了好久，没有找到skpsmtpmessage库解决这个问题的办法。唯一一个遇到同样问题的哥们在他的博客上说，虽然附件的文件名中文乱码，但是我们用英文就好了。

于是我从Java的类似类库着手，查阅附件名中文乱码的资料，很容易找到了解决方案。

用上面的附件代码，修改一下即可。修改前：


{% highlight objectivec linenos %}
NSString *vcfPath = [[NSBundle mainBundle] pathForResource:@"test" ofType:@"vcf"];
NSData *vcfData = [NSData dataWithContentsOfFile:vcfPath]; 
NSDictionary *vcfPart = [NSDictionary dictionaryWithObjectsAndKeys:@"text/directory;\r\n\tx-unix-mode=0644;\r\n\tname=\"测试.vcf\"",kSKPSMTPPartContentTypeKey,@"attachment;\r\n\tfilename=\"测试.vcf\"",kSKPSMTPPartContentDispositionKey,[vcfData encodeBase64ForData],kSKPSMTPPartMessageKey,@"base64",kSKPSMTPPartContentTransferEncodingKey,nil]; 
{% endhighlight %}

以上代码，文件名为 `测试.vcf`，当作为附件发送到目标邮箱后，用户下载邮件，附件文件名为乱码.

但是可以修改成一下形式

{% highlight objectivec linenos %}
NSString *vcfPath = [[NSBundle mainBundle] pathForResource:@"test" ofType:@"vcf"];
NSString *encodeFileName = [NSString stringWithFormat:@"=?UTF-8?B?%@?=",[[@"测试.vcf" dataUsingEncoding:NSUTF8StringEncoding] base64EncodedStringWithOptions:0]];
NSData *vcfData = [NSData dataWithContentsOfFile:vcfPath]; 
NSDictionary *vcfPart = [NSDictionary dictionaryWithObjectsAndKeys:[NSString stringWithFormat:@"text/directory;\r\n\tx-unix-mode=0644;\r\n\tname=\"%@\"",encodeFileName],kSKPSMTPPartContentTypeKey,[NSString stringWithFormat:@"attachment;\r\n\tfilename=\"%@\"",encodeFileName],kSKPSMTPPartContentDispositionKey,[vcfData encodeBase64ForData],kSKPSMTPPartMessageKey,@"base64",kSKPSMTPPartContentTransferEncodingKey,nil]; 
{% endhighlight %}

其中的重点是修改了filename，对其进行处理。

例如：

	=?UTF-8?B?5LiA5Y+35bqXNOWRqOW5tOW6hu+8jDEwMDDkuIfku7bng63plIA=?=

这个是邮件头的编码格式，B表示是Base64编码，UTF-8表示字符集的编码。

	5LiA5Y+35bqXNOWRqOW5tOW6hu+8jDEwMDDkuIfku7bng63plIA=

就是标题base64之后的结果。

使用这个就可以使附件的中文名不再乱码。



