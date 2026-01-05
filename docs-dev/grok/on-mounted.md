Tauri + Vue で、HTMLドキュメントの読み込み完了時に処理が走るようにするには、どう書けばいいんだぜ（＾～＾）？


Tauri + VueでHTMLドキュメントの読み込み完了時に処理を実行するには、Vueのライフサイクルフックを利用するのが一般的だぜ！具体的には、`onMounted`フックを使うと、コンポーネントがDOMにマウントされたタイミングで処理を走らせることができるよ（＾～＾）。Tauri特有の処理を絡める場合も、このタイミングでTauriのAPIを呼び出すのがスムーズだ。

以下に、具体的な実装例を示すぜ：

### コード例
```vue
<script setup>
import { onMounted } from 'vue';
import { invoke } from '@tauri-apps/api/core';

// ドキュメント読み込み完了時に実行される処理
onMounted(() => {
  console.log('ドキュメントの読み込みが完了したぜ！');

  // TauriのAPIを呼び出す例（例: バックエンドのコマンドを実行）
  invoke('my_command', { param: 'hoge' })
    .then((response) => {
      console.log('Tauriからの応答:', response);
    })
    .catch((error) => {
      console.error('エラーだぜ:', error);
    });
});
</script>

<template>
  <div>
    <h1>Tauri + Vue サンプル</h1>
    <p>ドキュメントが読み込まれたらコンソールを確認してね（＾～＾）</p>
  </div>
</template>
```

### 解説
1. **Vueの`onMounted`フック**:
   - `<script setup>`構文を使って、`onMounted`をインポート。
   - `onMounted(() => {...})`内に、DOM読み込み完了後に実行したい処理を書く。
   - これで、コンポーネントがマウントされた（HTMLがレンダリングされた）タイミングで処理が走るよ。

2. **TauriのAPI呼び出し**:
   - `@tauri-apps/api/core`から`invoke`をインポートして、Rust側で定義したコマンド（例: `my_command`）を呼び出す。
   - `invoke`は非同期処理なので、`then`で成功時の処理、`catch`でエラー処理を記述。

3. **Tauriのバックエンド設定**:
   - `invoke`で呼び出す`my_command`は、TauriのRust側（`src-tauri/src/main.rs`など）で定義しておく必要があるぜ。例えば：
     ```rust
     #[tauri::command]
     fn my_command(param: &str) -> String {
         format!("Hello, {}!", param)
     }
     ```
   - `tauri.conf.json`でコマンドが正しく登録されていることも確認してね。

### 注意点
- **Tauriのセットアップ**: Tauriのプロジェクトが正しく設定されてないと、`invoke`が動かないぜ。`tauri dev`で開発環境を起動して、動作確認しよう。
- **非同期処理**: `invoke`はPromiseを返すから、非同期処理（`async/await`や`then/catch`）で扱うのが大事。
- **ライフサイクル**: もし特定のタイミング（例えばウィンドウの読み込み完了）で処理を走らせたい場合は、Tauriの`@tauri-apps/api/window`モジュールのイベントリスナー（`appWindow.on...`）を組み合わせるのもアリだぜ。

### 補足: ウィンドウの読み込み完了を検知したい場合
もし「HTMLドキュメントの読み込み完了」じゃなくて、Tauriのウィンドウ全体の読み込み完了を検知したい場合は、以下のように`@tauri-apps/api/window`を使う方法もある：

```vue
<script setup>
import { onMounted } from 'vue';
import { appWindow } from '@tauri-apps/api/window';

onMounted(() => {
  // ウィンドウの読み込み完了を検知
  appWindow.onLoad(() => {
    console.log('Tauriウィンドウの読み込みが完了したぜ！');
  });
});
</script>
```

### まとめ
- 基本は`onMounted`で十分。コンポーネントがDOMにマウントされたタイミングで処理が走る。
- TauriのAPIを呼び出すなら、`invoke`を使ってRust側のコマンドと連携。
- 必要に応じて、`@tauri-apps/api/window`でウィンドウイベントをハンドル。

何か他に知りたいことがあれば、気軽に聞いてくれよ（＾～＾）d
