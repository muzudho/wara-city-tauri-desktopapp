# ジャーナル 2025-07

## [2025-07-27_Sun]

パレットを作りたいので、ドラッグできるパネルの作り方を調べてみる。  

[vue-draggable-resizable](https://github.com/mauricius/vue-draggable-resizable?tab=readme-ov-file)  

```shell
pnpm add vue-draggable-resizable
```

追加：  
```ts
<script lang="ts">
import VueDraggableResizable from 'vue-draggable-resizable';
import 'vue-draggable-resizable/style.css';
```

## [2025-07-29_Tue] 

### tsconfig.json に追加。import 文で types フォルダーをインポートするために

追加部分：  

```json
{
    "compilerOptions": {
        "baseUrl": "src",
        "paths": {
            "@/*": ["./*"]
        },
    },
    "vueCompilerOptions": {
        "extensions": [".ts", ".tsx", ".js", ".jsx", ".vue"] // [2025-07-29_Tue] .vueを明示。 import 文で @ を使えるようにするために
    }
}
```

## [2025-07-30_Wed]

📄 `tsconfig.json` が JSON の配列やブーリアンに対してエラーを出しているわけ分からない状態。  

```shell
npm list typescript
    // エラーが出てる
```

📁 node_modules フォルダーを削除。  

```shell
pnpm install
```

エラーが取れた。
