# 外部ドライブ

VSCode は NAS を認識しない。  
VSCode を管理者モードで起動すること。  

`vite` は UNCパスを認識しない。  
したがって、 `\\TAKAHASHI-NAS\Share\以下略` のようなパスを認識しない。  
`Z:\` のような仮想ドライブのパスにする必要がある。  
GUI で 仮想ドライブを割り当てていても、 VSCode はそれを認識していない。 VSCode のターミナルでも仮想ドライブを割り当てる。  

仮想ドライブを作る：  

```shell
subst Z: "\\TAKAHASHI-NAS\Share"
cd z:\

dir

cd z:\muzudho-github.com\muzudho\wara-city-tauri-desktopapp
```

作業終わったら以下で解除：  

```shell
subst z:\ /d
```

VSCode ターミナルで PowerShell 選択。  
（PowerShell は UNCパスをカレントディレクトリーとしてサポート）  

```shell
cd "\\TAKAHASHI-NAS\Share\muzudho-github.com\muzudho\wara-city-tauri-desktopapp"
```

## PowerShell の Execution Policy の設定

```shell
Get-ExecutionPolicy
    # Restricted
```

`Restricted` や `AllSigned` が出たら制限中。  

```shell
# 制限緩和
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
# > Y

pnpm install
```

```log
Progress: resolved 117, reused 0, downloaded 53, added 5
  44:     0x7ffb47fee8d7 - BaseThreadInitThunk
  45:     0x7ffb4934c53c - RtlUserThreadStart
thread caused non-unwinding panic. aborting.
```

👆 でコケた。  

`node_modules` フォルダー、  
`pnpm-lock.yaml` ファイル、  
`src-tauri/target` フォルダー  
を削除して以下を打鍵！  

```shell
rustup update
pnpm install --frozen-lockfile=false
```

```log
info: checking for self-update

  stable-x86_64-pc-windows-msvc updated - rustc 1.92.0 (ded5c06cf 2025-12-08) (from rustc 1.88.0 (6b00bc388 2025-06-23))

info: cleaning up downloads & tmp directories

thread 'main' panicked at src\utils\mod.rs:479:13:
Unable to clean up C:\Users\muzud\.rustup\tmp: Os { code: 5, kind: PermissionDenied, message: "アクセスが拒否されました。" }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

👆 コケた！  

```shell
# PowerShell（管理者）で
Remove-Item -Recurse -Force "$env:USERPROFILE\.rustup\tmp" -ErrorAction SilentlyContinue

rustup self update
```

```shell
# 再インストール手順
rustup toolchain uninstall stable
rustup toolchain install stable

rustup default stable
pnpm install
```

```log
  45:     0x7ffb4934c53c - RtlUserThreadStart
thread caused non-unwinding panic. aborting.
```

👆 コケた！  

```shell
# フルクリーン＆再インストール
Remove-Item -Recurse -Force node_modules, pnpm-lock.yaml, src-tauri/target
rustup update stable
pnpm install --shamefully-hoist --frozen-lockfile=false
```

