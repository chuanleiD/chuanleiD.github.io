---
layout: post
title: "视频网站视频获取_以bilibili为例"
date:   2024-8-16
tags: [web, Api, video]
comments: true
author: chenliang
---

本篇文档以bilibili为例记录了从视频网站获取本地视频的方法。

<!-- more -->

[TOC]

## 一、一般方法

1、获取video（视频），audio（音频）的API：

首先在访问视频网站例如：https://www.bilibili.com/video/BV1xx411c7mu/ 时抓包，找到含有 m4s 的两项。

其中，有5位数字的是音频`audioURL`，6位数字的是视频`vidoeURL`。

<img src="https://notes.sjtu.edu.cn/uploads/upload_129ab56b69ccfb143403c6430a72332b.png" alt="image-20240902010854616" style="zoom:50%;" />

2、由此得到请求方法：

```go
// Create HTTP client
	client := &http.Client{}

	// Create HTTP request with the given URL
	req, err := http.NewRequest("GET", url, nil) // vidoeURL或audioURL
	if err != nil {
		return fmt.Errorf("failed to create request: %v", err)
	}

	// Set headers
	req.Header.Set("Referer", referer) // 视频链接："https://www.bilibili.com/video/BV1xx411c7mu/"
	req.Header.Set("User-Agent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/127.0.0.0 Safari/537.36")

```

返回的即使视频或音频内容，使用`io.copy`保存到本地文件中即可。

## 二、进阶的方法
