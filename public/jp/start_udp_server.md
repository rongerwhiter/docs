# UDP サーバー

## プログラムコード

以下のコードを`udpServer.php`に記述してください。

```php
$server = new Swoole\Server('127.0.0.1', 9502, SWOOLE_PROCESS, SWOOLE_SOCK_UDP);

//データ受信イベントを監視。
$server->on('Packet', function ($server, $data, $clientInfo) {
    var_dump($clientInfo);
    $server->sendto($clientInfo['address'], $clientInfo['port'], "Server：{$data}");
});

//サーバーを起動
$server->start();
```

UDPサーバーはTCPサーバーとは異なり、UDPには接続の概念がありません。サーバーを起動した後、クライアントはConnectする必要はなく、直接サーバーが監視するポート9502にデータパケットを送信できます。これに対応するイベントは`onPacket`です。

* `$clientInfo`はクライアントの関連情報であり、クライアントのIPやポートなどが含まれる配列です。
* `$server->sendto`メソッドを使用してクライアントにデータを送信します。

!> DockerはデフォルトでTCPプロトコルを使用して通信します。UDPプロトコルを使用する場合は、Dockerネットワークを構成する必要があります。  
```shell
docker run -p 9502:9502/udp <image-name>
```

## サービスを起動

```shell
php udpServer.php
```

UDPサーバーは`netcat -u`を使用して接続してテストできます。

```shell
netcat -u 127.0.0.1 9502
hello
Server: hello
```
