# Solana Devnet SOL Transfer（単一 HTML 版）

Solana **Devnet 限定**で、ブラウザウォレット間の SOL 送受信を行う Web アプリです。
`index.html` 1 ファイルだけで動作し、ビルドや `npm install` は不要です（`@solana/web3.js` は CDN から読み込みます）。

## 主な機能

| 機能 | 内容 |
| --- | --- |
| ウォレット接続 | Wallet Standard 対応ウォレット（Phantom / Solflare / Backpack など）を自動検出。旧来の `window.phantom.solana` / `window.solflare` にもフォールバック |
| 残高表示 | WebSocket（`accountSubscribe`）で残高をリアルタイム更新 |
| エアドロップ | Devnet で 1 SOL をエアドロップ（レート制限時は [faucet.solana.com](https://faucet.solana.com/) を案内） |
| SOL 送金 | 送金先アドレスと数量（SOL、小数点以下 9 桁まで）を指定して送金。残高不足や rent-exempt 不足は送信前に検出 |
| 送金フロー可視化 | 入力検証 → Tx 構築 → シミュレーション → ウォレット署名 → ネットワーク送信 → Processed → Confirmed → Finalized の 8 段階をステッパーで表示。各段階の経過時間（ms）、blockhash・手数料・slot・シグネチャ、シミュレーションログをリアルタイム表示 |
| 送受信履歴 | 「すべて / 送信履歴 / 受信履歴」をタブで切り替え。日時、種別、送信元、送信先、数量、手数料、取引状態（送信中 / Processed / Confirmed / Finalized / 失敗 / 期限切れ）、シグネチャ（Explorer リンク）を表示。受信を検知すると自動で更新 |
| Devnet 限定 | 起動時と RPC 変更時に RPC のジェネシスハッシュを検証し、Devnet 以外の RPC では動作しません |

履歴は `getSignaturesForAddress` と `getParsedTransactions` で取得した**実際のオンチェーンデータ**です。
System Program の `transfer` / `transferWithSeed` / `createAccount` 命令（内部命令を含む）を解析して送受信を判定します。
SOL 送金を含まないトランザクションは「その他」として残高の変化量を表示します。

## 必要なもの

- モダンブラウザ（Chrome / Edge / Brave / Firefox など）
- Solana ウォレット拡張機能（例: [Phantom](https://phantom.com/)、[Solflare](https://solflare.com/)）
- Devnet の SOL（アプリ内のエアドロップボタン、または [faucet.solana.com](https://faucet.solana.com/) で取得）

### ウォレットを Devnet に切り替える（推奨）

送金処理自体はアプリが Devnet の RPC に直接送信するため、ウォレットのネットワーク設定に関係なく Devnet で実行されます。
ただし、ウォレット側の残高表示や署名画面の内容を一致させるため、Devnet への切り替えをおすすめします。

- **Phantom**: 設定 → 開発者設定 → 「テストネットモード」を ON → Solana のネットワークを **Devnet** に設定
- **Solflare**: 設定 → ネットワーク → **Devnet**

## 起動方法（ローカル）

> **注意:** `index.html` をダブルクリックして `file://` で開くと、多くのウォレット拡張機能はページに注入されず接続できません。
> 以下のいずれかの方法で、ローカルの HTTP サーバーから開いてください。

### Python がある場合

```bash
cd solana-devnet-transfer-html
python -m http.server 8000
```

### Node.js がある場合

```bash
cd solana-devnet-transfer-html
npx serve .
```

ブラウザで <http://localhost:8000>（`npx serve` の場合は <http://localhost:3000>）を開きます。

### VS Code の場合

拡張機能「Live Server」をインストールし、`index.html` を右クリックして「Open with Live Server」を選びます。

## 使い方

1. 「ウォレット」欄で使用するウォレットを選んで接続します。
2. 残高が 0 の場合は「1 SOL エアドロップ」を押すか、Faucet から SOL を取得します。
3. 「送金先アドレス」と「送金数量 (SOL)」を入力し、「送金する」を押します。
4. ウォレットのポップアップで署名を承認します。
5. 「送金処理フロー」で各段階の進み具合を確認します。Finalized になれば送金完了です（通常は送信から十数秒）。
6. 「取引履歴」に送信した取引が表示され、状態が Confirmed → Finalized と更新されます。受信側のウォレットでアプリを開くと、受信履歴として表示されます。

### 送金ルールの補足

- **未作成のアドレス**（一度も SOL を受け取ったことがないアドレス）には、rent-exempt 最小額（約 0.00089088 SOL）以上を送る必要があります。
- 送金後の自分の残高は「0」か「rent-exempt 最小額以上」のどちらかである必要があります。「最大」ボタンを押すと、手数料（5000 lamports）を差し引いた全額が入力されます。

### RPC エンドポイントの変更（任意）

公開 RPC（`https://api.devnet.solana.com`）はレート制限が厳しく、履歴の読み込みやエアドロップで
`429 Too Many Requests` になることがあります。その場合は「ウォレット」欄の「RPC 設定（Devnet のみ）」を開き、
Helius / QuickNode などの **Devnet 用** RPC URL を入力して「適用」を押してください。
Devnet 以外（Mainnet / Testnet）の URL は自動的に拒否されます。

## GitHub Pages での公開手順

静的ファイルだけで動くので、GitHub Pages にそのまま公開できます。GitHub Pages は HTTPS で配信されるため、ウォレット拡張機能も正常に動作します。

### 方法 A: ブラウザだけで公開する

1. GitHub で新しいリポジトリを作成します（例: `solana-devnet-transfer`、**Public**）。
2. リポジトリのページで **Add file → Upload files** を選び、`index.html` と `README.md` をアップロードして **Commit changes** を押します。
3. **Settings → Pages** を開きます。
4. **Build and deployment** の **Source** で **Deploy from a branch** を選びます。
5. **Branch** で `main`、フォルダで `/ (root)` を選んで **Save** を押します。
6. 1〜2 分待つと、ページ上部に公開 URL が表示されます。
   `https://<ユーザー名>.github.io/<リポジトリ名>/`

### 方法 B: git コマンドで公開する

```bash
cd solana-devnet-transfer-html
git init
git add index.html README.md
git commit -m "Add Solana devnet SOL transfer app"
git branch -M main
git remote add origin https://github.com/<ユーザー名>/<リポジトリ名>.git
git push -u origin main
```

push したあと、方法 A の手順 3〜6 で GitHub Pages を有効にします。

[GitHub CLI](https://cli.github.com/) を使う場合は、リポジトリ作成から push までを 1 コマンドで行えます。

```bash
gh repo create <リポジトリ名> --public --source=. --push
```

### 更新するとき

`index.html` を変更して commit / push すると、数分以内に自動で再デプロイされます。
デプロイの状況はリポジトリの **Actions** タブ（`pages build and deployment`）で確認できます。

## 技術メモ

- ライブラリ: [`@solana/web3.js` v1.99.0](https://www.npmjs.com/package/@solana/web3.js)（IIFE ビルドを jsDelivr から読み込み、失敗した場合は unpkg にフォールバック）
- ウォレット連携: [Wallet Standard](https://github.com/wallet-standard/wallet-standard) の `standard:connect` / `solana:signTransaction` を直接利用（`chain: "solana:devnet"`）
- 送信方式: ウォレットでは**署名のみ**を行い、アプリが Devnet RPC に `sendRawTransaction` で送信します。これにより、ウォレットのネットワーク設定に関係なく、必ず Devnet に送信されます。
- 確定の追跡: `getSignatureStatuses` を 500ms 間隔でポーリングして Processed / Confirmed / Finalized の到達を検出します。`lastValidBlockHeight` を超えた場合は「期限切れ」と判定します。
- 秘密鍵はアプリで一切扱いません。署名はすべてウォレット内で行われます。

## 注意事項

- このアプリは **Devnet 専用**です。Devnet の SOL に金銭的価値はありません。
- Devnet は定期的にリセットされることがあり、その場合は過去の履歴や残高が消えます。
- 公開 RPC のレート制限により、履歴の取得が一時的に失敗することがあります。その場合は少し待ってから「更新」を押してください。
