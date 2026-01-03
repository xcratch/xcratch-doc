# Xcratch 拡張機能の作り方

Xcratch の拡張機能を、エディタ本体の開発フローに沿って作成・デバッグする手順です。

## 開発環境のセットアップ

### ディレクトリ構成とクローン

Xcratch は monorepo 構成を採用しています。拡張機能のリポジトリと同じ階層に [xcratch/scratch-editor](https://github.com/xcratch/scratch-editor) を配置します。

```
.
├── scratch-editor/          # monorepo ルート
│   └── packages/
│       ├── scratch-gui/     # フロントエンド UI
│       ├── scratch-vm/      # 仮想マシン
│       ├── scratch-render/  # レンダリングエンジン
│       └── ...
└── xcx-my-extension/        # あなたの拡張機能
```

scratch-editor リポジトリをクローンし、依存パッケージをインストールします。

```sh
git clone https://github.com/xcratch/scratch-editor.git
cd scratch-editor
npm install
```

### HTTPS 証明書の準備

開発サーバーは HTTPS で起動するため、ローカルマシン用の証明書が必要です。[mkcert](https://github.com/FiloSottile/mkcert) を使って準備します。

```sh
# mkcert のインストール（macOS の場合）
brew install mkcert
mkcert -install

# 証明書の生成
cd scratch-editor
mkdir -p .vscode
mkcert -key-file .vscode/localhost-key.pem -cert-file .vscode/localhost.pem localhost 127.0.0.1 ::1
```

証明書のパスは [packages/scratch-gui/webpack.config.js](https://github.com/xcratch/scratch-editor/blob/xcratch/packages/scratch-gui/webpack.config.js) で設定されています。

### 拡張機能のスキャフォールディング

scratch-editor と同じ階層に移動し、[xcratch-create](https://www.npmjs.com/package/xcratch-create) で拡張機能の雛形を生成します。

```sh
cd ..  # scratch-editor の親ディレクトリへ
npx xcratch-create --repo=xcx-my-extension --account=githubAccount --extensionID=myExtension --extensionName='My Extension'
```

主な引数:
- --repo: GitHub リポジトリ名
- --account: GitHub アカウント
- --extensionID: Xcratch 内での拡張機能 ID
- --extensionName: 拡張機能名

### リポジトリ初期化

```sh
cd xcx-my-extension
git init -b main
git remote add origin <REPO_URL>
git add .
git commit -m "Scaffold code"
git push -u origin main
```

依存パッケージを入れます。xcratch-create が生成したテンプレートでは、scratch-vm への参照は monorepo 内のパッケージを使用するように設定されています。

```sh
npm install
```

### scratch-vm への参照設定

拡張機能の開発中に BlockType, Cast などの scratch-vm のソースコードを参照できるように、シンボリックリンクを作成します。

```sh
npm run setup-dev
```

## ビルドとウォッチ

rollup.js ベースのスクリプトでモジュールを生成します。

```sh
npm run build   # dist/extensionID.mjs を作成
npm run watch   # 変更を監視して自動ビルド
```

## モジュールを読み込んでデバッグ

### VSCode デバッグ機能を使う（推奨）

scratch-editor の VSCode デバッグ設定を使うと、拡張機能のソースコードに直接ブレークポイントを設定してデバッグできます。

#### 1. ワークスペースの設定

VSCode で scratch-editor と拡張機能の両方をワークスペースに追加します。

```
File > Add Folder to Workspace... > (scratch-editor と拡張機能のフォルダを追加)
```

#### 2. launch.json の確認

scratch-editor の [.vscode/launch.json](https://github.com/xcratch/scratch-editor/blob/xcratch/.vscode/launch.json) には、拡張機能のソースマップを解決するための設定が含まれています。

```json
{
  "webRoot": "${workspaceFolder}/..",
  "sourceMapPathOverrides": {
    "webpack://GUI/*": "${webRoot}/scratch-editor/packages/scratch-gui/*",
    "webpack://GUI/scratch-vm/*": "${webRoot}/scratch-editor/packages/scratch-vm/*",
    "https://0.0.0.0:5500/*": "${webRoot}/*"
  }
}
```

`https://0.0.0.0:5500/*` の部分は、拡張機能をローカルサーバーで配信する場合のソースマップ解決用です。

#### 3. 拡張機能のビルドとウォッチ

拡張機能のディレクトリで watch モードを起動します。

```sh
cd xcx-my-extension
npm run watch
```

#### 4. ローカル HTTPS サーバーの起動

Live Server で、拡張機能ディレクトリを HTTPS 配信します。

Live Server の設定例

scratch-editor の launch.json にあわせて、`https://localhost:5500/` で配信するように設定します。

.vscode/settings.json :

```json
{
  "liveServer.settings.port": 5500,
  "liveServer.settings.host": "0.0.0.0",
  "liveServer.settings.https": {
    "enable": true,
    "cert": "/path/to/localhost.pem",
    "key": "/path/to/localhost-key.pem"
  },
  "liveServer.settings.CustomBrowser": "chrome",
}
```

ワークスペースの親ディレクトリをルートにして拡張機能のモジュールをアクセスできるようにします。

xcx-my-extension.code-workspace :

```json
{
	"folders": [
		{
			"path": "xcx-my-extension"
		},
		{
			"path": "scratch-editor"
		}
	],
	"settings": {
		"liveServer.settings.root": "../",
		"liveServer.settings.multiRootWorkspaceName": "xcx-my-extension"
	}
}
```

#### 5. デバッグの開始

1. VSCode で scratch-editor を開く
2. F5 キーを押して「debug on dev-server」を起動
3. Chrome が自動的に開き、`https://localhost:8601` にアクセス
4. Xcratch Editor で「拡張機能を読み込む」から拡張機能を読み込み
   ```
   https://localhost:5500/dist/myExtension.mjs
   ```
5. 拡張機能のソースファイルにブレークポイントを設定
6. ブロックを実行してデバッグ

### ローカル Web サーバー経由で読み込み（シンプルな方法）

VSCode デバッグを使わない場合は、Live Server などで拡張機能を HTTPS 配信し、公開されている Xcratch Editor (https://xcratch.github.io/editor/) から読み込むこともできます。

Xcratch Editor の「拡張機能を読み込む」から、次の URL を指定します。

```
https://localhost:5500/dist/extensionID.mjs
```

拡張機能のサーバーは `https://xcratch.github.io/` からの CORS を許可している必要があります。

## 配布

### GitHub Pages での配布

Pages の配信元を `main` ブランチの `/(root)` に設定すると、モジュールは次のように公開されます。

```
https://<account>.github.io/<repository>/dist/<extensionID>.mjs
```

別サーバーで配信する場合も、`https://xcratch.github.io/` からの CORS を許可してください。

## サンプルプロジェクトと埋め込み

拡張機能のブロックを使ったプロジェクトを `projects/example.sb3` として用意すると、次の URL で自動的に拡張機能を読み込んで開けます。

```
https://xcratch.github.io/editor/#https://<account>.github.io/<repository>/projects/example.sb3
```

プレイヤーだけを埋め込む場合:

```html
<iframe src="https://xcratch.github.io/editor/player#https://<account>.github.io/<repository>/projects/example.sb3" width="600px" height="500px"></iframe>
```

カメラやマイクを使う拡張機能の場合、iframe 埋め込みで allow オプションを指定する必要があります。

```html
<iframe src="https://xcratch.github.io/editor/player#https://<account>.github.io/<repository>/projects/example.sb3" width="600px" height="500px" allow="camera; microphone;"></iframe>
```