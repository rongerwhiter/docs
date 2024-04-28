# その他の知識

## DNS解決のタイムアウトとリトライの設定

ネットワークプログラミングでは、`gethostbyname`や`getaddrinfo`などを使用してドメイン名の解決を行いますが、これらの`C`関数にはタイムアウトパラメータは提供されていません。実際には、`/etc/resolv.conf`を変更してタイムアウトやリトライロジックを設定できます。

!> `man resolv.conf`ドキュメントを参照できます

### 複数のネームサーバー <!-- {docsify-ignore} -->

```
nameserver 192.168.1.3
nameserver 192.168.1.5
option rotate
```

複数の`nameserver`を設定でき、最初の`nameserver`でのクエリが失敗した場合に自動的に次の`nameserver`に切り替えてリトライします。

`option rotate`の設定は、`nameserver`の負荷分散を行い、ラウンドロビンモードを使用します。

### タイムアウト制御 <!-- {docsify-ignore} -->

```
option timeout:1 attempts:2
```

* `timeout`：UDP受信のタイムアウト時間（秒単位）、デフォルトは`5`秒
* `attempts`：試行回数を制御し、`2`に設定すると最大で`2`回の試行が行われます。デフォルトは`5`回

`nameserver`が2つあり、`attempts`が`2`、タイムアウトが`1`の場合、すべてのDNSサーバーが応答しない場合、最大待機時間は`4`秒になります（`2x2x1`）。

### コールトレース <!-- {docsify-ignore} -->

[strace](/other/tools?id=strace)を使用してトレースを確認できます。

`nameserver`を存在しない2つのIPに設定し、`PHP`コードで`var_dump(gethostbyname('www.baidu.com'));`を使用してドメイン名を解決します。

ここで、合計で`4`回のリトライが行われ、`poll`呼び出しのタイムアウトが`1000ms`（`1秒`）に設定されていることがわかります。
