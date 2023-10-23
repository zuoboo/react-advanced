# react-advanced

reactの機能を色々と試すためのリポジトリ

## 実行手順
1. 動作確認したいディレクトリに移動 `cd <動作確認したいディレクトリ>`
2. `yarn install`
3. `yarn dev`実行後、`http://127.0.0.1:5173/`にアクセス


## フォームハンドリングの基本 
https://github.com/zuoboo/react-advanced/commit/12df33d085992ae111da9952b69e5ef60030e2f9
### Reactでフォームを扱う際に気を付けること
- コンポーネントからイベントを扱う際、 React が提供する Synthetic Event が適用される
- フォームの値をコンポーネントの state として持ち、値の反映には state のリフトアップを使

**`RegistrationForm.tsx`ファイルの説明**
- フォーム全体の値を`FormData`型のオブジェクトにして state として持ち回る。
- `handleChange`で受け取ったイベントのターゲットの種類によって抽出する値を振り分け、それを`setFormData`に渡して、stateを更新する。
- 値を state で管理されるフォーム要素のことを『制御されたコンポーネント(Controlled Components)』と呼ぶ。
  - Reactではフォームの実装はこの制御されたコンポーネントを使うことが推奨されている。
- `preventDefault`について
  - そのイベント本来の振る舞いを阻止している。フォームをsubmitすると、action属性で指定されているURL、それがなければ現在のURLにフォームの値がPOSTで送られてページが遷移してしまう。SPAでそれをやるとページがトップルートからレンダリングし直しになり、stateもすべてクリアされてしまう。それをさせないためにこの1行を入れている。
- 登場するReactの型
  - `SyntheticEvent`: イベントの基本イベント。イベントタイプが不明な場合に使用する。(Cheatsheetの内容をそのまま翻訳)
  - `ChangeEvent`: input、select、textarea要素の値を変更する。(Cheatsheetの内容をそのまま翻訳)

**参考になるドキュメント等**
- [@types/reactの型定義](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/react/index.d.ts)
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/forms_and_events/)
