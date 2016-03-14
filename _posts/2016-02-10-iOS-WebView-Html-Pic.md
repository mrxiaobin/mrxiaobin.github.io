---
layout: post
title: iOS WebView加载HTML文件时大图片处理
date: 2016-3-10
categories: blog
tags: [学习笔记,iOS]
description: 
---

在做的app有一块是加载一个网页，从服务器拿到的是HTML字符串然后放到HTML里边显示，发现有部分图片太大了，只能看到一部分，Google了下，使用以下JS脚本，设定一个最大的宽度，当图片宽度超过这个上限值的时候，等比例缩放，
```objectivec
-(void)webViewDidFinishLoad:(UIWebView *)webView {
    NSString *script = [NSString stringWithFormat:
                        @"var script = document.createElement('script');"
                        "script.type = 'text/javascript';"
                        "script.text = \"function ResizeImages() { "
                        "var img;"
                        "var maxwidth=%f;"
                        "for(i=0;i <document.images.length;i++){"
                        "img = document.images[i];"
                        "if(img.width > maxwidth){"
                        "img.width = maxwidth;"
                        "}"
                        "}"
                        "}\";"
                        "document.getElementsByTagName('head')[0].appendChild(script);", DeviceWidth - 20];
    [webView stringByEvaluatingJavaScriptFromString: script];
    [webView stringByEvaluatingJavaScriptFromString:@"ResizeImages();"];
}
```