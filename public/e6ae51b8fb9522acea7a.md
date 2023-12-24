---
title: 'Laravel:emptyの罠'
tags:
  - PHP
  - laravel5
private: false
updated_at: '2022-03-04T23:34:58+09:00'
id: e6ae51b8fb9522acea7a
organization_url_name: null
slide: false
ignorePublish: false
---
emptyの使い方、完全に勘違いしていた。

Laravelのcollectionに対して空かどうかの判定はisEmpty()を使う。
普通にforeachでイテレートできるのでempty(collect([]))で判定できるもんだと思っていたら、全部trueになってた。

[Laravel 5.8 コレクション](https://readouble.com/laravel/5.8/ja/collections.html)
>Illuminate\Support\Collectionクラスは配列データを操作するための、書きやすく使いやすいラッパーです。

例えばこのような例や
```php
$collection = collect([]);
while(!empty($collection)) {
    // do
}
// => 無限ループする
```

ガード節などでうまくいかない。
```php
if (!$collection) {
    return; // => ここに入ることはない。
}
// do
```

友人の言葉を借りると、
```php
empty($var)は !isset($var) || $var == falseのこと
$var = collect([])であるとすると、
値であるcollect([]) を代入しているから
!isset($var) // === false 
($var == false) // === false
より常にtrueになる
```

ちなみにisEmptyは内部でemptyを呼び出している。
```collection.php
public function __construct($items = [])
{
    $this->items = $this->getArrayableItems($items);
}

protected function getArrayableItems($items)
{
    if (is_array($items)) {
        return $items;
    } elseif ($items instanceof self) {
        // collection => array
        return $items->all();
    }  //...
}

public function isEmpty()
{
    return empty($this->items);
}
```

# まとめ
collection(というか自作クラス）にはemptyは使えない。
emptyやissetだけだと意図が伝わりにくいので、is_nullや比較演算子などで書いた方が良い気がした。
あとはLarastan入れれば教えてくれる見たい。
emptyの諸々の判定はマニュアルを見てください。

https://www.php.net/manual/ja/types.comparisons.php


そういえば、phpってなんでコレクションないんだろう。
