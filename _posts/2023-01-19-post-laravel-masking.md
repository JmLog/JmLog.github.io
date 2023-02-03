---
title: "[Laravel] 글자 마스킹 처리하기"
layout: single
categories: laravel
toc: true
toc_label: "목차"
toc_icon: "fas fa-list-ul"
author_profile: false
sidebar:
    nav: "docs"
---
*** 

홈페이지를 운영하다 보면 고객들의 이름, 주민번호, 핸드폰 번호등등을 마스킹 처리해서
표출해줘야 하는 상황이 있습니다.

Laravel에서 제공하는 Str::mask 라는 매서드 이용하면 됩니다.

## Str::mask 사용법
Str::mask 매서드 내부를 들여다 보면 4가지의 인자를 사용할 수 있습니다.

1. *첫번째 인자는 마스킹 할 `$string`*
2. *두번째 인자는 마스킹 할 때 사용되는 `$character`*
3. *세번째 인자는 마스킹을 시작할 `$index`*
4. *네번째 인자는 마스킹 수를 제한하는 `$length`*
5. *다섯번째 인자는 `$encoding`*

```php
mask($string, $character, $index, $length = null, $encoding = 'UTF-8')
```

## 이름 가운데 마스킹 처리하기
이름은 2글자인 경우와 3글자 이상인 경우를 예외처리 하면서 작업한다.
해당 코드는 2글자는 자동으로 뒤에 한 글자만 마스킹 처리된다.

```php
$name = '백지민'; //# 백지미니
//# 기본 마스킹 변수 설정
$index = '1';
$length = '1';

$cnt = mb_strlen($name);
//# 3글자 이상일 경우 (총 글자 수) - (마스킹이 필요없는 수 = 첫번째/마지막) 
if($cnt > 3) $length = $cnt - 2;

$res = Str::mask($name, '*', $index, $length);
//# $res = '백*민'
//# $res = '백**니'
```

## 핸드폰 번호 마스킹 처리하기
핸드폰 번호는 하이픈(’-’)이 포함된 경우와 포함되지 않은 경우만 예외처리 하면서 작업한다.

```php
$phone = '01000000000'; //# 010-0000-0000
//# 기본 마스킹 변수 설정
$index = '3';
$length = '4';

//# 핸드폰 번호에 하이픈이 포함된 경우
$phonePattern = '/^(010|011|016|017|018|019)-[0-9]{3,4}-[0-9]{4}$/';
if(preg_match($phonePattern, $phone, $match)) $index = '4';

$res = Str::mask($phone, '*', $index, $length);
//# $res = '010****0000'
//# $res = '010-****-0000'
```

## 이메일 3글자 제외 마스킹 처리하기
이메일은 구분자를(‘@’) 제외한 앞 3글자만 마스킹 처리한다.

```php
$email = 'wmlals0002@gmail.com';
//# 기본 마스킹 변수 설정
$index = '3';

//# 이메일 글자를 @로 배열화 시킨다
$emailArr = explode('@', $email);
//# $emailArr = [0 => ['wmlals0002'], 1 => ['@'], 2 => ['gmail.com']]
//# (@ 이전 문자의 글자수) - (마스킹이 필요 없는 수 = $index)
$length = strlen($emailArr[0]) - $index;

$res = Str::mask($email, '*', $index, $length);
//# $res = 'wml*******@gmail.com'
```