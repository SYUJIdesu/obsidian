---
title: Node(typescript) + docker 環境構築メモ
source: https://zenn.dev/gakin/scraps/4cc16e7761d1ef
author:
  - "[[Zenn]]"
published: 2022/01/06にコメント追加
created: 2025-05-15
description: aGlnYWtpbg==さんのスクラップ
tags:
  - web
  - Docker
---
## Dockerfile修正 ~ コード実行

```shell
mkdir src && touch src/app.ts
```

app.ts

```typescript
console.log("Hello, world!")
```

## package.json内のscriptを追加

package.json

```json
{
  // .....
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "npx ts-node-dev --respawn src/app.ts"
  },
  // ......
}
```

## Dockerfileを修正

Dockerfile

```dockerfile
FROM node:12-alpine3.12

WORKDIR /app

COPY package*.json ./
RUN npm install

CMD npm run dev
```

## コンテナ環境を再起動(再ビルド)

```shell
docker-compose up -d --build
```

```shell
# コンテナ環境のログを確認
docker-compose logs -f
```

以下のように hello worldが表示されていたら成功。

```shell
node-dev_1  |
node-dev_1  | > app@1.0.0 dev /app
node-dev_1  | > npx ts-node-dev --respawn src/app.ts
node-dev_1  |
node-dev_1  | [INFO] 15:29:47 ts-node-dev ver. 1.1.8 (using ts-node ver. 9.1.1, typescript ver. 4.5.4)
node-dev_1  | Hello, world!
```

## 開発時など(コンテナ環境内でコマンドを実行したい時)

docker-compose.yml

```yml
version: '3'
services:
  node-dev:
    build: 
      context: .
      dockerfile: Dockerfile
    tty: true
    volumes:
      - .:/app
    entrypoint: >
      /bin/sh -c "sleep 86400"
```

## コンテナ環境内に入って、任意のプロセスを実行

```shell
docker-compose up -d
# コンテナ環境内に入る
docker-compose exec node-dev sh

# コンテナ環境内で任意のコマンドを実行
npm run dev
```

以下のように `Hello, world!`が表示されたら成功。

```yml
> app@1.0.0 dev /app
> npx ts-node-dev --respawn src/app.ts

[INFO] 15:35:51 ts-node-dev ver. 1.1.8 (using ts-node ver. 9.1.1, typescript ver. 4.5.4)
Hello, world!
```

## Expressの利用例

## docker-composeファイルを修正

docker-compose.yml

```yml
version: '3'
services:
  node-dev:
    build: 
      context: .
      dockerfile: Dockerfile
    tty: true
    volumes:
      - .:/app
    ports:
      - 3001:3000
    entrypoint: >
      /bin/sh -c "sleep 86400"
```

## app.tsを修正

## コンテナ内で必要パッケージinstallとapiサーバー起動

```shell
docker-compose up -d
docker-compose exec node-dev sh

# コンテナ内に必要パッケージをinstall
npm install -D @types/node
npm install express
npm install -D @types/express

npm run dev
```

以下のように出力されていることを確認

```yml
> app@1.0.0 dev /app
> npx ts-node-dev --respawn src/app.ts

[INFO] 15:45:34 ts-node-dev ver. 1.1.8 (using ts-node ver. 9.1.1, typescript ver. 4.5.4)
Start on port 3000.
```

## コンテナ外で、apiサーバーの疎通確認

```shell
curl -sS localhost:3001/users | jq .
```

以下のように結果を出力されていればOK

```json
[
  {
    "id": 1,
    "name": "User1",
    "email": "user1@test.local"
  },
  {
    "id": 2,
    "name": "User2",
    "email": "user2@test.local"
  },
  {
    "id": 3,
    "name": "User3",
    "email": "user3@test.local"
  }
]
```