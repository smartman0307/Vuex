# ミューテーション

Vuex のミューテーションは基本的にイベントです。各ミューテーションは**名前**と**ハンドラ**を持ちます。ハンドラ関数は常に全体のステートツリーを第1引数として取得します:

``` js
import Vuex from 'vuex'

const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    INCREMENT (state) {
      // 変異するステート
      state.count++
    }
  }
})
```

ミューテーション名に対して全て大文字を使用するのは、容易にアクションと区別できるようにするための規則です。

直接ミューテーションハンドラ呼び出すことはできません。ここでのオプションは、よりイベント登録のようなものです。"`INCREMENT` イベントがディスパッチされるとき、このハンドラは呼ばれます。"ミューテーションハンドラを起動するためには、ミューテーションイベントをディスパッチする必要があります:

``` js
store.dispatch('INCREMENT')
```

### 引数によるディスパッチ

引数に沿って渡すことも可能です:

``` js
// ...
mutations: {
  INCREMENT (state, n) {
    state.count += n
  }
}
```
``` js
store.dispatch('INCREMENT', 10)
```

ここの `10` は `state` に続く第2引数としてミューテーションハンドラに渡されます。任意の追加引数と同じです。これら引数は、特定のミューテーションに対して**ペイロード**と呼ばれています。

### Vue のリアクティブなルールに従うミューテーション

Vuex store のステートは Vue によってリアクティブになっているので、ステートを変異するとき、ステートを監視している Vue コンポーネントは自動的に更新されます。これはまた、Vuex のミューテーションは、純粋な Vue で動作しているとき、同じリアクティブな警告の対象となっているのを意味します:

1. 前もって全て望まれるフィールドによって、あなたの store の初期ステートを初期化することを好みます

2. 新しいプロパティをオブジェクトに追加するとき、いずれか必要です:

  - `Vue.set(obj, 'newProp', 123)` を使用または -

  - 全く新しいものでオブジェクトを置き換える。例えば、stage-2 の [object spread syntax](https://github.com/sebmarkbage/ecmascript-rest-spread) を使用して、このようにそれを書くことができます:

  ``` js
  state.obj = { ...state.obj, newProp: 123 }
  ```

### ミューテーション名に対して定数を使用

ミューテーション名には定数を使用することが一般的です。コードに対して linter のようなツールの長所を利用することができ、そして、単一ファイルに全ての定数を設定することは、あなたの協力者にミューテーションがアプリケーション全体で可能であるかが一目見ただけで理解できるビューを得ることができます:

``` js
// mutation-types.js
export const SOME_MUTATION = 'SOME_MUTATION'
```

``` js
// store.js
import Vuex from 'vuex'
import { SOME_MUTATION } from './mutation-types'

const store = new Vuex.Store({
  state: { ... },
  actions: { ... },
  mutations: {
    // 定数を関数名として使用できる ES2015 の算出プロパティ (computed property) 名機能を使用できます
    [SOME_MUTATION] (state) {
      // 変異するステート
    }
  }
})
```

定数を使用するかどうか大部分が好みであり、それは多くの開発者による大規模アプリケーションで役に立ちますが、好きではないならば、それは完全にオプションです。

### アクションの上へ

これまでのところ、`store.dispatch` の手動呼び出しによってミューテーションをトリガしていました。これは実行可能なアプローチですが、実際には私たちのコンポーネントのコードでこれを行うことはほとんどありません。ほとんどの時間、[アクション](actions.md) を呼び出し、非同期データフェッチングのようなより複雑なロジックをカプセル化することができます。

また、全てのミューテーションハンドラは**同期**でなければならないという、1つ重要なルールを覚えておきましょう。任意の非同期な操作はアクションに属しています。