# 起動の方法

* 初回は、先に `how-to-install.md` を読んでください。

（パワーシェルではなく）コマンドプロンプトを使う。  

```shell
# ブラウザで開く（画面の更新だけしたとき）：
pnpm dev
    # ターミナルに、以下のような表示がされたら、それをクリック。
    # `➜ Local:    http://localhost:3000/`

# デスクトップアプリとして開く（バックエンドも更新したとき）：
pnpm tauri dev
```


## 承認で build scripts 有効化

```shell
pnpm approve-builds
# esbuild を approve してください。
```
