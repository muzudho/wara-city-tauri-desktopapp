<template>
    <v-app>
        <v-main>
            <!-- クライアント領域 -->
            <v-container class="pa-0" :style="clientAreaStyle">
                <!-- 単純にタイルを敷き詰めているだけ。 -->
                <!-- 例えば、以下のようなタグをリピート。
                    <Tile
                            :srcLeft="0"
                            :srcTop="0"
                            :srcWidth="32"
                            :srcHeight="32"
                            :tilemapUrl="'/public/img/tiles/tilemap_sea.png'"/>
                    selectedTilemapKey: {{ selectedTilemapKey }}<br/>
                    <container
                            style="line-height: 12px;"
                            v-for="tilePath in board.tilepathArray">
                            tilePath: {{ tilePath }}<br/>
                    </container>
                -->
                <Tile
                        v-for="(tilePath, index) in board.tilepathArray"
                        :key="index"
                        :srcLeft="srcTileCollection.getTileByPath(tilePath).left"
                        :srcTop="srcTileCollection.getTileByPath(tilePath).top"
                        :srcWidth="srcTileCollection.getTileByPath(tilePath).width"
                        :srcHeight="srcTileCollection.getTileByPath(tilePath).height"
                        :tilemapUrl="getTilemapUrlByName(newTilepath(tilePath).getTilemapName())"
                        @click="onMapTileClick(index)"
                        @mousedown="onMapTileMouseDown(index)"
                        @mouseup="onMapTileMouseUp(index)"
                        @mousemove="onMapTileMouseMove(index)"
                        style="z-index: 1;"/>

                <!-- ブラシ・ウィンドウ
                -->
                <BrushPanel
                        v-bind:selectedTilepath="selectedTilePathVM"
                        v-bind:selectedTileData="selectedTileDataVM"
                        v-bind:drawingMethodName="selectedDrawingMethodNameVM"
                        v-on:selectDrawingMethodName="onSelectedDrawingMethodNameSelected"
                        style="position:fixed; top:16px; left:16px; z-index: 30;"/>

                <!-- タイル・パレット・ウィンドウ

                    TODO 🌟 タイルや、タイルマップのコレクションは、イベントによって、後から渡すようにできないか？ ref とか defineExpose が使えるのでは？
                -->
                <TilePalettePanel
                        :srcTileCollection="srcTileCollection"
                        :srcTilemapCollection="srcTilemaps"
                        :startConfig="startConfig"
                        v-on:selectTilemap="onTilemapSelected"
                        v-on:selectTile="onSrcTileSelected"
                        style="position:fixed; top:170px; left:16px; z-index: 10;"/>

                <!-- ターミナル・ウィンドウ
                -->
                <TerminalPanel
                        :srcTileCollection="srcTileCollection"
                        :srcTilemaps="srcTilemaps"
                        :board="board"
                        style="position:fixed; top:16px; left:240px; z-index: 20;"/>

            </v-container>
        </v-main>
    </v-app>    
</template>

<script setup lang="ts">

    // ##############
    // # インポート #
    // ##############

    import { invoke } from "@tauri-apps/api/core";
    import { computed, onMounted, Reactive, ref } from "vue";

    // ++++++++++++++++++++++++++++++++++
    // + インポート　＞　コンポーネント +
    // ++++++++++++++++++++++++++++++++++
    //
    // @はsrcへのエイリアス
    //

    import Tile from '@/components/Tile.vue';
    import TilePalettePanel from '@/components/TilePalettePanel.vue';
    import TerminalPanel from '@/components/TerminalPanel.vue';
    import BrushPanel from '@/components/BrushPanel.vue';

    // ++++++++++++++++++++++++++++++++++
    // + インポート　＞　コンポーザブル +
    // ++++++++++++++++++++++++++++++++++

    import { Board, createEmptyBoard } from '@/composables/board';
    import { paint } from '@/composables/paint';
    import { createEmptySourceTilesCollection, loadSourceTilesCollection, SourceTileCollection } from '@/composables/source-tile-collection';
    import { createEmptySourceTilemapCollection, loadSourceTilemapCollection, SourceTilemapCollection } from '@/composables/source-tilemap-collection';
    import { getTilemapUrlByName } from '@/composables/tilemap-filepath-collection';
    import { newTilepath } from '@/composables/tilepath';

    // ++++++++++++++++++++++++++++++++++++
    // + インポート　＞　インターフェース +
    // ++++++++++++++++++++++++++++++++++++

    import { TileData } from '@/interfaces/tile-data';

    // ##############
    // # 共有データ #
    // ##############

    const startConfig = ref<any | null>();   // 読み込む前と、読み込んだ後の２状態がある。

    //
    // NOTE: Reactive にするオブジェクトは、null ではなく、空オブジェクトで初期化します。
    //
    // 盤情報は、ゲーム内のターミナル・ウィンドウと共有できる変数にしたい。
    const srcTileCollection : Reactive<SourceTileCollection> = createEmptySourceTilesCollection();
    const srcTilemaps: Reactive<SourceTilemapCollection> = createEmptySourceTilemapCollection(srcTileCollection);
    const board : Reactive<Board> = createEmptyBoard(srcTileCollection);   // 盤。いわゆるマップ。

    // ##############
    // # 起動時処理 #
    // ##############

    onMounted(async () => {
        const confFileName = 'start-config.json';
        try {
            // Rust言語（バックグラウンド相当）の関数を呼び出し
            // NOTE: `invoke` は Webブラウザーでは動作しません。Tauri ウィンドウで起動してください。
            startConfig.value = await invoke('read_bundle_text_file', {'fileName': confFileName});
            //alert(`DEBUG: ファイル読み取り練習中： ${JSON.stringify(startConfig.value, null, "    ")}`);
        } catch (error) {
            alert(`${confFileName}ファイルを読込めませんでした。これが invoke が読めないことによるエラーの場合、Webブラウザーではなく Tauri ウィンドウで起動してください。エラーが出ましたが、継続します。\n起動時エラー： ${error}`);
        }

        try {           
            // TODO データを後から読み込みたい。
            await loadSourceTilesCollection(ref(srcTileCollection.tileDict));     // FIXME: こんな書き方でいいのか？
            loadSourceTilemapCollection(startConfig, srcTilemaps, srcTileCollection);
            //board.reloadBoard();

        } catch (error) {
            alert(`ゲームを正常に起動できませんでした。\n起動時エラー： ${error}`);
        }
    });    

    // ############################
    // # このコンポーネントの画面 #
    // ############################

    // ++++++++++++++++++++
    // + クライアント領域 +
    // ++++++++++++++++++++

    // クライアント領域。マップのタイルが敷き詰めてある。
    const clientAreaStyle = computed(
        function(): string {
            return '' + //
                ' max-width: ' + board.widthPixels + 'px;' + // NOTE: max-width が 1200px ぐらいしかないような気がする。増やしておく。
                ' width:' + board.widthPixels + 'px;' +   // 横幅。
                ' line-height: 0;'
        }
    );

    // ++++++++++++++++++++++++++++++++
    // + クライアント領域　＞　マップ +
    // ++++++++++++++++++++++++++++++++

    const mouseDraggingVM = ref(false);

    /**
     * マップタイルをクリックしたとき。
     * @param index クリックしたタイルのインデックス。
     */
    function onMapTileClick(index: number) {
        //alert(`マップタイルをクリックした： index=${index}`)
        // if (selectedTilePathVM.value == '') {
        //     return;
        // }

        // マップタイルを更新
        callPaint(
            selectedDrawingMethodNameVM.value,
            index);
    }

    function onMapTileMouseDown(_index: number) {
        mouseDraggingVM.value = true;
    }

    function onMapTileMouseUp(_index: number) {
        mouseDraggingVM.value = false;
    }

    function onMapTileMouseMove(index: number) {
        if (! mouseDraggingVM.value) {
            return;
        }

        // マップタイルを更新
        callPaint(
            selectedDrawingMethodNameVM.value,
            index);
    }

    // ++++++++++++++++++++++++++++++++++++++++
    // + クライアント領域　＞　ブラシ・パネル +
    // ++++++++++++++++++++++++++++++++++++++++

    const selectedDrawingMethodNameVM = ref<string>("dot")     // FIXME: 初期値どうする？

    function onSelectedDrawingMethodNameSelected(drawingMethodName: string) {
        selectedDrawingMethodNameVM.value = drawingMethodName;
    }

    // ++++++++++++++++++++++++++++++++++++++++++
    // + クライアント領域　＞　タイル・パレット +
    // ++++++++++++++++++++++++++++++++++++++++++

    const selectedTilePathVM = ref('sea_0');  // FIXME: 初期値どうする？
    const selectedTileDataVM = ref<TileData>({  // FIXME: 初期値どうする？
        left: 0,
        top: 0,
        width: 32,
        height: 32,
    });

    function onTilemapSelected(_tilemapName: string, tilePath: string, tile: TileData) {
        //alert(`タイルマップを変更した： タイルマップ名：${_tilemapName} タイルパス=${tilePath} タイル：${tile.srcLeft} ${tile.srcTop}`)
        selectedTilePathVM.value = tilePath;
        selectedTileDataVM.value = tile;
    }

    /**
     * ソースタイルをクリックしたとき。
     * @param tilePath 
     */
    function onSrcTileSelected(tilePath: string, tile: TileData) {
        //alert(`ソースタイル２をクリックした： tilePath=${tilePath}`)
        selectedTilePathVM.value = tilePath;
        selectedTileDataVM.value = tile;
    }

    // ################
    // # サブルーチン #
    // ################

    /**
     * Tauriのコマンドを呼び出し。
     * 文字列を渡すと、指定の操作を実施後の文字列を返す。
     * @param tileIndex クリックしたタイルのインデックス
     */
    async function callPaint(drawingName: string, tileIndex: number): Promise<string> {

        if (board == null) {
            return "board is null.";
        }

        /**
         * 更新のレシピを返す。
         * @returns エラーメッセージ
         */
        const indexAndTilepathDict : Record<number, string> = paint({
            drawingName: drawingName,
            tileIndex: tileIndex,

            // TODO: 選択タイルが境界線かどうか、種類も渡したい。
            selectedTilepath: selectedTilePathVM.value,

            // TODO: マップ上のタイルが、境界線にどう影響するか、どのように Rust 側で判定する？
            // タイルパスと、タイル属性の紐づきを丸ごと Rust 側に投げるか？
            board: board,
        });

        // // 更新のレシピを返す。
        // const indexAndTilepathDict : Record<number, string> = await invoke<Record<number, string>>('paintRs',
        //     {
        //         drawingName: drawingName,
        //         tileIndex: tileIndex,

        //         // TODO: 選択タイルが境界線かどうか、種類も渡したい。
        //         selectedTilepath: selectedTilePathVM.value,

        //         // TODO: マップ上のタイルが、境界線にどう影響するか、どのように Rust 側で判定する？
        //         // タイルパスと、タイル属性の紐づきを丸ごと Rust 側に投げるか？
        //         board: toPlainBoard(board),
        //     });
        
        // // デバッグ表示
        // let text = "";
        // text += `描き方: ${drawingName}, インデックス1: ${tileIndex}\n`;
        // Object.entries(indexAndTilepathDict).forEach(([tileIndex2, tilepath]) => {
        //     text += `インデックス2: ${tileIndex2}, タイルパス: ${tilepath}\n`;
        // });
        // alert(text);

        // レシピ通り更新。
        Object.entries(indexAndTilepathDict).forEach(([tileIndex2, tilepath]) => {
            const tileIndex3 = Number(tileIndex2);
            board.tilepathArray[tileIndex3] = tilepath;
        });

        return "";
    }

</script>

<style>
html {
    /* Tauri のウィンドウに横スクロールバーを付ける */
    overflow-x: scroll !important;
}

:root {
    font-family: Inter, Avenir, Helvetica, Arial, sans-serif;
    font-size: 16px;
    /*line-height: 24px;*/
    font-weight: 400;

    color: #0f0f0f;
    background-color: #f6f6f6;

    font-synthesis: none;
    text-rendering: optimizeLegibility;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    -webkit-text-size-adjust: 100%;
}

.container {
    /*height: 95vh;*/
    margin: 0;
    display: flex;
    flex-direction: column;
    justify-content: center;
    text-align: center;
}

.row {
    display: flex;
    justify-content: center;
}

input,
button {
    border-radius: 8px;
    border: 1px solid transparent;
    font-size: 1em;
    font-weight: 500;
    font-family: inherit;
    color: #0f0f0f;
    background-color: #ffffff;
    transition: border-color 0.25s;
    box-shadow: 0 2px 2px rgba(0, 0, 0, 0.2);
}

button {
    padding: 0.6em 1.2em;
    cursor: pointer;
}

button:hover {
    border-color: #396cd8;
}
button:active {
    border-color: #396cd8;
    background-color: #e8e8e8;
}

input,
button {
    outline: none;
}

@media (prefers-color-scheme: dark) {
    :root {
        color: #f6f6f6;
        background-color: #2f2f2f;
    }

    input,
    button {
        color: #ffffff;
        background-color: #0f0f0f98;
    }
    button:active {
        background-color: #0f0f0f69;
    }
}
</style>