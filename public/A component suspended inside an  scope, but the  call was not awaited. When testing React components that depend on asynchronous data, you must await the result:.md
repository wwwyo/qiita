---
title: React 19 アップグレード後に react-testing-library でテストが失敗する問題と解決策
tags:
  - React
  - react-testing-library
private: false
updated_at: '2025-05-03T14:24:10+09:00'
id: c12776d64155a8e4aac6
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

React のバージョンを 19 にアップグレードした際に、`react-testing-library` を使用しているテストが失敗する問題に遭遇しました。

この記事では、このエラーの原因と暫定的な解決策について解説する。

## 原因

詳細は不明だが、React 19 における `Suspense` の挙動変更に起因しているらしい。

特に、`use` API を利用してデータを Suspend するようなコンポーネントのテストでこの問題が発生しやすいよう。

## 暫定的な解決策

現在のところ、テストコードを `act` で Wrap すると解決する。

```tsx
import { render } from '@testing-library/react'
import { act } from 'react'
import MyComponent from './MyComponent'

test('renders component correctly', async () => {
  await act(async () => {
    render(<MyComponent />)
  })
})
```

`act` ユーティリティは、React に対してコンポーネントの更新がこれから行われることを伝え、関連する更新が処理されるまで待機するように指示する。

## まとめ

React 19 へのアップグレードに伴い、`react-testing-library` を使用したテストで `Suspense` に関連するエラーが発生する場合がある。現状では、`render` 呼び出しを `act` で Wrap すると解決する。

今後の `react-testing-library` のバージョンアップで、より根本的な解決策が提供される可能性もある。

## 参考

- Testing Library React issue: [render / waitFor seem to have issue with async component and suspense in react 19 beta #1375](https://github.com/testing-library/react-testing-library/issues/1375)
