---
title: "[Laravel] 이미지 URL 유효성 체크"
layout: single
categories: laravel
toc: true
toc_label: "목차"
toc_icon: "fas fa-list-ul"
author_profile: false
sidebar:
    nav: "docs"
---

### curl을 활용한 url 유효성 체크

```php
$url = "Your Img Url";

$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, $url);
curl_setopt($ch, CURLOPT_NOBODY, 1);
curl_setopt($ch, CURLOPT_FAILONERROR, 1);

$res = curl_exec($ch);
//# 결과: TRUE, FALSE
```