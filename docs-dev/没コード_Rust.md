# 没コード Rust

```rust
use serde::{Deserialize, Serialize};
//use std::fs::{self};
//use std::path::Path;
//use csv::Reader;
//use std::fs::File;
//use std::collections::HashMap;
//use tauri::command;


// マップ・データ
#[derive(Debug, Deserialize)]
#[serde(rename_all = "camelCase")]
struct Board {
    width_cells: i32,        // number -> i32
    height_cells: i32,       // number -> i32
    _area_cells: i32,         // number -> i32
    _width_pixels: i32,       // number -> i32
    _height_pixels: i32,      // number -> i32
    tilepath_array: Vec<String>, // string[] -> Vec<String>
}

#[derive(Serialize, Deserialize)]
struct CsvRow {
    // CSVのヘッダーに合わせてフィールドを定義（例: name, age）
    name: String,
    age: String,
}


// Learn more about Tauri commands at https://tauri.app/develop/calling-rust/
#[tauri::command]
#[allow(non_snake_case)]
fn greetRs(name: &str) -> String {
    format!("Hello, {}! You've been greeted from Rust!", name)
}


/*
#[tauri::command]
#[allow(non_snake_case)]
fn paintRs(drawingName:&str, tileIndex:i32, selectedTilepath:&str, board:Board) -> HashMap<i32, String> {
    let mut dict = HashMap::new();

    // １マスずつ
    if drawingName == "dot" {
        dict.insert(tileIndex, String::from(selectedTilepath));
    
    // 塗り潰し
    } else if drawingName == "fill" {

        let mut checkboard : Vec<bool> = vec![false; board.tilepath_array.len()];
        let mut tile_index_buffer : Vec<i32> = vec![];
        let mut next_tile_index_buffer : Vec<i32> = vec![];

        tile_index_buffer.push(tileIndex);
        let target_tilepath = board.tilepath_array[tileIndex as usize].clone();

        // NOTE: 再帰関数はスタック・オーバーフローするタイミングが読めないので、ただのループで実装する。
        // FIXME: 🌟 Rust は高速でも、TypeScript の方が高速ではない？ JSON受け渡しが遅い？
        loop {
            if tile_index_buffer.is_empty() || 100 < dict.len() {   // 100 ぐらいなら速い
                break;
            }

            next_tile_index_buffer.clear();

            for tile_index_2 in tile_index_buffer.iter() {

                // 塗る
                dict.insert(*tile_index_2, String::from(selectedTilepath));
                checkboard[*tile_index_2 as usize] = true;

                // 次の四方向を追加
                add_4_sides(
                        *tile_index_2,
                        &mut next_tile_index_buffer,
                        &board,
                        &mut checkboard,
                        target_tilepath.as_str());
            }

            // 入れ替え
            tile_index_buffer.clear();
            tile_index_buffer.extend(next_tile_index_buffer.iter().cloned()); // コピーして追加
        }

    // 境界線の自動接続
    } else if drawingName == "border" {
        dict.insert(tileIndex, String::from(selectedTilepath));
        
    }

    dict
}
*/

/*
fn add_4_sides(
        tile_index: i32,
        next_tile_index_buffer: &mut Vec<i32>,
        board:&Board,
        checkboard : &mut Vec<bool>,
        target_tilepath: &str
) {

    // 上のタイルが同じなら塗り潰し
    let up_index = tile_index - board.width_cells;
    if 0 <= up_index && !checkboard[up_index as usize] {
        if target_tilepath == board.tilepath_array[up_index as usize] {
            next_tile_index_buffer.push(up_index);
        }
    }

    // 右のタイルが同じなら塗り潰し
    if tile_index%board.width_cells != (board.width_cells-1) {
        let right_index = tile_index + 1;
        if (right_index as usize) < board.tilepath_array.len() && !checkboard[right_index as usize] {
            if target_tilepath == board.tilepath_array[right_index as usize] {
                next_tile_index_buffer.push(right_index);
            }
        }
    }

    // 左のタイルが同じなら塗り潰し
    if tile_index%board.width_cells!=0 {
        let left_index = tile_index - 1;
        if !checkboard[left_index as usize] {
            if target_tilepath == board.tilepath_array[left_index as usize] {
                next_tile_index_buffer.push(left_index);
            }
        }
    }

    // 下のタイルが同じなら塗り潰し
    if tile_index%board.height_cells != (board.height_cells-1) {
        let down_index = tile_index + board.width_cells;
        if (down_index as usize) < board.tilepath_array.len() && !checkboard[down_index as usize] {
            if target_tilepath == board.tilepath_array[down_index as usize] {
                next_tile_index_buffer.push(down_index);
            }
        }
    }
}
 */


// ディレクトリー内のファイル名を取得するTauriコマンド。
// 読取成功時はファイル名のリストを、失敗時はエラーメッセージを返す。
// NOTE: 引数にアンダースコア区切りは使えないようだ。
//      (Grok)
//      Rust 側で #[tauri::command] を付けた関数の引数名は、JavaScript 側でそのままキー名として使われる。
//      JavaScript ではスネークケースは一般的じゃないから、Tauri はキャメルケースを前提にシリアライズ/デシリアライズを行う。
//      スネークケースだと、Vue から送ったデータが Rust 側で正しくマッチしないんだ。
//      ついでに、#[tauri::command] の関数名自体もスネークケース推奨だぜ。キャメルケースの関数名でも動くけど、Tauri のドキュメントや慣習ではスネークケースが一般的だから、統一すると読みやすいよ。
#[tauri::command]
#[allow(non_snake_case)]
fn translateRs(sourceStr:&str, commandName: &str) -> String {
    if commandName == "都道府県スプリット1" {
        let mut s = String::from("");
        let lines: Vec<&str> = sourceStr.lines().collect();
        for &line in lines.iter() {
            let pref = get_pref(line);  // 都道府県名にマッチするものを探す
            if pref != "" {
                let pref_len = pref.len();      // 都道府県名の長さを取得                    
                let rest = &line[pref_len..];   // 都道府県以降の部分を抽出
                s.push_str(&format!("{0},{1}\n",pref,rest));
            }
            else{
                s.push_str(&format!(",{}\n",&line));    // 空列を入れる。
            }
        }

        return s
    }
    sourceStr.to_string()
}

pub fn get_pref(line: &str) -> &'static str {
    // 47都道府県のリスト
    let prefectures = [
        "北海道", "青森県", "岩手県", "宮城県", "秋田県", "山形県", "福島県",
        "茨城県", "栃木県", "群馬県", "埼玉県", "千葉県", "東京都", "神奈川県",
        "新潟県", "富山県", "石川県", "福井県", "山梨県", "長野県", "岐阜県",
        "静岡県", "愛知県", "三重県", "滋賀県", "京都府", "大阪府", "兵庫県",
        "奈良県", "和歌山県", "鳥取県", "島根県", "岡山県", "広島県", "山口県",
        "徳島県", "香川県", "愛媛県", "高知県", "福岡県", "佐賀県", "長崎県",
        "熊本県", "大分県", "宮崎県", "鹿児島県", "沖縄県",
    ];

    for &pref in prefectures.iter() {
        if line.starts_with(pref) {    // 都道府県名にマッチするものを探す
            return pref
        }
    }        

    return ""
}

#[tauri::command]
#[allow(non_snake_case)]
fn readCsvRs(file_path: String) -> Result<Vec<CsvRow>, String> {
    let file = File::open(&file_path).map_err(|e| e.to_string())?;
    let mut rdr = Reader::from_reader(file);
    let mut rows = Vec::new();

    for result in rdr.deserialize() {
        let record: CsvRow = result.map_err(|e| e.to_string())?;
        rows.push(record);
    }

    Ok(rows)
}

```
