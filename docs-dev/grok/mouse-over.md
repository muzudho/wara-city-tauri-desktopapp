例えば <Tile :srcLeft="10" :srcTop="20"/> というタグがあって、そこにマウスカーソルを重ねると、10 と 20 を受け取るイベントハンドラー関数みたいなものってどうやって書くんだぜ（＾～＾）？

おっ、Vue3で`<Tile>`コンポーネントにマウスカーソルを重ねたときに、`:srcLeft`や`:srcTop`の値（例えば`10`や`20`）を受け取って処理するイベントハンドラーを作りたいんだな？ バッチリ分かったぜ（＾～＾）！ TypeScriptを使って、型安全に実装する方法を教えるよ。連想配列（`Record`）を使ってパネルのデータを管理し、マウスオーバー（`mouseover`）イベントで`srcLeft`と`srcTop`をハンドリングする例を示すぜ！

### 実装例：マウスオーバーで`srcLeft`と`srcTop`を受け取る
以下は、`<Tile>`コンポーネントにマウスカーソルを重ねると、`srcLeft`と`srcTop`をイベントハンドラーで受け取って処理するコードだ。連想配列でパネルを管理し、TypeScriptで型安全に書くよ。

#### 親コンポーネント（`DraggablePanel.vue`）
```vue
```vue
<template>
  <div class="panel-container">
    <Tile
      v-for="(panel, key) in panels"
      :key="key"
      :srcLeft="panel.srcLeft"
      :srcTop="panel.srcTop"
      @mouseover="handleMouseOver(key, panel.srcLeft, panel.srcTop)"
    />
    <p>最後にホバーしたパネル: {{ lastHovered }}</p>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import Tile from './Tile.vue';

// 連想配列の値の型
interface PanelData {
  srcLeft: number;
  srcTop: number;
}

// 連想配列の型
type PanelMap = Record<string, PanelData>;

export default defineComponent({
  name: 'DraggablePanel',
  components: {
    Tile
  },
  data() {
    return {
      panels: {
        panel1: { srcLeft: 10, srcTop: 20 },
        panel2: { srcLeft: 30, srcTop: 40 },
        panel3: { srcLeft: 50, srcTop: 60 }
      } as PanelMap,
      lastHovered: '' // ホバーしたパネルの情報を表示
    };
  },
  methods: {
    handleMouseOver(key: string, srcLeft: number, srcTop: number) {
      this.lastHovered = `キー: ${key}, srcLeft: ${srcLeft}, srcTop: ${srcTop}`;
      console.log(`Hovered on panel: ${key}, srcLeft: ${srcLeft}, srcTop: ${srcTop}`);
    }
  }
});
</script>

<style scoped>
.panel-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 16px;
  padding: 16px;
}
</style>
```
```

#### 子コンポーネント（`Tile.vue`）
```vue
```vue
<template>
  <div class="tile" @mouseover="$emit('mouseover', $event)">
    <div>Tile (srcLeft: {{ srcLeft }}, srcTop: {{ srcTop }})</div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';

export default defineComponent({
  name: 'Tile',
  props: {
    srcLeft: {
      type: Number,
      required: true
    },
    srcTop: {
      type: Number,
      required: true
    }
  },
  emits: ['mouseover']
});
</script>

<style scoped>
.tile {
  width: 200px;
  height: 150px;
  background: #f0f0f0;
  border: 1px solid #999;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}
</style>
```
```

### 解説
1. **型定義**：
   - `PanelData`：各パネルのデータ（`srcLeft`と`srcTop`）を`number`型で定義。
   - `PanelMap`：`Record<string, PanelData>`で、キーが文字列、値が`PanelData`型の連想配列を定義。
   - `srcLeft`と`srcTop`を数値型（`number`）として扱ってるけど、必要なら`string`に変更可能。

2. **テンプレート**：
   - `<Tile>`を`v-for`でループし、連想配列`panels`の各エントリを描画。
   - `@mouseover="handleMouseOver(key, panel.srcLeft, panel.srcTop)"`で、マウスオーバー時に親コンポーネントの`handleMouseOver`メソッドを呼び出し、`key`, `srcLeft`, `srcTop`を渡す。

3. **イベントハンドリング**：
   - `Tile.vue`：`@mouseover="$emit('mouseover', $event)"`で、マウスオーバーイベントを親に伝える。
   - `DraggablePanel.vue`：`handleMouseOver`メソッドで`key`, `srcLeft`, `srcTop`を受け取り、例えば`lastHovered`に情報を保存したり、コンソールに出力。
   - 出力例（コンソール）：
     ```
     Hovered on panel: panel1, srcLeft: 10, srcTop: 20
     ```

4. **データと表示**：
   - `panels`：連想配列でパネルのデータを管理。例として`panel1`, `panel2`, `panel3`を初期化。
   - `lastHovered`：ホバーしたパネルの情報を表示するために追加。UIで確認しやすいぜ。

5. **スタイル**：
   - `.panel-container`：グリッドレイアウトでパネルを配置。
   - `.tile`：各`<Tile>`にカーソルを合わせやすいように、`cursor: pointer`を設定。

### 動的追加の例
新しいパネルを追加して、ホバーイベントも機能するようにする例：

```vue
```vue
<template>
  <div class="panel-container">
    <Tile
      v-for="(panel, key) in panels"
      :key="key"
      :srcLeft="panel.srcLeft"
      :srcTop="panel.srcTop"
      @mouseover="handleMouseOver(key, panel.srcLeft, panel.srcTop)"
    />
    <button @click="addPanel">新しいパネルを追加</button>
    <p>最後にホバーしたパネル: {{ lastHovered }}</p>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import Tile from './Tile.vue';

interface PanelData {
  srcLeft: number;
  srcTop: number;
}

type PanelMap = Record<string, PanelData>;

export default defineComponent({
  name: 'DraggablePanel',
  components: {
    Tile
  },
  data() {
    return {
      panels: {
        panel1: { srcLeft: 10, srcTop: 20 },
        panel2: { srcLeft: 30, srcTop: 40 },
        panel3: { srcLeft: 50, srcTop: 60 }
      } as PanelMap,
      lastHovered: ''
    };
  },
  methods: {
    handleMouseOver(key: string, srcLeft: number, srcTop: number) {
      this.lastHovered = `キー: ${key}, srcLeft: ${srcLeft}, srcTop: ${srcTop}`;
      console.log(`Hovered on panel: ${key}, srcLeft: ${srcLeft}, srcTop: ${srcTop}`);
    },
    addPanel() {
      const newKey = `panel${Object.keys(this.panels).length + 1}`;
      this.panels[newKey] = {
        srcLeft: Object.keys(this.panels).length * 20 + 10,
        srcTop: Object.keys(this.panels).length * 20 + 20
      };
    }
  }
});
</script>

<style scoped>
.panel-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 16px;
  padding: 16px;
}
</style>
```
```

- `addPanel`：新しいパネルを連想配列に追加。`srcLeft`と`srcTop`に適当な値を設定。
- 新しいパネルもマウスオーバーイベントが正しく機能するぜ。

### ドラッグ可能なパネルと組み合わせる
前の質問で使った`vue-draggable-resizable`と組み合わせたい場合、以下のようにイベントハンドラーを追加：

```vue
```vue
<template>
  <div class="panel-container">
    <vue-draggable-resizable
      v-for="(panel, key) in panels"
      :key="key"
      :w="panel.width"
      :h="panel.height"
      :x="panel.x"
      :y="panel.y"
      :draggable="true"
      :resizable="true"
      :parent="true"
      class-name="panel"
      @dragging="(x, y) => updatePosition(key, x, y)"
      @resizing="(x, y, w, h) => updateSize(key, x, y, w, h)"
      @mouseover="handleMouseOver(key, panel.srcLeft, panel.srcTop)"
    >
      <Tile :srcLeft="panel.srcLeft" :srcTop="panel.srcTop" />
    </vue-draggable-resizable>
    <p>最後にホバーしたパネル: {{ lastHovered }}</p>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import VueDraggableResizable from 'vue-draggable-resizable';
import 'vue-draggable-resizable/style.css';
import Tile from './Tile.vue';

interface PanelData {
  srcLeft: number;
  srcTop: number;
  width: number;
  height: number;
  x: number;
  y: number;
}

type PanelMap = Record<string, PanelData>;

export default defineComponent({
  name: 'DraggablePanel',
  components: {
    VueDraggableResizable,
    Tile
  },
  data() {
    return {
      panels: {
        panel1: { srcLeft: 10, srcTop: 20, width: 200, height: 150, x: 0, y: 0 },
        panel2: { srcLeft: 30, srcTop: 40, width: 200, height: 150, x: 220, y: 0 },
        panel3: { srcLeft: 50, srcTop: 60, width: 200, height: 150, x: 440, y: 0 }
      } as PanelMap,
      lastHovered: ''
    };
  },
  methods: {
    handleMouseOver(key: string, srcLeft: number, srcTop: number) {
      this.lastHovered = `キー: ${key}, srcLeft: ${srcLeft}, srcTop: ${srcTop}`;
      console.log(`Hovered on panel: ${key}, srcLeft: ${srcLeft}, srcTop: ${srcTop}`);
    },
    updatePosition(key: string, x: number, y: number) {
      this.panels[key].x = x;
      this.panels[key].y = y;
    },
    updateSize(key: string, x: number, y: number, width: number, height: number) {
      this.panels[key].x = x;
      this.panels[key].y = y;
      this.panels[key].width = width;
      this.panels[key].height = height;
    }
  }
});
</script>

<style scoped>
.panel-container {
  position: relative;
  min-height: 500px;
  width: 100%;
  border: 1px solid #ccc;
}

.panel {
  background: #f0f0f0;
  border: 1px solid #999;
  border-radius: 4px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
  cursor: pointer;
}
</style>
```
```

- `@mouseover`を`<vue-draggable-resizable>`に追加し、ホバー時に`srcLeft`と`srcTop`を処理。
- `<Tile>`はドラッグ＆リサイズ可能なパネル内に配置される。

### 注意点
- **イベントの伝播**：`<Tile>`内で`@mouseover`を`$emit`してるので、親コンポーネントで直接`@mouseover`を指定する場合は、子コンポーネントがイベントを正しく発火するように注意。
- **型安全性**：TypeScriptで`srcLeft`と`srcTop`を`number`型として定義。もし文字列が必要なら`string`に変更してな。
- **カーソルの視覚的フィードバック**：`.tile`や`.panel`に`cursor: pointer`を追加して、ホバー可能なことを分かりやすくしてる。
- **依存関係**：ドラッグ機能を追加するなら、`vue-draggable-resizable`をインストール（`npm i vue-draggable-resizable`）して、CSSをインポート。

これで、マウスオーバーで`srcLeft`と`srcTop`を受け取るイベントハンドラーが実装できたぜ！（＾～＾） もし他にカスタマイズ（例えば、ホバー時に特定のスタイルを適用、特定のアクションを実行）や質問があれば、ガンガン教えてくれよ！
