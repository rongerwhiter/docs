# Linux 信号リスト
## 完全対応表

| 信号      | 値       | デフォルトの動作 | 意味（信号の原因）                                   |
| --------- | -------- | --------------- | ------------------------------------------------- |
| SIGHUP    | 1        | Term            | 端末の切断またはプロセスの終了                           |
| SIGINT    | 2        | Term            | キーボードからの割り込みシグナル                           |
| SIGQUIT   | 3        | Core            | キーボードからの終了シグナル                             |
| SIGILL    | 4        | Core            | 不正な命令                                       |
| SIGABRT   | 6        | Core            | abort からの例外シグナル                            |
| SIGFPE    | 8        | Core            | 浮動小数点例外                                    |
| SIGKILL   | 9        | Term            | キル                                            |
| SIGSEGV   | 11       | Core            | セグメンテーション違反（無効なメモリ参照）                     |
| SIGPIPE   | 13       | Term            | パイプが壊れています：読み取りプロセスがいないパイプに書き込んでいます         |
| SIGALRM   | 14       | Term            | アラームからのタイマーが時間切れです                        |
| SIGTERM   | 15       | Term            | 終了                                           |
| SIGUSR1   | 30,10,16 | Term            | ユーザー定義シグナル 1                              |
| SIGUSR2   | 31,12,17 | Term            | ユーザー定義シグナル 2                              |
| SIGCHLD   | 20,17,18 | Ign             | 子プロセスが停止または終了です                           |
| SIGCONT   | 19,18,25 | Cont            | 停止している場合、継続します                            |
| SIGSTOP   | 17,19,23 | Stop            | 端末以外の停止シグナルです                             |
| SIGTSTP   | 18,20,24 | Stop            | 端末からの停止シグナルです                              |
| SIGTTIN   | 21,21,26 | Stop            | バックグラウンドプロセスが端末に読み取ります                    |
| SIGTTOU   | 22,22,27 | Stop            | バックグラウンドプロセスが端末に書き込みます                    |
|           |          |                 |                                             |
| SIGBUS    | 10,7,10  | Core            | バスエラー（メモリアクセスエラー）                         |
| SIGPOLL   |          | Term            | ポーリング可能なイベントが発生しました（Sys V と同義）                |
| SIGPROF   | 27,27,29 | Term            | プロファイル用のタイマーが時間切れです                        |
| SIGSYS    | 12,-,12  | Core            | 不正なシステムコール(SVr4)                          |
| SIGTRAP   | 5        | Core            | トレース/ブレークポイントトラップ                         |
| SIGURG    | 16,23,21 | Ign             | ソケットの緊急シグナル(4.2BSD)                        |
| SIGVTALRM | 26,26,28 | Term            | 仮想タイマーが時間切れです(4.2BSD)                        |
| SIGXCPU   | 24,24,30 | Core            | CPUの制限時間を超過しました(4.2BSD)                       |
| SIGXFSZ   | 25,25,31 | Core            | ファイルサイズ制限を超過しました(4.2BSD)                      |
|           |          |                 |                                             |
| SIGIOT    | 6        | Core            | IOT トラップ、SIGABRT と同義                        |
| SIGEMT    | 7,-,7    |                 | Term                                         |
| SIGSTKFLT | -,16,-   | Term            | 浮動小数点演算エラー(未使用)                          |
| SIGIO     | 23,29,22 | Term            | ディスクリプタ上で I/O 操作を行うことができます                |
| SIGCLD    | -,-,18   | Ign             | SIGCHLD と同義                                 |
| SIGPWR    | 29,30,19 | Term            | 電力障害(System V)                              |
| SIGINFO   | 29,-,-   |                 | SIGPWR と同義                                |
| SIGLOST   | -,-,-    | Term            | ファイルロックが失われました                           |
| SIGWINCH  | 28,28,20 | Ign             | ウィンドウサイズが変更されました(4.3BSD、Sun)               |
| SIGUNUSED | -,31,-   | Term            | 未使用シグナル（SIGSYS になります）                        |
## 非可靠信号

| 名称      | 说明                        |
| --------- | --------------------------- |
| SIGHUP    | 接続が切断されました         |
| SIGINT    | 端末中断                    |
| SIGQUIT   | 終了                        |
| SIGILL    | 不正なハードウェア命令       |
| SIGTRAP   | ハードウェアトラップ         |
| SIGABRT   | 異常終了(abort)             |
| SIGBUS    | ハードウェアエラー           |
| SIGFPE    | 算術例外                    |
| SIGKILL   | 終了                        |
| SIGUSR1   | ユーザー定義シグナル         |
| SIGUSR2   | ユーザー定義シグナル         |
| SIGSEGV   | 無効なメモリ参照            |
| SIGPIPE   | パイプに書き込まれましたが読み取るプロセスがない |
| SIGALRM   | タイマーが時間切れとなりました |
| SIGTERM   | 終了                        |
| SIGCHLD   | 子プロセスの状態が変化しました |
| SIGCONT   | 一時停止中のプロセスを再開   |
| SIGSTOP   | 中断                        |
| SIGTSTP   | 端末停止                    |
| SIGTTIN   | バックグラウンドプロセスが tty を読む |
| SIGTTOU   | バックグラウンドプロセスが tty に書き込む |
| SIGURG    | 緊急(ソケット)              |
| SIGXCPU   | CPU 利用制限を超えました     |
| SIGXFSZ   | ファイルサイズ制限を超えました |
| SIGVTALRM | 仮想時間アラーム             |
| SIGPROF   | プロファイル時間が終了しました |
| SIGWINCH  | 端末ウィンドウサイズが変更されました |
| SIGIO     | 非同期 I/O                  |
| SIGPWR    | 電源障害/再起動             |
| SIGSYS    | 無効なシステムコール         |
```plaintext
## Reliable Signals

| Name        | User-defined |
| ----------- | ------------ |
| SIGRTMIN    |              |
| SIGRTMIN+1  |              |
| SIGRTMIN+2  |              |
| SIGRTMIN+3  |              |
| SIGRTMIN+4  |              |
| SIGRTMIN+5  |              |
| SIGRTMIN+6  |              |
| SIGRTMIN+7  |              |
| SIGRTMIN+8  |              |
| SIGRTMIN+9  |              |
| SIGRTMIN+10 |              |
| SIGRTMIN+11 |              |
| SIGRTMIN+12 |              |
| SIGRTMIN+13 |              |
| SIGRTMIN+14 |              |
| SIGRTMIN+15 |              |
| SIGRTMAX-14 |              |
| SIGRTMAX-13 |              |
| SIGRTMAX-12 |              |
| SIGRTMAX-11 |              |
| SIGRTMAX-10 |              |
| SIGRTMAX-9  |              |
| SIGRTMAX-8  |              |
| SIGRTMAX-7  |              |
| SIGRTMAX-6  |              |
| SIGRTMAX-5  |              |
| SIGRTMAX-4  |              |
| SIGRTMAX-3  |              |
| SIGRTMAX-2  |              |
| SIGRTMAX-1  |              |
| SIGRTMAX    |              |
``` 

## 信頼性の高い信号
