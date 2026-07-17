---
id: m0-db-usage
sidebar_position: 18
---

# Glows.ai リモート PostgreSQL データベース Matrix0 利用ガイド

## 機能紹介

### リモート PostgreSQL サービスとは？

Glows.ai は、マネージド PostgreSQL データベースサービスを提供しています。主な特徴は以下のとおりです。

| 特徴 | 説明 |
| ---- | ---- |
| **リレーショナルストレージ** | 標準 PostgreSQL 機能により、ユーザーデータや設定などを保存できます |
| **ベクトルストレージ** | pgvector 拡張機能を内蔵し、ベクトルデータの保存と検索に対応しています |
| **運用不要** | データベースのインストール、設定、メンテナンスは不要です |
| **高性能** | 専用インスタンスにより、低レイテンシーかつ高スループットを実現します |
| **自動バックアップ** | データは自動的にバックアップされ、安全に保護されます |

### 利用シーン

- AI Agent プロジェクト（RAG、ベクトル検索）
- 大規模言語モデルアプリケーション（会話メモリ、ナレッジベース）
- データ分析プロジェクト
- データベースを必要とするあらゆるプロジェクト

## サービスの有効化

サービスのクローズドベータ期間中は、Public IP を購入済みのお客様のみ利用できます。テスト利用をご希望の場合は、[こちらからお問い合わせ](#お問い合わせ)ください。

## インスタンスの作成

Glows.ai で必要に応じてインスタンスを作成します。作成方法は[チュートリアル](https://docs.glows.ai/docs/create-new)を参照してください。本ガイドでは **CUDA12.8 Torch2.8.0 Base**（img-6ypgvgpw）イメージを使用します。

`Create New` 画面で Workload Type に Inference GPU -- 4090 を選択し、先に **CUDA12.8 Torch2.8.0 Base** イメージを選択します。このイメージには、AI プロジェクトに必要な基礎環境（CUDA、Pytorch など）が公式に事前設定されています。

![ ](../../../../../docs/docs-images/p18/001.png)

必要に応じて `Unit Qty`（GPU 数）と `Mount Datadrive`（Glowsai クラウドストレージ）を設定できます。データベース機能を利用する場合は、さらに `Bind Public IP Address` の下にある `Bind` ボタンをクリックし、固定 IP を設定してください。

![ ](../../../../../docs/docs-images/p18/002.png)

インスタンスの起動が完了したら、インスタンス ID（ins-xxxx）を [glows小幫手](https://sass-ai.chatshare.biz/c/69e1af93-1f10-8330-a72c-a5b1705349ba#聯繫我們) に送信してください。エンジニアが設定を行った後、データベースの内部ネットワーク接続情報が提供されます。

以下の情報を受け取ります。

| パラメータ | 説明 | 例 |
| ---------- | ---- | -- |
| `HOST` | データベースアドレス | `172.172.1.1` |
| `PORT` | データベースポート | `3306` |
| `USER` | ユーザー名 | `glowsai` |
| `PASSWORD` | パスワード | `********` |
| `DATABASE` | デフォルトデータベース | `postgres` |

## 基本的な使い方

### 接続ツールのインストール

データベースへ直接接続する場合は、postgresql-client ツールを使用できます。SSH でインスタンスに接続した後、以下のコマンドでツールパッケージをインストールします。

```bash
# PostgreSQL クライアントをインストール
apt-get update && apt-get install -y postgresql-client

# インストールを確認
psql --version
```

![ ](../../../../../docs/docs-images/p18/003.png)

### データベースへ接続

psql コマンドラインを使用して接続します。以下の手順に従って操作してください。

```bash
# 基本接続コマンド
psql -h <HOST> -p <PORT> -U <USER> -d <DATABASE>

# 例（小幫手から提供された実際の情報に置き換えてください）
psql -h 172.172.1.1 -p 3306 -U glowsai -d postgres
```

![ ](../../../../../docs/docs-images/p18/004.png)

### データベースを作成

```sql
-- 新しいデータベースを作成
CREATE DATABASE my_project_db;

-- 新しいデータベースへ接続
\c my_project_db
```

![ ](../../../../../docs/docs-images/p18/005.png)

### ベクトルデータの基本操作

基本的なデータベース操作と同様に、以下ではデータベースへの追加、削除、検索、更新操作を示します。

#### ベクトルデータテーブルを作成

```sql
-- ベクトル拡張をインストール
CREATE EXTENSION IF NOT EXISTS vector;

-- ベクトルカラムを持つテーブルを作成
CREATE TABLE embeddings (
    id bigserial PRIMARY KEY,
    content text,
    embedding vector(4)
);
```

#### ベクトルデータを挿入

```sql
-- テストデータを挿入（ベクトルを模擬）
INSERT INTO embeddings (content, embedding) VALUES
('Hello world', '[0.1, 0.1, 0.3, 0.4]'),
('Document 1', '[0.1, 0.2, 0.3, 0.4]'),
('Document 2', '[0.4, 0.5, 0.6, 0.7]'),
('Document 3', '[0.4, 0.5, 0.6, 0.7]'),
('Document 4', '[0.5, 0.7, 0.6, 1.0]');
```

![ ](../../../../../docs/docs-images/p18/006.png)

#### ベクトル類似度検索

```bash
-- コサイン距離（テキスト埋め込みに推奨）
SELECT id, content, embedding <=> '[0.1, 0.2, 0.3, 0.3]'::vector as distance
FROM embeddings
ORDER BY embedding <=> '[0.1, 0.2, 0.3, 0.3]'::vector
LIMIT 2;
```

**説明**:

- `<=>` はコサイン距離演算子です
- `1 - distance` により類似度へ変換できます（0-1 の範囲で、値が大きいほど類似しています）

![ ](../../../../../docs/docs-images/p18/007.png)

#### ベクトルデータを更新

```sql
-- 特定レコードのベクトルを更新
UPDATE embeddings
SET embedding = array_fill(0.5, ARRAY[4])::vector
WHERE id = 1;

-- 更新を確認
SELECT id, content, embedding FROM embeddings WHERE id = 1;
```

![ ](../../../../../docs/docs-images/p18/008.png)

#### ベクトルデータを削除

```sql
-- 指定したレコードを削除
DELETE FROM documents WHERE id = 3;

-- 残りのレコード数を確認
SELECT COUNT(*) FROM embeddings;

-- すべてのレコードを削除
DELETE FROM embeddings;
```

### Python 接続例

システムツールで直接接続するだけでなく、Python プログラムに設定して利用することもできます。まず以下のコマンドで接続ツールをインストールします。

```bash
pip install psycopg2-binary
```

データベースからの読み取りテスト：

```bash
import psycopg2

conn = psycopg2.connect(
    host="172.172.1.1",
    port=3306,
    user="glowsai",
    password="xxxxx",
    dbname="my_project_db"
)

cur = conn.cursor()
cur.execute("SELECT * FROM embeddings;")
rows = cur.fetchall()

for row in rows:
    print(row)

cur.close()
conn.close()
```

![ ](../../../../../docs/docs-images/p18/009.png)

## 実践プロジェクト：LiteLLM + Glows Matrix0 db

LiteLLM は、OpenAI、Anthropic など複数の大規模モデルを統一的に呼び出すためのプロキシおよびツールレイヤーです。OpenAI 互換インターフェースを提供し、異なるモデル間の切り替えや管理を容易にします。実際のデプロイでは、LiteLLM は通常 PostgreSQL をデータベースとして組み合わせて利用します。PostgreSQL は安定性と信頼性が高く、高い同時接続数や複雑なクエリに対応し、pgvector などのベクトル拡張も利用できるため、ログや embeddings の保存、後続の分析や検索に適しています。

### Docker で LiteLLM をデプロイ

Docker に対応した CPU VM を作成します。最も簡単な方法は、Docker で LiteLLM を直接デプロイすることです。
Glowsai プラットフォームで図の手順に従い、以下を選択します。

`Create New` → `CPU` → `Ubuntu 24.04 Docker NV 580` イメージ

![ ](../../../../../docs/docs-images/p18/010.png)

### LiteLLM を起動

インスタンスの起動と接続が完了したら、以下の 3 つのファイルを作成するだけで LiteLLM をすばやく起動できます。

- `docker-compose.yml`：サービス起動設定
- `.env`：環境変数設定
- `config.yaml`：LiteLLM 設定ファイル

#### docker-compose.yml の設定

このファイルでは、コンテナで使用するイメージと起動パラメータを定義します。

```bash
services:
  litellm:
    image: docker.litellm.ai/berriai/litellm:main-stable
    container_name: litellm-gateway
    restart: unless-stopped
    env_file:
      - .env
    volumes:
      - ./config.yaml:/app/config.yaml:ro
    command: ["--config", "/app/config.yaml", "--port", "4001", "--num_workers", "4"]
    ports:
      - "0.0.0.0:4001:4001"
```

#### .env 環境変数の設定

サービスを起動する前に、`.env` ファイルへ以下の内容を追加してください。

```bash
# API マスターキー（API 呼び出しと LiteLLM WebUI ログインに使用）
LITELLM_MASTER_KEY=sk-glowsai

# PostgreSQL データベース接続文字列
DATABASE_URL=postgresql://glowsai:xxxxxxx@172.172.1.1:3306/postgres

# モデルデータをデータベースに保存するかどうか
STORE_MODEL_IN_DB=True
```

#### config.yaml の設定

`config.yaml` に以下の設定を追加します。

```yaml
general_settings:
  master_key: os.environ/LITELLM_MASTER_KEY
  store_model_in_db: true
```

### サービスを起動

上記の設定が完了したら、以下のコマンドを実行して LiteLLM を起動します。

```bash
# サービスを起動
docker compose up -d

# コンテナログを確認
docker logs -f litellm-gateway
```

LiteLLM は `.env` 内の `DATABASE_URL` に従って自動的にデータベースへ接続し、関連するテーブルを初期化して作成します。

![ ](../../../../../docs/docs-images/p18/011.png)

### データベースを検証

この時点で再度リモートデータベースへ接続し、`\dt` を入力すると、データベース内に LiteLLM 関連のテーブルが多数作成されていることを確認できます。

```sql
\dt
```

![ ](../../../../../docs/docs-images/p18/012.png)

サービスの起動が完了すると、LiteLLM の管理画面にも正常にログインでき、API Key の作成、モデル Provider やモデルの設定などを行えます。

![ ](../../../../../docs/docs-images/p18/013.png)

## FAQs

**1、現在の利用フローはどのようになりますか？**

まず当社に連絡して利用権限を有効化します（Glowsai Public IP が 1 つ割り当てられます）。次に、インスタンスを起動し、チュートリアルの手順に従って Bind Public IP を設定します。その後、インスタンス ID を当社にお知らせください。エンジニアがデータベースの内部ネットワーク接続を設定し、インスタンス内からの接続方法をお伝えします。

**2、リモートデータベース接続情報の IP と Port はカスタマイズできますか？**

はい。ご希望の IP と Port をお知らせいただければ設定できます。

**3、Glows.ai に LiteLLM をデプロイした場合、WebUI にはどのようにアクセスしますか？**

本チュートリアルに従って操作すると、LiteLLM サービスはデフォルトで 4001 Port にデプロイされます。インスタンス画面で `New Port Binding` をクリックし、Instance Service Port と Public IP Port を入力してから Create をクリックしてください。Port 設定が完了すると、Glowsai Public IP + Public IP Port を使用して、パブリックネットワークからインスタンス内のサービスへアクセスできます。

![ ](../../../../../docs/docs-images/p18/014.png)

## お問い合わせ

Glows.ai の利用中にご不明点やご提案がある場合は、メール、Discord、または Line でお気軽にお問い合わせください。

**Email:** [support@glows.ai](mailto:support@glows.ai)

**Discord:** [https://discord.com/invite/glowsai](https://discord.com/invite/glowsai)

**Line:** [https://lin.ee/fHcoDgG](https://lin.ee/fHcoDgG)