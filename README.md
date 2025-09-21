# qiita-content

## 1. セットアップ手順

### [Qiita 公式の記事](https://qiita.com/Qiita/items/32c79014509987541130) にそってセットアップを行う

- 項目「[3. Qiita CLIをセットアップする](https://qiita.com/Qiita/items/32c79014509987541130#3-qiita-cli%E3%82%92%E3%82%BB%E3%83%83%E3%83%88%E3%82%A2%E3%83%83%E3%83%97%E3%81%99%E3%82%8B)」で、[公式の Qiita CLI サイト](https://github.com/increments/qiita-cli) へ手順を誘導している。基本的に手順に沿えば問題なく設定できる。

- 最初の記事を書いた後、`npx qiita publish --all` で公開ができれば、問題なくセットアップできている想定。

- 私の場合は、ページの最初に書く yaml 設定の tags に指定した値にスペースが含まれていて、下記のようなエラーが出ていました。同じエラーが出た場合は、yaml 設定を疑うと良いかもしれないです。

```
エラーが発生しました (unknown or unexpected option: --help)
  バグの可能性がある場合は、Qiita Discussionsよりご報告いただけると幸いです
  https://github.com/increments/qiita-discussions
```

## 2. 管理を行うにあたってのポイント

- `npx qiita publish --all` コマンドで公開した後、Github リポジトリにプッシュするとしたら、`npx qiita pull` を実施しておいた方が良い認識。理由としては、publish コマンドによる公開の後、投稿した記事のページの yaml 設定項目の **id** と **updated_at** が変わるため。

## 3. これから行いたいこと

1. PRI の organization を作成し、投稿した記事を PRI の organization に紐づけられるようにする。
