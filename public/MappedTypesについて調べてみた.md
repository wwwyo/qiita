---
title: MappedTypesについて調べてみた
tags:
  - 'Typescript'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

Typescript MappedTypes の勉強メモ。
公式のドキュメントを自分なりにまとめたものです。

- https://typescriptbook.jp/reference/type-reuse/mapped-types
- https://www.typescriptlang.org/docs/handbook/2/mapped-types.html

# Mapped Types

## 一言で

既存の型から新しい Object 型を定義する構文

## 構文

```tsx
type MappedTypes = {
  [P in T]: U
}
```

### **具体例**

Person のプロパティを Readonly にした型を作る

```tsx
type Person = {
  name: string
  age: string
}

// Mapped Types
type ReadonlyPerson1 = {
  readonly [K in keyof Person]: Person[K]
}

// ReadonlyPerson1と等しい
type ReadonlyPerson2 = {
  readonly name: string
  readonly age: string
}
```

:::note info
keyof を用いた反復処理と一緒に使われることが多い
:::

:::note warn
[Index 型](https://typescriptbook.jp/reference/values-types-variables/object/index-signature)とは明確に違う。
Index 型はプロパティ名を指定しない。MappedTypes はより厳密に定義したもの。

```tsx
type IndexSignature = {
  [K: string]: number
}
```

:::

</aside>

## 何が良いの?

- 既存の型から生成できるため、コードの重複を減らせる
- 厳密にプロパティ名を指定できるため型安全性が向上する

## 修飾子

### **readonly**

プロパティを読み取り専用に変換

```tsx
type Person = {
  name: string
  age: string
}

type ReadonlyPerson = {
  readonly [K in keyof Person]: Person[K]
}

// {
//    readonly name: string;
//    readonly age: string;
// }
```

### オプショナル

プロパティをオプショナルに変換

```tsx
type OptionalPerson = {
  [K in keyof Person]?: Person[K]
}

// {
//    name?: string | undefined;
//    age?: string | undefined;
// }
```

### マイナス

修飾子を消せる

```tsx
type RemoveReadonlyPerson = {
  -readonly [K in keyof ReadonlyPerson]: ReadonlyPerson[K]
}

// {
//    name: string;
//    age: string;
// }
```

### プラス

実は省略されていただけ

```tsx
type ReadonlyPerson = {
  readonly [K in keyof Person]: Person[K]
}

type PlusReadonlyPerson = {
  +readonly [K in keyof Person]: Person[K]
}

// {
//    readonly name: string;
//    readonly age: string;
// }
```

### `as` による再マッピング

キーを再マップできる

```tsx
type GetterPerson = {
  [K in keyof Person as `get${Capitalize<K>}`]: string
}

// {
//    getName: () => Person["name"];
//    getAge: () => Person["age"];
// }
```

## ⚠️ 注意

### プロパティを追加できない

```tsx
type PersonWithWeight = {
  [K in keyof Person]: Person[K]
  // ^ A mapped type may not declare properties or methods.
}
```

intersection を使う

```tsx
type PersonWithWeight = {
  [K in keyof Person]: Person[K]
} & { weight: number }
```

### Interface では使えない

理由は知らない。interface は素朴にオブジェクトを表現することに徹しているのかも。

```tsx
interface ReadonlyPerson {
  readonly [K in keyof Person]: Person[K]
}
```

## なぜこの名前?

どうやら Map は数学で”写像”を意味するらしい。
fyi: https://zenn.dev/axoloto210/articles/advent-calender-2023-day22

「なんすか写像って」ときかれた際は、Mapped Types のリンクを送ると平和になるかもしれない。
