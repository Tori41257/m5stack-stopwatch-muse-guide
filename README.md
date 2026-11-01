# Windows・VS CodeでM5Stack StopWatchにMuseを導入する

Windows PCにESP-IDFを導入し、Muse Gadget SDKをM5Stack StopWatch向けにビルドして、USBで書き込む手順です。日本語の返答字幕を表示する設定まで扱います。

2026年11月1日に、Windows x64環境でビルド・書き込み・Museとの接続・日本語字幕表示を確認しました。PlatformIOではなく、VS CodeとEspressif公式のESP-IDF拡張を使います。既存のPlatformIO環境は残して構いません。

初回は1〜10章を順番に進めてください。11章は再書き込み用、12章はトラブル時の参照用です。アバター変更はこの手順に含めません。

## 目次

- [1. 必要なもの](#1-必要なもの)
- [2. VS Code・Git・ESP-IDF拡張を入れる](#2-vs-codegitesp-idf拡張を入れる)
- [3. ESP-IDF v6.0.1を入れる](#3-esp-idf-v601を入れる)
- [4. Muse Gadget SDKを取得する](#4-muse-gadget-sdkを取得する)
- [5. VS CodeでESP-IDFを使えるようにする](#5-vs-codeでesp-idfを使えるようにする)
- [6. MuseのSDKトークンを取得する](#6-museのsdkトークンを取得する)
- [7. StopWatch専用設定と日本語フォントを有効にする](#7-stopwatch専用設定と日本語フォントを有効にする)
- [8. ビルドする](#8-ビルドする)
- [9. StopWatchへ書き込む](#9-stopwatchへ書き込む)
- [10. スマホとペアリングする](#10-スマホとペアリングする)
- [11. 設定変更後の再ビルド・再書き込み](#11-設定変更後の再ビルド再書き込み)
- [12. うまくいかないとき](#12-うまくいかないとき)
- [13. GitHubに公開するファイル](#13-githubに公開するファイル)
- [14. 用語と参考リンク](#14-用語と参考リンク)

## 1. 必要なもの

| 項目 | 今回の構成 |
|---|---|
| PC | Windows x64 |
| 本体 | M5Stack StopWatch、製品コードC152 |
| USBケーブル | データ通信対応USBケーブル |
| エディター | Visual Studio Code |
| 開発環境 | ESP-IDF **v6.0.1** |
| ファームウェア | MetaのMuse Gadget SDK |
| スマホ | Museアプリと利用可能なMuseアカウント |
| ネットワーク | 2.4GHzのWi-Fiとインターネット接続 |

StopWatchはESP32-S3、16MB Flash、8MB PSRAMを搭載しています。本記事の対象はMetaのAIエージェントMuseです。

- [StopWatch製品ページ（スイッチサイエンス）](https://www.switch-science.com/products/11200)
- [StopWatch公式ドキュメント](https://docs.m5stack.com/en/core/StopWatch)
- [Muse公式サイト](https://muse.ai/)
- [Muse Gadget SDK](https://github.com/facebookincubator/muse-gadget-sdk)

Muse SDKの確認時点の指定バージョンはESP-IDF v6.0.1です。新しいESP-IDFを無条件に選ばず、SDKの[ESP32向けREADME](https://github.com/facebookincubator/muse-gadget-sdk/blob/main/esp32/README.md)と[ビルド手順](https://github.com/facebookincubator/muse-gadget-sdk/blob/main/esp32/AGENTS.md)を優先してください。SDKのmainブランチは更新されるため、将来も同じ手順で動くとは限りません。

USB書き込みでは、本体に入っているファームウェアが置き換わります。工場出荷時に戻す必要がある場合は、先に[StopWatch公式ドキュメント](https://docs.m5stack.com/en/core/StopWatch)のファームウェア復元手順を確認してください。

## 2. VS Code・Git・ESP-IDF拡張を入れる

### 2-1. VS CodeとGit

未導入の場合は、次の公式サイトからWindows用をインストールします。

- [Visual Studio Code](https://code.visualstudio.com/download)
- [Git for Windows](https://git-scm.com/downloads/win)

Gitの導入後はVS Codeを起動し直してください。VS Codeの［ターミナル］→［新しいターミナル］でPowerShellを開き、確認します。

```powershell
git --version
```

バージョンが表示されれば次へ進めます。

### 2-2. ESP-IDF拡張

1. VS Codeで`Ctrl + Shift + X`を押します。
2. `Espressif IDF`を検索します。
3. 提供元がEspressif Systemsの拡張をインストールします。

[ESP-IDF拡張の配布ページ](https://marketplace.visualstudio.com/items?itemName=espressif.esp-idf-extension)

拡張機能はVS CodeからESP-IDFを操作するためのものです。拡張を入れただけでは、ESP-IDF本体やコンパイラーの導入は終わっていません。

## 3. ESP-IDF v6.0.1を入れる

### 3-1. インストールマネージャーを起動する

VS Codeで`Ctrl + Shift + P`を押し、次を実行します。

```text
ESP-IDF: Open ESP-IDF Installation Manager
```

起動しない場合は、公式配布ページから直接ダウンロードします。今回もこの方法を使いました。

- [ESP-IDF Installation Manager配布ページ](https://dl.espressif.com/dl/eim/)
- [GitHubのリリース一覧](https://github.com/espressif/idf-im-ui/releases)
- [今回確認したEIM v0.20.1のWindows x64 GUI版](https://github.com/espressif/idf-im-ui/releases/download/v0.20.1/eim-gui-windows-x64.exe)

Windows x64では`eim-gui-windows-x64.exe`を選び、ダウンロードしたファイルを直接起動します。`cli`はコマンド操作用、`gui`は画面操作用です。EIMのv0.20.1と、これから入れるESP-IDFのv6.0.1は別のバージョンです。

### 3-2. インストール画面で選ぶもの

今回のExpert Installationでは、次のように指定しました。画面名や項目はEIMのバージョンによって変わる場合があります。

| 画面 | 選択内容 |
|---|---|
| Prerequisites Check | 必須項目の確認を完了する |
| Python Sanity Check | Pythonの動作確認を完了する |
| Select Target Chips | `All`を外し、**esp32s3**だけ選ぶ |
| Select IDF Version | **v6.0.1**を選ぶ |
| Select Download Mirrors | 下記の既定ミラーを使う |
| Select ESP-IDF Features | 必須の`core`のみ |
| Select Tools | 必須ツールをそのまま選択。`qemu-xtensa`は不要 |
| Select Installation Location | 今回は`C:\esp` |

v6.0.1が表示されないときは、**Show old stable versions**にチェックを入れて過去の安定版を表示します。`release-v6.0`や`master`は選びません。

日本からのダウンロードでは、今回は次のミラーで進めました。

| 用途 | URL |
|---|---|
| ESP-IDF Repository | `https://github.com` |
| ESP-IDF Tools | `https://dl.espressif.com/github_assets` |
| PyPI | `https://pypi.org/simple` |

インストール先の画面では、`Use custom tool download / install folder locations`は未選択で進めます。`Delete temporary installation files after the install completes`は、完了後に一時ファイルを削除したい場合に選びます。

`Installation Complete!`が表示されたら終了します。デスクトップにはIDF PowerShellのショートカットが作成されます。

### 3-3. インストールの確認

デスクトップのIDF PowerShellを開き、起動処理が終わったら実行します。

```powershell
idf.py --version
```

次の表示を確認します。

```text
ESP-IDF v6.0.1
```

これが表示されたら本体の導入は完了です。以後、同じ場所に再インストールする必要はありません。

今回の環境では`IDF_PATH`は`C:\esp\v6.0.1\esp-idf`、ツール群は`C:\Espressif\tools`に配置されました。実際のパスはIDF PowerShellの起動時表示で確認できます。

## 4. Muse Gadget SDKを取得する

ESP-IDF本体のインストール先と、開発するコードの保存先は別です。コードはDocuments内の`ESP-IDF`フォルダーに置きます。

PowerShellで次を実行してください。

```powershell
New-Item -ItemType Directory -Force "$env:USERPROFILE\Documents\ESP-IDF" | Out-Null
Set-Location "$env:USERPROFILE\Documents\ESP-IDF"
git clone https://github.com/facebookincubator/muse-gadget-sdk.git
```

すでに取得済みなら、`git clone`を繰り返す必要はありません。

構成は次のようになります。

```text
Documents\ESP-IDF\
└─ muse-gadget-sdk\
   └─ esp32\          ← VS Codeで開くフォルダー
      ├─ components\
      ├─ devices\
      ├─ main\
      ├─ CMakeLists.txt
      └─ sdkconfig.defaults
```

取得した版を後から確認できるよう、次の結果を自分の作業メモに残しておくと便利です。

```powershell
git -C "$env:USERPROFILE\Documents\ESP-IDF\muse-gadget-sdk" rev-parse HEAD
```

## 5. VS CodeでESP-IDFを使えるようにする

### 5-1. プロジェクトを開く

VS Codeの［ファイル］→［フォルダーを開く］で、次を開きます。`ユーザー名`は自分のWindowsユーザー名に置き換えます。

```text
C:\Users\ユーザー名\Documents\ESP-IDF\muse-gadget-sdk\esp32
```

［ワークスペースを開く］でファイルを探す必要はありません。既存のSDKを開くので、新しいESP-IDFプロジェクトの作成も不要です。

### 5-2. ESP-IDFの環境を選ぶ

`Ctrl + Shift + P`から次を実行し、v6.0.1を選びます。

```text
ESP-IDF: Select Current ESP-IDF Version
```

続いて次でターミナルを開きます。

```text
ESP-IDF: Open ESP-IDF Terminal
```

ターミナルで`idf.py --version`を実行し、v6.0.1と表示されることを確認します。

### 5-3. 環境一覧が出ない場合

今回うまくいった方法は、IDF PowerShellからVS Codeを起動する方法です。

1. 開いているVS Codeのウィンドウをすべて閉じます。
2. デスクトップのIDF PowerShellを開きます。
3. 次を実行します。

```powershell
code "$env:USERPROFILE\Documents\ESP-IDF\muse-gadget-sdk\esp32"
```

開いたVS CodeでESP-IDF Terminalを起動し、もう一度`idf.py --version`を確認します。VS Codeを閉じずに実行すると既存プロセスが使われ、環境変数を引き継げない場合があります。

公式拡張はEIMの`eim_idf.json`から環境を検出します。ファイルの場所が既定と異なる場合は、拡張設定の`idf.eimIdfJsonPath`で実在するファイルを指定する方法もあります。詳しくは[公式の環境設定手順](https://docs.espressif.com/projects/vscode-esp-idf-extension/en/latest/installation.html)を参照してください。

以後のコマンドは、**ESP-IDFを有効にしたPowerShellで、SDK内のesp32フォルダーを作業場所にして**実行します。

## 6. MuseのSDKトークンを取得する

1. [SDK tokens](https://gadgets.muse.ai/settings/sdk-tokens)をブラウザーで開きます。
2. スマホのMuseアプリと同じアカウントでログインします。
3. `Your tokens`で新しいトークンを発行します。
4. `Copy your new token`の［Copy］でコピーし、手元で保管します。
5. コピー後に［Done］を押します。

トークン全文は発行時に一度だけ表示されます。失くした場合は［Revoke］で失効させて再発行します。トークンが写ったスクリーンショットを公開した場合も、失効させて作り直してください。

公開ページにはトークン単体の購入案内は見当たりませんでした。ただし、Museサービス本体の契約・料金・利用可能地域などはアカウント側で確認してください。

トークンは個人・非商用向けの条件で提供されています。SDKのソースコードのライセンスと、Museへのアクセスに使うトークンの利用条件は別です。[Gadget SDK Token Terms](https://gadgets.muse.ai/sdk-terms)を確認してください。

## 7. StopWatch専用設定と日本語フォントを有効にする

### 7-1. 専用のビルド設定を作る

ESP32-S3を選ぶだけでは、画面・音声・電源などのStopWatch用設定は入りません。SDKの共通設定、Muse UI設定、StopWatch設定の順に読み込みます。

VS CodeのESP-IDFターミナルで、次の1行を実行します。

```powershell
idf.py -B build-muse-stopwatch -DIDF_TARGET=esp32s3 -DSDKCONFIG=build-muse-stopwatch/sdkconfig "-DSDKCONFIG_DEFAULTS=sdkconfig.defaults;devices/sdkconfig.muse;devices/sdkconfig.muse-m5stack-stopwatch" menuconfig
```

PowerShellでは、セミコロンを含む`-DSDKCONFIG_DEFAULTS=...`全体を引用符で囲んでください。

| 指定 | 役割 |
|---|---|
| `-B build-muse-stopwatch` | StopWatch専用のビルド先 |
| `-DIDF_TARGET=esp32s3` | CPUの種類 |
| `-DSDKCONFIG=...` | 今回の設定保存先 |
| `sdkconfig.defaults` | SDK共通設定 |
| `devices/sdkconfig.muse` | Museの画面UIなどの設定 |
| `devices/sdkconfig.muse-m5stack-stopwatch` | StopWatch固有の設定 |

初回は設定の生成や依存ライブラリの取得に時間がかかります。この手順を使う場合、先に別途`set-target`を実行する必要はありません。

### 7-2. トークンを入力する

`menuconfig`はキーボードで操作する設定画面です。ターミナル内をクリックし、半角英数入力にしてください。

| キー | 操作 |
|---|---|
| ↑・↓ | 項目を選ぶ |
| Enter | メニューを開く・入力を確定する |
| Space | チェックを切り替える |
| `/` | 設定項目を検索する |
| `S` | 保存する |
| `Q` | 終了する |

次の項目を開き、取得したトークンを入力します。

```text
ESP32 Device SDK
└─ Muse Gadgets SDK token
```

入力後に確定します。以前`idf.py menuconfig`だけで設定したトークンは別のビルド設定に保存されている場合があるため、このStopWatch専用設定にも入力してください。

### 7-3. 日本語字幕を有効にする

次の階層に移動します。

```text
Component config
└─ Muse
   └─ Show Chinese, Japanese and Korean in captions
```

または`/`で`MUSE_CJK_FONT`を検索し、該当項目へ移動します。

Spaceを押して、次のように`[*]`へ変更します。

```text
[*] Show Chinese, Japanese and Korean in captions
[ ] Draw CJK captions with Noto Sans SC instead of Unifont
```

今回はNotoの選択は外し、標準のUnifontを使いました。Unifontはかな・漢字を含む広い文字範囲を収録し、約1.2MBのFlashを使います。これは**返答字幕の日本語表示**を有効にする設定です。設定メニューの英語表記はそのままです。

### 7-4. 保存して終了する

1. `S`を押します。
2. 保存先を確認し、そのままEnterを押します。
3. `Configuration saved`を確認します。
4. `Q`で終了します。

保存先は`build-muse-stopwatch/sdkconfig`です。

設定を確認する場合は、トークンを表示しないよう必要な項目だけ検索します。

```powershell
Select-String -Path .\build-muse-stopwatch\sdkconfig -Pattern '^CONFIG_MUSE_BOARD_M5STACK_STOPWATCH=y$','^CONFIG_MUSE_CJK_FONT=y$'
```

両方が表示されれば、StopWatchと日本語フォントの設定が有効です。

## 8. ビルドする

同じターミナルで実行します。

```powershell
idf.py -B build-muse-stopwatch build
```

初回は多数のファイルをコンパイルします。途中の`NOTE`や警告だけで失敗と判断せず、最後の結果を確認します。

```text
Project build complete. To flash, run:
```

この表示が出たら成功です。書き込むファイルは`build-muse-stopwatch/muse-gadget.bin`に生成されます。

## 9. StopWatchへ書き込む

### 9-1. 接続ポートを特定する

StopWatchをデータ通信対応USBケーブルでPCにつなぎ、実行します。

```powershell
python -m serial.tools.list_ports
```

例：

```text
COM1
COM3
COM8
COM10
4 ports found
```

この一覧だけでは機種を判断できません。複数ある場合は次の方法で確認します。

1. StopWatchのUSBを抜き、同じコマンドを実行します。
2. 一覧から消えたCOM番号を確認します。
3. USBを挿し直し、同じ番号が戻るか確認します。

接続を変えると番号も変わる場合があります。以下では**COM10を例**にしています。必ず自分の番号へ置き換えてください。

### 9-2. 書き込みとログ表示

```powershell
idf.py -B build-muse-stopwatch -p COM10 flash monitor
```

`flash`で書き込み、`monitor`で起動ログを表示します。完了後、本体が再起動してMuseの画面が表示されることを確認します。

ここではVS Code下部の書き込みボタンを使わず、**`-B build-muse-stopwatch`を指定したコマンドを使います**。拡張側が別の`build`フォルダーを使っていると、古い設定のファームウェアを書き込む場合があります。

ログ表示を終了するには`Ctrl + ]`を押します。本体はそのまま動き続けます。

## 10. スマホとペアリングする

1. スマホでMuseアプリを開きます。
2. ［Settings］→［Devices］でDeveloper modeをオンにします。
3. ［Add Device］から`MuseGadget...`を選びます。
4. 本体の確認画面で、確認用のボタンを押します。
5. アプリの案内に従い、2.4GHzのWi-Fiを設定します。
6. 接続後、ボタンを押しながら話し、離して送信します。

今回の通常設定では、USB端子を下にして見たとき、青い右上ボタンが話す／ペアリング確認、黄色い左上ボタンがスピーカー切り替えです。実験的な`Live voice calls with Muse`を有効にした構成では役割が変わるため、本体のアイコンと[SDKのStopWatch説明](https://github.com/facebookincubator/muse-gadget-sdk/blob/main/esp32/devices/README.md)を確認してください。

日本語表示の確認では、次のように話しかけます。

```text
「こんにちは」と日本語で一言だけ返事して。
```

本体の返答字幕が日本語になれば完了です。日本語フォントを入れても、返答言語が自動的に日本語へ固定されるわけではありません。

## 11. 設定変更後の再ビルド・再書き込み

次回以降もESP-IDF Terminalを使い、作業場所をSDK内の`esp32`にします。まずシリアルモニターを`Ctrl + ]`で終了してください。

設定を変えるとき：

```powershell
idf.py -B build-muse-stopwatch menuconfig
```

ビルドするとき：

```powershell
idf.py -B build-muse-stopwatch build
```

書き込むとき：

```powershell
idf.py -B build-muse-stopwatch -p COM10 flash monitor
```

初回のターゲットやdefaultsの指定は、既存のビルドフォルダーに保存されています。ビルドフォルダーを削除して作り直す場合は、7章の初回コマンドから設定してください。通常の再書き込みで`erase-flash`は不要です。

## 12. うまくいかないとき

### ESP-IDFのコマンドが検索に出ない

`Ctrl + Shift + P`で`ESP-IDF`だけ入力します。1つも出なければ拡張のインストールと有効状態を確認してください。`Developer: Reload Window`で再読み込みする方法もあります。

### 「拡張機能をアクティブにしています…」のまま進まない

`Developer: Reload Window`を試します。`No standard ESP-IDF project was found...`が出た場合は、SDKの`esp32`フォルダーを開いていることを確認したうえで、必要に応じて［Activate Anyway］を押します。

### 「EIM executable not found」やインストールボタンが無反応

VS Code側の処理をキャンセルし、3章の公式サイトからEIMのGUI版を取得して直接起動します。詳しいメッセージは［出力］タブで出力先を`ESP-IDF`に切り替えて確認できます。

### 「No ESP-IDF Setups found」

環境が検出されていない状態です。まずデスクトップのIDF PowerShellで`idf.py --version`を確認します。v6.0.1と出るなら再インストールせず、5-3の方法でVS Codeを起動してください。

### 再インストール時に「Path C:\esp is not valid...」

既存ファイルと衝突しているという表示です。すでに`Installation Complete!`を確認した場合はインストーラーを閉じ、IDF PowerShellでバージョンを確認します。動作確認済みのフォルダーを削除する必要はありません。

### 「compile_commands.json is missing」

コード解析用ファイルがまだ生成されていません。環境導入前にこのボタンを押しても、ESP-IDF本体はインストールされません。環境とStopWatch設定を整え、まずビルドを完了させます。

### 「No SDK token」

トークン未設定の警告です。`idf.py -B build-muse-stopwatch menuconfig`で、今回書き込む構成にトークンを保存してください。別フォルダーの設定を更新しても反映されません。

### 書き込み時にポートを開けない

COM番号を確認し、別のシリアルモニターやターミナルがそのポートを使用していないか確認します。ほかのモニターを閉じてから再実行してください。ポート自体が現れない場合は、データ通信対応ケーブルか、本体が起動しているかを確認します。

接続待ちで止まる場合のダウンロードモードへの入り方は、[StopWatch公式ドキュメント](https://docs.m5stack.com/en/core/StopWatch)を参照してください。

### 日本語が「□□□」になる

日本語の返答は届いていても、実行中のファームウェアに必要なフォントがない可能性があります。今回は、日本語フォント入りのビルドを明示して再書き込みすることで直りました。

1. `CONFIG_MUSE_CJK_FONT=y`を確認します。
2. `idf.py -B build-muse-stopwatch build`を実行します。
3. **同じ`-B build-muse-stopwatch`を付けて**書き込みます。

日本語が英語で返ってくる場合はフォントとは別の問題です。「日本語で返事して」と依頼してください。

### フォルダーの構成を確認したい

| 場所 | 内容 |
|---|---|
| `C:\esp` | 今回のESP-IDF本体のインストール先 |
| `C:\Espressif\tools` | 今回の開発ツールの配置先 |
| `Documents\ESP-IDF\muse-gadget-sdk\esp32` | 編集・コマンド実行するプロジェクト |
| `esp32\build-muse-stopwatch` | StopWatch用の生成物と設定 |
| `esp32\build` | 別の構成の生成物が残ることがある場所 |

## 13. GitHubに公開するファイル

この手順書だけを公開するなら、この`README.md`をアップロードすれば読めます。SDK本体は公式リポジトリへのリンクで案内できます。

自分のSDKの作業フォルダー全体を、そのままアップロードしないでください。トークンは設定ファイルだけでなく、生成されたヘッダー・バイナリーなどにも含まれます。特に次のものは公開対象から外します。

- `build/`、`build-muse-stopwatch/`などのビルドフォルダー
- 実際のトークンを保存した`sdkconfig`とバックアップ
- トークン入りのファームウェア、フラッシュバックアップ
- トークンやWi-Fi情報が写ったスクリーンショット・ログ

この手順書は実トークンを含みません。過去にGitへコミットした秘密情報は、後から`.gitignore`へ追加するだけでは履歴から消えません。漏れたトークンは失効させてください。

GitHubのWeb画面で公開する場合は、自分のリポジトリで［Add file］→［Upload files］を開き、このREADMEを選択します。既存のREADMEがある場合は、`docs/stopwatch-muse-setup-ja.md`など別名で追加できます。

## 14. 用語と参考リンク

| 用語 | 意味 |
|---|---|
| ESP-IDF | EspressifのESP32向け開発環境 |
| EIM | ESP-IDFと必要なツールを導入するマネージャー |
| ターゲット | ビルド対象のチップ。今回はESP32-S3 |
| menuconfig | キーボードで操作する設定画面 |
| sdkconfig | 保存されたビルド設定。今回はトークンも含む |
| ビルド | ソースコードから本体用ファームウェアを作る処理 |
| flash | USBなどを通じて本体へ書き込む処理 |
| monitor | 本体のシリアルログを見る処理 |
| COM番号 | WindowsでUSBシリアル機器を指定する番号 |
| CJKフォント | 中国語・日本語・韓国語の文字を扱うフォント |

公式資料：

- [VS Codeのダウンロード](https://code.visualstudio.com/download)
- [Git for Windows](https://git-scm.com/downloads/win)
- [ESP-IDF拡張](https://marketplace.visualstudio.com/items?itemName=espressif.esp-idf-extension)
- [ESP-IDF拡張のインストール・設定](https://docs.espressif.com/projects/vscode-esp-idf-extension/en/latest/installation.html)
- [ESP-IDF Installation Manager](https://dl.espressif.com/dl/eim/)
- [Muse Gadget SDKのESP32向けREADME](https://github.com/facebookincubator/muse-gadget-sdk/blob/main/esp32/README.md)
- [Muse Gadget SDKのビルド手順](https://github.com/facebookincubator/muse-gadget-sdk/blob/main/esp32/AGENTS.md)
- [対応ボードとStopWatchの操作](https://github.com/facebookincubator/muse-gadget-sdk/blob/main/esp32/devices/README.md)
- [日本語フォントの説明](https://github.com/facebookincubator/muse-gadget-sdk/blob/main/esp32/components/muse/fonts/README.md)
- [SDKトークンの発行](https://gadgets.muse.ai/settings/sdk-tokens)
- [SDKトークンの利用条件](https://gadgets.muse.ai/sdk-terms)
- [M5Stack StopWatch](https://docs.m5stack.com/en/core/StopWatch)

問題が解消しない場合は、使用したSDKのコミット、ESP-IDFのバージョン、実行コマンド、エラー末尾をそろえて[SDKのIssues](https://github.com/facebookincubator/muse-gadget-sdk/issues)で既知の問題を確認してください。投稿するログからはトークンや個人情報を除いてください。

