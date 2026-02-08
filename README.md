# Wall Street Junior Clone - Financial Simulation Engine

## 概要 (Overview)

本プロジェクトは、Docker、FastAPI、MySQLを用いたブラウザベースの金融取引シミュレーションゲームです。
株式、債券、外国為替、融資などの多様な金融商品を扱い、マクロ経済ニュースや金利変動が市場に影響を与えるリアルな経済モデルをシミュレートします。

## 技術スタック (Tech Stack)

* **Frontend:** HTML5 / JavaScript (Vanilla)
* **Backend:** Python 3.12+ (FastAPI)
* **Database:** MySQL 8.0
* **Infrastructure:** Docker & Docker Compose

## シミュレーション詳細仕様

### 1. 資産クラス (Asset Classes)

#### A. 株式 (Equities)

* **パラメータ:** 現在値、配当利回り、ボラティリティ、企業業績ランク。
* **ロジック:** 基本はランダムウォークだが、四半期ごとの「決算発表」や「セクターニュース」によりトレンドが発生する。

#### B. 債券 (Bonds) - Fixed Income

* **特徴:** 満期（Maturity）と表面利率（Coupon）を持つ。
* **価格形成:** 市場金利（Interest Rate）と逆相関する。
* *金利上昇 → 債券価格下落*
* *金利低下 → 債券価格上昇*


* **格付け:** AAA〜C。格付けが低いほどデフォルトリスクが高いが、利回りが高い。

#### C. 外国為替 (Forex)

* **特徴:** 2国間の通貨ペア（例: USD/JPY）。
* **変動要因:** 各国の「金利差」と「インフレ率」に強く影響される。

#### D. 融資・銀行業務 (Loans)

* **プレイヤーの役割:** 銀行として顧客に融資を行う。
* **リスク管理:** 顧客の信用スコア（Credit Score）に基づき、貸し倒れリスクと金利を天秤にかける。

### 2. 経済指標 (Macro Economics)

シミュレーション全体に影響を与えるグローバル変数。

* **基準金利 (Base Interest Rate):** 中央銀行が決定。債券とForexに即座に影響。
* **GDP成長率:** 株式市場全体の底上げ/押し下げ。
* **市場心理 (Sentiment):** Bullish（強気） / Bearish（弱気）。

## データベース設計 (Schema Overview)

### `market_data`

現在の市場価格を管理する。
| Column | Type | Description |
| :--- | :--- | :--- |
| `ticker` | VARCHAR | シンボル (e.g. "AAPL", "USDJPY") |
| `price` | DECIMAL | 現在価格 |
| `type` | ENUM | STOCK, BOND, FOREX |
| `volatility` | FLOAT | 変動の激しさ |

### `users`

プレイヤー情報。
| Column | Type | Description |
| :--- | :--- | :--- |
| `id` | INT | PK |
| `cash` | DECIMAL | 現金残高 (Buying Power) |

### `portfolios`

プレイヤーの保有資産。
| Column | Type | Description |
| :--- | :--- | :--- |
| `user_id` | INT | FK |
| `ticker` | VARCHAR | 保有銘柄 |
| `quantity` | INT | 保有数 |
| `average_price` | DECIMAL | 平均取得単価 |

### `orders`

注文履歴とアクティブな注文。
| Column | Type | Description |
| :--- | :--- | :--- |
| `id` | INT | PK |
| `status` | ENUM | PENDING, EXECUTED, CANCELED |
| `limit_price` | DECIMAL | 指値 (成行の場合はNULL) |

## 開発環境セットアップ (Setup)

1. **リポジトリのクローン**
```bash
git clone [repository_url]
cd financial-sim

```


2. **コンテナの起動**
```bash
docker-compose up -d --build

```


3. **APIドキュメントへのアクセス**
ブラウザで `http://localhost:8000/docs` にアクセスし、Swagger UIが表示されることを確認する。
4. **マイグレーション (初期化)**
```bash
# テーブル作成等の初期化スクリプトを実行（例）
docker-compose exec backend python init_db.py

```


### 補足: 実装上のヒント

* **小数点の扱い:** 金融計算において浮動小数点数（float）は誤差が出るため厳禁です。内部では整数型を使用し、フロントエンドで桁を調整するなどして正確な値を表示します。
