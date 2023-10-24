# Reactの機能を色々と試すためのリポジトリ

## 実行手順
1. 動作確認したいディレクトリに移動 `cd <動作確認したいディレクトリ>`
2. `yarn install`
3. `yarn dev`実行後、`http://127.0.0.1:5173/`にアクセス


## フォームハンドリングの基本 
[対応コミット](https://github.com/zuoboo/react-advanced/commit/12df33d085992ae111da9952b69e5ef60030e2f9)
### Reactでフォームを扱う際に気を付けること
- コンポーネントからイベントを扱う際、 React が提供する Synthetic Event が適用される
- フォームの値をコンポーネントの state として持ち、値の反映には state のリフトアップを使用

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

## フォームヘルパーを使用してみる(react-hook-form)

### バリデーションライブラリについて
よく使用されるものにZodとYupがある。
```
// Yup version
const yupSchema = yup.object({
username: yup.string().required('必須項目です'),
zipcode: yup.string().max(7).matches(/\d{7}/, '7 桁の数字で入力してください'), gender: yup.mixed().oneOf(Object.keys(genderCode)),
isAgreed: yup.boolean().oneOf([true], '同意が必要です').required(),
});

// Zod version
const zodSchema = z.object({
username: z.string().min(1, { message: '必須項目です' }), zipcode: z.optional(
z.string().max(7).regex(/\d{7}/, { message: '7 桁の数字で入力してください' }) ).or(z.literal('')),
gender: z.optional(
z.enum(Object.keys(genderCode) as never).or(z.literal('')) ),
isAgreed: z.boolean().refine((val) => val, { message: '同意が必要です' }), });
```
ZodはYupと逆に項目のデフォルトがrequiredなのでそれをoptionalにする書き方や、入力値がなかった場合の判定法やエラーメッセージの与え方がかなり変則的。上記の通り、使い勝手が良いのはYup。

### react-hook-form
**`useForm`:  React Hook Formの心臓部のAPI。メインの目的はフォーム要素をReact Hook Formの管理下に置くための関数などを生成することで、引数としてひとつのオブジェクトを取り、そのプロパティで各種オプションを設定するようになっている。主なプロパティは下記参照(デフォルト値には『*』を付与)**
- `mode : onChange | onBlur | onSubmit* | onTouched | all`
  - どのタイミングでバリデーションがトリガーされるか。
- `reValidateMode : onChange* | onBlur | onSubmit`
  - エラーのある入力が再度バリデーションされるタイミング。
- `defaultValues : { elementName: elementValue }`
  - 各フォーム要素のデフォルト値。この値を適切に設定していれば型推論が効くため型引数は省略可能。
- `resolver`
  - 外部バリデーションライブラリを利用するためのカスタムリゾルバを設定する。
- `shouldUnregister : true | false*`
  - 登録していたフォーム要素がアンマウントされると同時に React Hook Form もその入力値を破棄する。
- `shouldFocusError : true* | false`
  - フォームが送信されエラーが含まれている場合に、エラーのある最初のフィールドにフォーカスする。

useForm の戻り値はさらに複雑で、そのオブジェクトの中に 15 個のプロパティが含まれている。よく使うものだけを記載
- `register`: フォーム要素をReact Hook Form の管理下に置くよう登録するための関数。 ref 、name 、onChange 、onBlurの属性に対応したプロパティを含むオブジェクトが返される。
- `formState`: フォームの各種状態を検知するためのオブジェクト。以下のプロパティが含まれる。
  - `isDarty: フォーム内容が初期値から変更されているか`
  - `dartyFields: 初期値から変更されたフォーム要素`
  - `touchedFields: 一度でもユーザーによる操作のあったフォーム要素`
  - `isSubmitted: フォームが送信されたかどうか`
  - `isSubmitSuccessful: フォームの送信が成功したかどうか`
  - `isSubmitting: フォームが送信中かどうか`
  - `submitCount: フォームが送信された回数`
  - `isValid: フォームの内容にバリデーションエラーがないかどうか`
  - `isValidating: フォームの内容をバリデーション中かどうか`
  - `errors: 各フォーム要素に対応したバリデーションエラーの内容が格納されているオブジェクト`
- `watch`: 各フォーム要素の値を監視し、変更が即時反映された値を返す関数。コンポーネントの再レンダリングを引き起こすため、パフォーマンス悪化につながらないよう使用には注意が必要。
- `handleSubmit`: 引数として関数を受け取り、フォームが送信されたときにフォームデータをその関数に渡して実行する高階関数。
- `reset`: フォームの状態をすべてリセットする関数。引数のオプション指定で部分的かつ条件をカスタマイズした リセットが可能。
- `setError`: バリデーションの結果に関わらず、手動で任意のエラーを設定できる関数。
- `clearErrors`: エラーをクリアする関数。引数で指定した要素のエラーのみクリアすることもできる。

**`RegistrationForm.tsx`ファイルの説明**

[対応コミット](https://github.com/zuoboo/react-advanced/pull/4/commits/2d582008dc2446f261311d5efed99b0165ed3fd8)

`register`関数によるフォーム要素の登録が肝でこの関数の戻り値にはrefのプロパティが含まれている。それを展開してフォーム要素の属性として与えることで、非制御コンポーネントとして対応するリアルDOMをReact Hook Formが管理下に置く。
```
// register箇所を抜粋

<Input {...register('username')} />
<Input maxLength={7} {...register('zipcode')} />
<Select placeholder="性別を選択..." {...register('gender')}>
<Checkbox {...register('isAgreed')}>
```

**参考になるドキュメント**
- [APIリファレンス](https://react-hook-form.com/docs)
- [registerについて](https://react-hook-form.com/docs/useform/register)
- [クライアント側の「フォーム検証](https://developer.mozilla.org/ja/docs/Learn/Forms/Form_validation)

### Yupでスキーマバリデーション
**`ValidRegistrationForm.tsx`ファイルの説明**

[対応コミット](https://github.com/zuoboo/react-advanced/pull/5/commits/0811b29647017226e13d6af1d2db568206a91092)

`useForm`の引数オプションに`resolver`としてYupのカスタムリゾルバを設定。それから戻り値で`formState`を受け取り、さらにその中の`errors`をピックアップ。→その中にスキーマで検証した入力値のバリデーションエラー情報が入る。
