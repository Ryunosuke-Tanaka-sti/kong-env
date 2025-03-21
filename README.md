# Kongに関しての調査

## 参考資料

- [https://docs.konghq.com/gateway/latest/install/docker/#main]
- [https://jp.konghq.com/blog/prometheus-grafana-kong-gateway]
- [https://github.com/Kong/docker-kong/blob/master/compose/README.md]

## 概要調査

### APIの構築

#### APIテスト

APIの機能、効率性、データの正確性をテストする環境を提供します。
開発者はこの機能を使用して、本番環境にデプロイする前にAPIの品質を確認できます。

#### データセキュリティ

IPホワイトリスト、攻撃緩和、データ暗号化などのセキュリティプラクティスをサポートします。
これにより、APIを通じて送受信されるデータの保護が強化されます。

#### オーケストレーション

複数のバックエンドリソースやデータベースを利用するAPIの作成が可能です。
複雑なシステム間の連携を簡素化し、効率的なAPIの設計を支援します。

### API管理

#### トラフィック制御

不審な訪問者のアクセスを制限し、DDoS攻撃などの過負荷を防ぐためのトラフィックスパイクを監視します。
APIの安定性と可用性を維持するのに役立ちます。

#### ログ/ドキュメンテーション

使用状況や機能に関する詳細を記録し、分析とレポート作成に活用できます。
これにより、APIの性能や利用パターンを把握し、改善点を特定できます。

#### APIモニタリング

機能、ユーザーアクセシビリティ、トラフィックフロー、改ざんの異常を検出します。
リアルタイムでAPIの健全性を監視し、問題を早期に発見できます。

### データ統合

#### アプリケーション統合

APIをサードパーティアプリケーションと統合し、パフォーマンスを向上させる機能を提供します。
これにより、異なるシステム間のシームレスな連携が可能になります。

#### データ変換

複雑なデータセットやバックエンドシステムを、アプリケーションが解釈できる形式に変換します。
データの互換性の問題を解決し、スムーズなデータ流通を実現します。

### 開発

#### スケーラビリティ

機能を拡張しながら、負荷のバランスを維持します。
需要の増加に対応しつつ、機能を低下させることなくサービスを提供できます。

### 実装の流れ

1. Kongのインストール
   - DBが必要でありそう
   - DBレスモードあり
2. 設定ファイルの作成
   - Proxy周りの設定
3. Kongの起動
   - DBの初期化と起動
4. APIリソースの管理
   - Kongの管理用APIを使用したエンドポイントの追加
5. セキュリティの追加
   - APIキー認証などの設定
6. モニタリングとログの設定
   - Kongのログ機能でAPIを監視できる

## Docker Compose環境での構築

- [公式リポジトリ](https://github.com/Kong/kong)
- [https://github.com/Kong/docker-kong/blob/master/compose/README.md]

公式リポジトリから紹介されている、リポジトリにcomposeファイルがあったのを発見
参考に構成を検討

以下ファイルで起動確認

```yml

version: "3.8"

services:
  kong-database:
    image: postgres:13
    environment:
      POSTGRES_USER: kong
      POSTGRES_DB: kong
      POSTGRES_PASSWORD: kongpass
    volumes:
      - type: volume
        source: kong_data
        target: /var/lib/postgresql/data
    networks:
      - kong-net

  kong:
    image: kong:latest
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: kong-database
      KONG_PG_USER: kong
      KONG_PG_PASSWORD: kongpass
      KONG_PROXY_ACCESS_LOG: /dev/stdout
      KONG_ADMIN_ACCESS_LOG: /dev/stdout
      KONG_PROXY_ERROR_LOG: /dev/stderr
      KONG_ADMIN_ERROR_LOG: /dev/stderr
      KONG_ADMIN_LISTEN: 0.0.0.0:8001
    depends_on:
      - kong-database
    ports:
      - "8000:8000"
      - "8001:8001"
    networks:
      - kong-net

networks:
  kong-net:
    driver: bridge

volumes:
  kong_data:

```

動作未確：Docker Composeの起動まで確認

```bash

# 初期化
docker compose up -d kong-database
docker compose run --rm kong kong migrations bootstrap

# 全体起動
docker compose up -d

```

起動確認
/api配下にレスポンスを含めてまとめる

```bash
# 管理用エンドポイントの確認
curl -i http://localhost:8001

# サービスの追加 http://mockbin.orgに対して登録
curl -i -X POST --url http://localhost:8001/services/ --data 'name=example-service' --data 'url=http://mockbin.org'

# ルートの追加 http://mockbin.org
curl -i -X POST --url http://localhost:8001/services/example-service/routes --data 'paths[]=/example'

# ルートのテスト
curl -i -X GET --url http://localhost:8000/example
```

---

要確認事項

実際に動作するAPIへのルートがどのように動作するかわからないため実際に作成して確認する必要がある。
flaskでダミーのAPIを作成して確認