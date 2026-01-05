# 外部ドライブ

VSCode は NAS を認識しない。  
VSCode を管理者モードで起動すること。  

仮想ドライブを作る：  

```shell
subst Z: "\\TAKAHASHI-NAS\Share\[muzudho-github.com]\muzudho\wara-city-tauri-desktopapp" > cd /d Z:
```

作業終わったら以下で解除：  

```shell
subst Z: /d
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
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser > Y

pnpm install
```
