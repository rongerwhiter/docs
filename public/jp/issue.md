# エラーレポートを提出する

## 事前に知るべきこと

Swooleのコアのバグを見つけたと感じた場合は、レポートを提出してください。Swooleのコア開発者は問題の存在をまだ知らないかもしれませんが、報告を提出しない限り、バグは発見され修正されにくくなるでしょう。GitHubのissueエリア（ https://github.com/swoole/swoole-src/issues ）でエラーレポートを提出することができます（右上の「New issue」ボタンをクリックしてください）。ここでのエラーレポートは最優先で解決されます。

エラーレポートはメーリングリストや個人メールでは送らないでください。GitHubのissueエリアでもSwooleに関する要望や提案を行うことができます。

エラーレポートを提出する前に、以下の**エラーレポートの提出方法**をよく読んでください。

## Issueの作成

Issueを作成する際には、システムが以下のテンプレートを提供するので、十分に入力してください。情報が不足しているとIssueが無視されることがあります：

```markdown

Please answer these questions before submitting your issue. Thanks!
> 提出する前に、以下の質問にお答えください。ありがとうございます！
    
1. What did you do? If possible，provide a simple script for reproducing the error.
> 問題が発生した手順を詳細に説明してください。関連するコードを貼り付けてください。可能であれば、エラーを再現できる簡単なスクリプトを提供してください。

2. What did you expect to see?
> 何が予想されるのか？

3. What did you see instead?
> 代わりに何が表示されましたか？

4. What version of Swoole are you using (`php --ri swoole`)?
> 使用しているSwooleのバージョンは何ですか？ (`php --ri swoole` の出力を貼り付けてください)

5. What is your machine environment used (including the version of kernel & php & gcc)?
> 使用しているマシンの環境は何ですか（カーネルのバージョン、PHP、gccのバージョンを含む）？
> `uname -a`、`php -v`、`gcc -v` コマンドで確認できます

```

特に重要なのは**再現できる簡単なスクリプトを提供**することです。そうでない場合は、開発者がエラーの原因を判断するのに他の情報を提供する必要があります。

## メモリ解析（強く推奨）

通常、Valgrindはgdbよりもメモリ問題を見つけるのに役立ちます。以下のコマンドを使用してプログラムを実行し、エラーが発生するまで実行してください。

```shell
USE_ZEND_ALLOC=0 valgrind --log-file=/tmp/valgrind.log php your_file.php
```

* プログラムでエラーが発生した場合は `ctrl+c` を入力し、`/tmp/valgrind.log` ファイルをアップロードして開発チームに問題を特定させてください。

## セグメンテーション違反（コアダンプ）

また、特定の場合には、デバッグツールを使用して問題を特定するのに役立ちます

```shell
WARNING	swManager_check_exit_status: worker#1 abnormal exit, status=0, signal=11
```

上記のようなSwooleログに(signal11) が表示された場合、プログラムが`コアダンプ`を起こしたことを示します。問題の発生位置を特定するためにデバッグツールを使用する必要があります

> `gdb` を使用して `swoole`をトレースする前に、より多くの情報を保存するためにコンパイル時に `--enable-debug` パラメーターを追加してください

コアダンプファイルを有効にする
```shell
ulimit -c unlimited
```

問題をトリガーすると、コアダンプファイルがプログラムディレクトリまたはシステムのルートディレクトリまたは `/cores` ディレクトリに生成されます（システムの構成に応じて異なります）

以下のコマンドを入力し、gdbでプログラムをデバッグします

```
gdb php core
gdb php /tmp/core.1234
```

次に `bt` を入力してエンターキーを押すと、問題の呼び出しスタックが表示されます
```
(gdb) bt
```

`f 数字` を入力して特定の呼び出しスタックフレームを確認できます
```
(gdb) f 1
(gdb) f 0
```

これらの情報をIssueにすべて貼り付けてください
