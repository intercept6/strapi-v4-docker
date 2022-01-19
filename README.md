# Strapi v4をDockerで動かすサンプルリポジトリ

Strapi v4のDockerイメージは2022年1月19日時点では提供されていません。なので、独自にDockerイメージを作る方法を調査しました。このリポジトリはその成果物です。
Strapiはデータ構造の定義はローカル環境で、データの追加は本番環境で行う必要がある仕様なので、その運用についても考慮して作成しました。
データベースにはPostgreSQLを使用しています。

## Dockerイメージの作成

- /opt: `node_modules`を置くディレクトリ
- /opt/app: Strapiのアプリケーションを**マウント**するディレクトリ

マウントを行うディレクトリとは異なるディレクトリに`node_modules`を置く事で、マウントを行ってもイメージ作成時に生成された`node_modules`を参照出来るようになります。

```Dockerfile
WORKDIR /opt/
COPY package.json yarn.lock ./
ENV PATH /opt/node_modules/.bin:$PATH

RUN yarn config set network-timeout 600000 -g
RUN yarn install

WORKDIR /opt/app
COPY ./ .
RUN yarn build
```

## ローカル環境での起動

ローカル環境で利用する際にもPostgreSQLが必要になるので、docker-composeを使ってStrapiと一緒に立ち上げます。
ローカル環境では、`strapi develop`を呼び出す必要があるので、コマンドを`yarn develop`上書きして起動します。
初回起動時はPostgreSQLのセットアップに時間がかかり、リクエストの受け付けを始める前にStrapiが接続を試みて停止してしまうので、`depends_on > [container_name]] > condition`を`service_healthy`に設定してリクエストの受け付け開始を待ってからStrapiを起動します。ヘルスチェックには`pg_isready`を使用しています。