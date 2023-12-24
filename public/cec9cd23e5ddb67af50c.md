---
title: 'Laravel: with(eager load) Tips'
tags:
  - laravel5
private: false
updated_at: '2022-03-03T03:25:49+09:00'
id: cec9cd23e5ddb67af50c
organization_url_name: null
slide: false
ignorePublish: false
---
# with
- リレーションのメソッドをwith()内に書くことでeager loadができる。

```User.php
$users = User::with('posts')->get();

// リレーションメソッド
public function posts() {
    return $this->hasMany(Post::class, 'user_id', 'id');
}
```

- リレーション先のobjectには動的プロパティとしてアクセスできる。

```php
foreach ($users as $user) {
    $user->posts;
}
```

- リレーションのネストや複数記入も可、その場合は一つずつクエリが発行されている。

```php
$users = User::with([
    'posts.tag',
    'address', 
])->get();
```

```sql
select * from `posts` where `posts`.`user_id` in (userのids) and `posts`.`deleted_at` is null
select * from `tags` where `tags`.`post_id` in (postのids) and `tags`.`deleted_at` is null
-- //
```

- 上記でのクエリからもわかるが「deleted_at is null」がデフォルトで指定。
便利だが、table名のaliasをつけた時に元の名前でdeleted_atを呼び出そうとしてエラーが出る。
->回避方法あるのか?

```php
// error
User::from('users as u')->get();
```

- 複数に分けても可
```php
$query = User::query();
$query->with('posts');
$query->with('address');
```

- カラムを選択できる
    - リレーション関係の外部キーは必須
    - 半角に注意
```php
$users = User::with('posts:user_id,title');

// 半角開けるとエラー
$users = User::with('posts:user_id, title');
```


 





