---
title: 初めてのプライベートリポジトリへの GitHub Actions 導入
tags:
  - GitHubActions
private: false
updated_at: '2025-09-21T17:46:53+09:00'
id: e465ecc1122dd395c7da
organization_url_name: null
slide: false
ignorePublish: false
---

## はじめに

この記事では、Github Actions ワークフローをプライベートリポジトリ上で実行させる上でポイントだった点を紹介します。

### ワークフローの全体像

以下は、今回のワークフローで使用するコードです。このワークフローは、PRが作成または更新された際に、PRに `will-be-deployed` というラベルが付けられている場合のみ、ビルドとデプロイを実行します。
このコードは、[vite のスターターコード](https://ja.vite.dev/guide/#%E6%9C%80%E5%88%9D%E3%81%AE-vite-%E3%83%95%E3%82%9A%E3%83%AD%E3%82%B7%E3%82%99%E3%82%A7%E3%82%AF%E3%83%88%E3%82%92%E7%94%9F%E6%88%90%E3%81%99%E3%82%8B)を使ってテストコードを追加したのみのプロジェクトを動作確認のために使用しました。

```yaml
name: Deploy website
on:
  pull_request:
    types: [opened, synchronize, labeled]
env:
  # Use github.base_ref for PR events to get the target branch name.
  BRANCH_NAME: ${{ github.base_ref }}
permissions:
  pull-requests: write
  contents: read
  packages: write
jobs:
  build-and-deploy:
    # Run the job only if the PR contains the label "will-be-deployed"
    if: contains(github.event.pull_request.labels.*.name, 'will-be-deployed')
    runs-on: ubuntu-latest
    steps:
      - name: Repository Checkout
        uses: actions/checkout@v4

      - name: Check The Target Branch
        run: echo "$BRANCH_NAME"

      - name: Setup NodeJS
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: "yarn"

      - name: Check Yarn Global Bin & PATH
        run: |
          echo "Yarn global bin is: $(yarn global bin)"
          echo "Current PATH: $PATH"

      - name: Install Dependencies
        run: yarn install --frozen-lockfile

      - name: Run test
        run: yarn run test

      - name: Build project
        run: yarn run build

      - name: Post comment on PR
        uses: peter-evans/create-or-update-comment@v2
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          issue-number: ${{ github.event.pull_request.number }}
          body: |
            **Workflow completed successfully!**
```

## 詰まったポイント

### 権限の設定

ワークフロー実行を試している時に「Repository Checkout」セクションのところで、なぜか「remote: Repository not found.」というエラーが出ていていました。原因と回避方法を調べていたら、github actions checkout の公式リポジトリの `Issue #254` で下記のコメントを見つけました。

**_what you need to do is simply add the Permission for you Github action Job to read from your private repo._**

つまり、
**_プライベートリポジトリ向けのワークフロー実行には、権限設定が必要。_**

コメントの内容を参考に下記の通りに設定したところ、エラーが出ずにワークフローを実行できるようになりました。

```yaml
permissions:
  pull-requests: write
  contents: read
  packages: write
```

公式リンクは下記。
https://docs.github.com/ja/actions/writing-workflows/choosing-what-your-workflow-does/controlling-permissions-for-github_token

補足：上記コメントに加え、さらに最低限「contents: read」があれば、今回のケースの中での該当のエラーを回避できることをテストしました。

## おまけ

### 各セクションの解説

#### 1. トリガー設定

```
on:
pull_request:
types: [opened, synchronize, labeled]
```

::: note warn
このワークフローは、PRが新規作成されたり、更新（synchronize）されたり、ラベルが変更されたときに実行されます。
ラベルの変更イベントも対象にしているため、PRに will-be-deployed というラベルが付与された時にのみジョブが実行されます。
:::

#### 2. 環境変数の設定

```
BRANCH_NAME: ${{ github.base_ref }}
```

::: note warn
PRイベントの場合、ターゲットブランチ名を取得するために github.base_ref を利用しています。
:::

::: note info
これにより、デプロイ先ブランチを動的に決定でき、より柔軟な運用が可能です。
:::

#### 3. ジョブ「build-and-deploy」

```
jobs:
  build-and-deploy:
    if: contains(github.event.pull_request.labels.\*.name, 'will-be-deployed')
```

::: note warn
ジョブは、指定された条件（PRにwill-be-deployedラベルが含まれる）を満たす場合のみ実行されます。
:::

::: note info
ポイント:
条件付き実行により、不要なビルドやデプロイメントを避け、CIリソースを効率的に利用できます。
:::

#### 4. ステップごとの詳細 (各ステップの名前ごとに項目化しています)

##### Repository Checkout

actions/checkout を利用してリポジトリのコードを取得します。

##### Check The Target Branch

環境変数に設定したターゲットブランチ名を表示。

run: echo "$BRANCH_NAME"

##### Setup NodeJS

Node.js（バージョン20）とYarnのキャッシュ機能をセットアップします。

```
uses: actions/setup-node@v4
with:
node-version: 20
cache: "yarn"
```

##### Check Yarn Global Bin & PATH:

YarnのグローバルバイナリのパスとシステムPATHを確認し、環境の状態を検証します。

###### Install Dependencies:

yarn install --frozen-lockfile により、依存パッケージを正確なバージョンでインストールします。

::: note info
--frozen-lockfile オプションを使用することで、lockfileに記載されているバージョンから変更が加えられないように保証します。
:::

##### Run test

テストを実行し、コードの品質をチェックします。

##### Build project

プロジェクトのビルドを行います。

::: note info
ビルドが成功することで、デプロイメント可能な状態に変換されます。
:::

##### Post comment on PR

最後に、peter-evans/create-or-update-comment アクションを使用して、PRに成功のメッセージをコメントします。

::: note info
コメントにより、開発者はワークフローの結果を簡単に確認できます。とても便利な機能だと思います。
:::

## まとめ

この記事では、GitHub Actionsを利用して、プライベートリポジトリ上で実行されるワークフローの設定方法を中心に解説しました。
他にもすぐに有用になりそうな点や、特にyarnを使っているプロジェクトで応用可能な点も、記載しましたので参考になれば幸いです。
