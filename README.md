# たびもの手帳 — iOS / TestFlight 配信プロジェクト

「たびもの手帳」（単一HTMLファイルのPWA）を、[Capacitor](https://capacitorjs.com/) でネイティブのiOSアプリとして包んだプロジェクトです。**Codemagic**（クラウド上のMacでビルドするCI/CDサービス）を使ってビルドし、TestFlightへアップロードします。**お使いのMacは不要です**。すでに「ゆめ貯金」で使っているのと同じ仕組みなので、操作感は近いはずです。

すでに終わっていること：
- Capacitorプロジェクトの作成、iOSネイティブプロジェクト（`ios/`）の生成
- アプリアイコン・起動画面（スプラッシュ）を、アプリ本体の画像から作成して設定済み（`ios/App/App/Assets.xcassets/` 内に直接コミット済みなので、ビルド時にSVGから生成する手順は不要です）
- アプリ表示名「たびもの手帳」を設定済み
- Codemagicでビルド・TestFlightアップロードするための `codemagic.yaml` を用意済み

あなたにお願いしたいこと：

---

## 1. Bundle ID を決める

今は仮に `com.tabimonotecho.app` を設定しています。このままでも構いませんが、他の人がすでに使っている可能性もあるので、心配であれば自分だけのものに変更してください（例：`com.あなたのドメインや名前.tabimono`）。

変更する場合は、このプロジェクト内の2箇所を書き換えます。

- `capacitor.config.json` の `"appId"`
- `codemagic.yaml` の `bundle_identifier` と `vars.BUNDLE_ID`

## 2. App Store Connect で Bundle ID とアプリを登録する

1. [developer.apple.com/account](https://developer.apple.com/account) → **Certificates, Identifiers & Profiles** → **Identifiers** → ＋ボタンで新規App IDを作成し、上で決めたBundle ID（例：`com.tabimonotecho.app`）を登録します。
2. [appstoreconnect.apple.com](https://appstoreconnect.apple.com) → **マイApp** → ＋ボタン → 「新規App」。プラットフォームは iOS、先ほどのBundle IDを選択し、App名（例：たびもの手帳）を入力して作成します。
   - この時点でTestFlightの配信先アプリが用意されます。審査提出はまだ不要です（TestFlightの内部テストだけなら審査不要です）。

## 3. GitHubリポジトリを作成してこのフォルダをアップロードする

このフォルダをまだGitHubに置いていない場合は、新しいリポジトリ（Privateでも構いません）を作成し、この中身一式をアップロードしてください。「ゆめ貯金」とは別の、新しいリポジトリにするのがおすすめです（アプリごとに分けた方がビルド設定がシンプルになります）。

```
cd tabimono-ios
git init
git add .
git commit -m "Initial iOS wrapper for TestFlight (Codemagic)"
git branch -M main
git remote add origin https://github.com/あなたのアカウント/tabimono-ios.git
git push -u origin main
```

## 4. Codemagicにリポジトリを接続する

1. [codemagic.io](https://codemagic.io) にログイン（「ゆめ貯金」で使っているのと同じアカウントでOKです）。
2. **Add application** → 手順3で作ったGitHubリポジトリを選択して接続します。
3. ビルド設定の検出方法を聞かれたら、「リポジトリ内の `codemagic.yaml` を使う」を選びます（このプロジェクトに最初から入っています）。

## 5. App Store Connect の連携（署名・アップロード用）を設定する

Codemagicの **Teams settings** → **Integrations** → **Developer Portal** で、Apple Developerアカウントと連携するApp Store Connect APIキーを設定します。

- すでに「ゆめ貯金」用に登録済みのAPIキー（同じApple Developerアカウント・チームのものであれば）をそのまま**再利用できます**。その場合、連携の名前（Reference name）を確認し、`codemagic.yaml` の `integrations.app_store_connect` の値をその名前に書き換えてください（今は仮に `tabimono_asc_key` としています）。
- 新しく作る場合は、「ゆめ貯金」の時と同じ手順です。
  1. App Store Connect → **ユーザとアクセス** → **統合** → **App Store Connect API** でキーを生成（権限は「App Manager」以上）。
  2. Codemagicの連携設定で、そのKey ID・Issuer ID・ダウンロードした `.p8` ファイルを登録し、わかりやすい名前（Reference name）を付ける。例：`tabimono_asc_key`
  3. `codemagic.yaml` の `integrations.app_store_connect` の値を、その名前に合わせる（現在のファイルは `tabimono_asc_key` という名前を前提にしています。連携に別の名前を付けた場合は書き換えてください）。

この連携があれば、証明書・プロビジョニングプロファイルの発行から署名、TestFlightへのアップロードまで、Codemagicが自動でやってくれます（手動で `.p12`/`.mobileprovision` を用意する必要はありません）。

## 6. ビルドを実行する

Codemagicのアプリ画面 → **Start new build** → ブランチ（`main`）とワークフロー（`tabimono-ios-testflight`）を選んで実行します。

- 「ゆめ貯金」と同じく、**GitHubにpushしても自動でビルドは始まりません**。新しいバージョンを配信したいときは、その都度 Codemagic 画面で **Start new build** を押してください。
- 初回は証明書・プロビジョニングプロファイルの生成も入るので15〜25分ほどかかります。進捗はCodemagicのビルドログで確認できます。
- 失敗した場合は、ログ末尾のエラーメッセージを教えてください。原因を一緒に確認します。

アップロードが終わると、App Store Connect側で「処理中」表示になり、数分〜数十分でTestFlightで使えるようになります（この「Apple側の処理」はこちらでは短縮できません）。

## 7. TestFlightでテスターを招待する

App Store Connect → 対象のApp → **TestFlight** タブで、内部テスター（自分のApple IDなど、最大100人・審査不要）を追加すれば、TestFlightアプリ経由ですぐにインストールして試せます。外部テスター（最大1万人）を使う場合は簡単な審査（Beta App Review）が入ります。

---

## バージョンを上げて再配信したいとき

`codemagic.yaml` が、TestFlightの最新ビルド番号を自動取得して+1するようにしているので、通常は何もしなくても再度ビルドを実行するだけで新しいビルドとしてアップロードされます。アプリのバージョン表記（例：1.0 → 1.1）を変えたい場合は、`ios/App/App.xcodeproj/project.pbxproj` 内の `MARKETING_VERSION` を変更してください。

## アプリの中身を更新したいとき

中身（`www/index.html`）を新しいバージョンに差し替えてから、GitHubにpushし、Codemagicで再度ビルドを実行するだけです。Xcodeプロジェクト側（`ios/`フォルダ）は触る必要はありません。

## もしMacが使えるようになったら

`ios/` フォルダを Xcode で直接開く（`npx cap open ios`、または `ios/App/App.xcworkspace` をダブルクリック）ことで、シミュレータでの動作確認や手動ビルドもできます。Codemagicでの自動化と併用して問題ありません。

## トラブルシューティング

- **ビルドが署名関連のエラーで失敗する**：手順2でApp Store ConnectにApp自体を登録できているか、手順5のAPIキー権限が「App Manager」以上か、連携の名前（Reference name）と `codemagic.yaml` の `integrations.app_store_connect` の値が一致しているか確認してください。
- **「Bundle identifier is already in use」と出る**：手順1のBundle IDを、誰も使っていない文字列に変更してください（`capacitor.config.json` と `codemagic.yaml` の両方）。
- **アイコンや画面を差し替えたい**：`www/index.html` を新しいものに差し替えればOKです。アプリアイコン自体（ホーム画面の四角いアイコン）を変えたい場合は `ios/App/App/Assets.xcassets/AppIcon.appiconset/AppIcon-512@2x.png`（1024×1024・透過なし）を差し替えてください。
- **「ゆめ貯金」と同じApple Developerアカウントだが、連携を分けるべきか迷う**：同じチーム（同じApple Developerアカウント）であれば、App Store Connect APIキーの連携は1つを複数アプリ・複数Codemagicプロジェクトで共有して問題ありません。
