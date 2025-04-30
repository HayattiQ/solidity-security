[<img width="200" alt="get in touch with Consensys Diligence" src="https://user-images.githubusercontent.com/2865694/56826101-91dcf380-685b-11e9-937c-af49c2510aa0.png">](https://consensys.net/diligence/)<br/>
<sup>
[[  🌐  ](https://consensys.net/diligence/)  [  📩  ](mailto:diligence@consensys.net)  [  🔥  ](https://consensys.net/diligence/tools/)]
</sup><br/><br/>


# スマートコントラクトセキュリティのベストプラクティス

ドキュメントサイトを訪問: https://consensysdiligence.github.io/smart-contract-best-practices/

中国語でドキュメントを読む: https://github.com/ConsenSysDiligence/smart-contract-best-practices/blob/master/README-zh.md
ベトナム語でドキュメントを読む: https://github.com/ConsenSysDiligence/smart-contract-best-practices/blob/master/README-vi.md

## 貢献大歓迎！

小さな修正から、新しいセクション全体まで、どんなものでもプルリクエストを自由に提出してください。新しいコンテンツを書いている場合は、スタイルのガイダンスについて[contributing](./docs/about/index.md)ページを参照してください。

カバーまたは更新する必要があるトピックについては、[issues](https://github.com/ConsenSysDiligence/smart-contract-best-practices/issues)を参照してください。議論したいアイデアがある場合は、[Gitter](https://gitter.im/ConsenSys/smart-contract-best-practices)でチャットしてください。

## ドキュメントサイトの構築

```
git clone git@github.com:ConsenSys/smart-contract-best-practices.git
cd smart-contract-best-practices
pip install -r requirements.txt
mkdocs build 
```

サーバーを実行するには（失敗時に再起動）:

```
until mkdocs serve; do :; done
```

`mkdocs serve`コマンドを使用して、localhostでサイトを表示し、変更を保存するたびにライブリロードすることもできます。

## ドキュメントサイトの再デプロイ

```
mkdocs gh-deploy
