---
title: Template Literal Typesについて調べてみた
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

Typescript Template Literal Types の勉強メモ。
公式のドキュメントを自分なりにまとめたものです。

- https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html

# Template Literal Types

## 一言で

既存の型から文字列リテラル型を作り出せる

## 構文

```ts
type World = 'world'
type Greeting = `hello ${World}`
// type Greeting = "hello world"
```

union や intersection と組み合わせることで、より複雑な型を作ることができる

```ts
type TLD = 'jp' | 'com'
type SLD = 'example' | 'test'
type Domain = `${SLD}.${TLD}`
// type Domain = "example.jp" | "example.com" | "test.jp" | "test.com"
```

## EventEmitter の例

:::note
EventEmitter クラスの(on)[https://nodejs.org/docs/latest/api/events.html#events_emitter_on_eventname_listener]メソッドに近いもの
:::

次の passedObject のプロパティの変更を検知するリスナーメソッドを定義したい時の例
次の制約を満たすような makeWatchedObject 関数型を実装する

1. メソッド名は on
2. on の引数には`<プロパティ名>Changed` と戻り値が void であるコールバック関数を受け取る

```ts
const passedObject =  makeWatchedObject({
  firstName: 'Saoirse',
  lastName: 'Ronan',
  age: 26,
})

passedObject.on('firstNameChanged', (v) => {...})
```

こうすると、passedObject に指定のシグネチャに沿った on メソッドが生える

```ts
type PropEventSource<Type> = {
    on(
      eventName: `${string & keyof Type}Changed`,
      callback: (newValue: any) => void
    ): void;
};

declare function makeWatchedObject<Type>(obj: Type): Type & PropEventSource<Type>;

// 推論される
passedObject.on(eventName: "firstNameChanged" | "lastNameChanged" | "ageChanged", callback: (newValue: any) => void): void
```

## Inference with Template Literals

さらに、Template Literal Types を使って、コールバック関数の引数の型を推論することができる

```ts
type PropEventSource<Type> = {
    on<Key extends string & keyof Type>(
        eventName: `${Key}Changed`,
        callback: (newValue: Type[Key]) => void
    ): void;
};

// 推論される
passedObject.on<Key>(eventName: `${Key}Changed`, callback: (newValue: {
    firstName: string;
    lastName: string;
    age: number;
}[Key]) => void): void
```

## Intrinsic String Manipulation Types

テンプレートリテラルを使った文字列型が ts に組み込まれて提供されている

- Uppercase
- Lowercase
- Capitalize
- Uncapitalize

余談、Uppercase を作ってみる

```ts
type _Uppercase<T extends {}> = T extends `${infer First}${infer Rest}`
  ? `${UppercaseChar<First>}${Rest}`
  : T

type UppercaseChar<Char extends string> = Char extends 'a'
  ? 'A'
  : Char extends 'b'
  ? 'B'
  : Char extends 'c'
  ? 'C'
  : Char extends 'd'
  ? 'D'
  : Char extends 'e'
  ? 'E'
  : Char extends 'f'
  ? 'F'
  : Char extends 'g'
  ? 'G'
  : Char extends 'h'
  ? 'H'
  : Char extends 'i'
  ? 'I'
  : Char extends 'j'
  ? 'J'
  : Char extends 'k'
  ? 'K'
  : Char extends 'l'
  ? 'L'
  : Char extends 'm'
  ? 'M'
  : Char extends 'n'
  ? 'N'
  : Char extends 'o'
  ? 'O'
  : Char extends 'p'
  ? 'P'
  : Char extends 'q'
  ? 'Q'
  : Char extends 'r'
  ? 'R'
  : Char extends 's'
  ? 'S'
  : Char extends 't'
  ? 'T'
  : Char extends 'u'
  ? 'U'
  : Char extends 'v'
  ? 'V'
  : Char extends 'w'
  ? 'W'
  : Char extends 'x'
  ? 'X'
  : Char extends 'y'
  ? 'Y'
  : Char extends 'z'
  ? 'Z'
  : Char
```

余談 2
Uppercase は ts では以下のように実装されている

```ts
type Uppercase<S extends string> = intrinsic
```

intrinsic はコンパイラの内部実装として隠蔽されていることを示す

詳細: https://zenn.dev/uhyo/articles/typescript-intrinsic
