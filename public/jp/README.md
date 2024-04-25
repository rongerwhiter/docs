# Swoole

?> `Swoole` は、`C++` 言語で書かれた非同期イベント駆動とコルーチンに基づく並列ネットワーク通信エンジンであり、`PHP` に[コルーチン](/coroutine)および[高性能](/question/use?id=how-is-the-performance-of-swoole)ネットワークプログラミングサポートを提供します。さまざまな通信プロトコルのネットワークサーバーおよびクライアントモジュールを提供し、`TCP/UDPサービス`、`高性能Web`、`WebSocketサービス`、`IoT`、`リアルタイム通信`、`ゲーム`、`マイクロサービス`などを簡単かつ迅速に実装でき、`PHP`を従来のWeb領域にとどまらないようにします。

## Swooleクラス図

!>リンクをクリックして該当のドキュメントページに移動できます

[//]: # (https://naotu.baidu.com/file/bd9d2ba7dfae326e6976f0c53f88b18c)

<embed src="/_images/swoole_class.svg" type="image/svg+xml" alt="Swoole Architecture Diagram" />

## 公式ウェブサイト

* [Swoole Official Website](//www.swoole.com)
* [Commercial Products and Support](//business.swoole.com)
* [Swoole Q&A](//wenda.swoole.com)

## プロジェクトのリンク

* [GitHub](//github.com/swoole/swoole-src) **（サポートする場合はStarをクリックしてください）**
* [Gitee](//gitee.com/swoole/swoole)
* [PECL](//pecl.php.net/package/swoole)

## 開発ツール

* [IDE Helper](https://github.com/swoole/ide-helper)
* [Yasd](https://github.com/swoole/yasd)
* [debugger](https://github.com/swoole/debugger)

## 著作権情報

このドキュメントの元の内容は以前の [旧版Swooleドキュメント](https://wiki.swoole.com/wiki/index/prid-1) から引用されており、ずっと問題とされていたドキュメントの問題を解決することを目的としています。現代的な文書構成形式を採用し、`Swoole4`の内容のみを含んでおり、古い文書の間違いを多く修正し、文書の詳細を最適化し、例と教育的なコンテンツを追加し、`Swoole`の初心者にとって親しみやすくなりました。

このドキュメントのすべての内容、すべてのテキスト、画像、音声および動画資料は **上海シウォネットワークテクノロジー有限公司** に著作権があり、メディア、Webサイト、個人は外部リンク形式で引用できますが、協定授権なしにいかなる形式でもコピー、公開、発表できません。

## ドキュメントの起草者

* 杨才 [GitHub](https://github.com/TTSimple)
* 郭新华 [Weibo](https://www.weibo.com/u/2661945152)
* [鲁飞](https://github.com/sy-records) [Weibo](https://weibo.com/5384435686)

## 問題のフィードバック

このドキュメントに関する内容の問題（誤字、例の間違い、内容の不足など）および要望は、すべて [swoole-inc/report](https://github.com/swoole-inc/report) プロジェクトに統一して `issue` を提出してください。また、右上の [フィードバック](/?id=main) をクリックして直接`issue`ページに移動することもできます。

採用された場合、提出者の情報は[貢献者](/CONTRIBUTING) リストに追加され、感謝の意を示します。

## ドキュメントの原則

わかりやすい言葉を使います。`Swoole`の低レベル技術の詳細といくつかの低レベルの概念を**できるだけ**少なく紹介します。後続の低レベルに関することは、専用の`hack`セクションを維持する必要があります。

どうしても回避できない概念がある場合、この概念についての集中的な紹介が**必ず**必要であり、他の場所への内部リンクが必要です。例：[イベントループ](/learn?id=什么是eventloop)；

ドキュメントを作成する際には、他の人が理解できるかどうかを初心者の視点で考えます。

後続の機能変更が発生した場合は**必ず**すべての関連箇所を修正し、1つの箇所のみを修正することはできません。

各機能モジュールには、完全な例が**必ず**必要です。
