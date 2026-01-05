# インストール方法

Type Scrpt/Java Script 等のモジュールをまだインストールしていなければ、以下のコマンドを打鍵してください：  

（パワーシェルではなく）コマンドプロンプトを使う。  

```shell
pnpm add -D vite @vitejs/plugin-vue
pnpm add -D @tauri-apps/cli

# VSCode に TypeScript の型定義を教える
pnpm add -D @types/node

# Tauri の NPM/Rust 両方のバージョンの統一
pnpm add -D @tauri-apps/cli@latest @tauri-apps/api@latest @tauri-apps/plugin-opener@latest @tauri-apps/plugin-dialog@latest @tauri-apps/plugin-fs@latest

pnpm install
```
