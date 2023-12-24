---
title: 'php: 参照渡し'
tags:
  - PHP
private: false
updated_at: '2022-03-08T14:49:16+09:00'
id: bd93faa39dbba89b7a27
organization_url_name: null
slide: false
ignorePublish: false
---
引数の変数に&を付けるとリファレンス渡しになる。
グローバル変数のように扱えるが可読性は悪くなる。

https://www.php.net/manual/ja/language.references.pass.php

## 値に参照
```php
function increment(&$var)
{
    $var++;
}

$a = 0;
increment($a);
echo $a; // 1

increment($a);
echo $a; //2
```

## また関数をリファレンス渡しすることもできる
```php
function increment(&$var)
{
    $var++;
    return $var;
}

function &reference()
{
    $a = 0;
    return $a;
}

$a = increment(reference());
echo $a; // 1
```

## 配列やオブジェクトの参照
少し面白い。
配列は参照できないが、objectは参照渡しできるみたい。

```php
function increment($arr)
{
    foreach ($arr as &$i) {
        $i++;
    }
}

$arr = [1,2,3,4,5];
increment($arr);
print_r($arr); // [1,2,3,4,5]

$obj = (object)$arr;
increment($obj);
print_r($obj); // [2,3,4,5,6]
```
