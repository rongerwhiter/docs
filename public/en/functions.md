# Function List

Swoole provides some functions for PHP programs to retrieve system information in addition to functions related to network communication.
## swoole_set_process_name()

Used to set the name of a process. After modifying the process name, what will be seen through the ps command will no longer be `php your_file.php`, but the specified string.

This function takes a string parameter.

This function is the same as the `cli_set_process_title` function provided by PHP 5.5. However, `swoole_set_process_name` can be used on any version above PHP 5.2. The compatibility of `swoole_set_process_name` is lower than that of `cli_set_process_title`, so if the `cli_set_process_title` function exists, it is preferable to use `cli_set_process_title`.

```php
function swoole_set_process_name(string $name): void
```

Example of usage:

```php
swoole_set_process_name("swoole server");
```
### How to rename each process of Swoole Server <!-- {docsify-ignore} -->

* Modify the name of the main process when [onStart](/server/events?id=onstart) is called.
* Modify the name of the manager process when [onManagerStart](/server/events?id=onmanagerstart) is called.
* Modify the name of the worker process when [onWorkerStart](/server/events?id=onworkerstart) is called.

!> Older versions of the Linux kernel and Mac OSX do not support process renaming.
## swoole_strerror()

Convert error code to error message.

Function prototype:

```php
function swoole_strerror(int $errno, int $error_type = 1): string
```

Error types:

* `1`: Standard `Unix Errno` generated by system call errors such as `EAGAIN`, `ETIMEDOUT`, etc.
* `2`: Error code from `getaddrinfo`, generated by `DNS` operations.
* `9`: Swoole's underlying error code, obtained using `swoole_last_error()`.

Usage example:

```php
var_dump(swoole_strerror(swoole_last_error(), 9));
```
## swoole_version()

Get the version number of the Swoole extension, such as `1.6.10`

```php
function swoole_version(): string
```

Usage example:

```php
var_dump(SWOOLE_VERSION); // The global variable SWOOLE_VERSION also represents the Swoole extension version
var_dump(swoole_version());
/**
Returns:
string(6) "1.9.23"
string(6) "1.9.23"
**/
```
## swoole_errno()

Get the error code of the most recent system call, equivalent to the `errno` variable in `C/C++`.

```php
function swoole_errno(): int
```

The value of the error code is system-dependent. You can use `swoole_strerror` to convert the error into an error message.
## swoole_get_local_ip()

This function is used to obtain the IP addresses of all network interfaces on the local machine.

```php
function swoole_get_local_ip(): array
```

Usage example:

```php
// Get the IP addresses of all network interfaces on the local machine
$list = swoole_get_local_ip();
print_r($list);
/**
Returned value
Array
(
      [eno1] => 10.10.28.228
      [br-1e72ecd47449] => 172.20.0.1
      [docker0] => 172.17.0.1
)
**/
```

!> Note
* Currently, it returns only IPv4 addresses and filters out the local loop address 127.0.0.1 from the results.
* The result array is an associative array with the interface name as the key. For example, `array("eth0" => "192.168.1.100")`
* This function calls the `ioctl` system call in real-time to obtain interface information, without any caching at the lower level.
## swoole_clear_dns_cache()

Clears the built-in DNS cache of Swoole, which is effective for `swoole_client` and `swoole_async_dns_lookup`.

```php
function swoole_clear_dns_cache()
```
## swoole_get_local_mac()

Get the `Mac` addresses of the local network cards.

```php
function swoole_get_local_mac(): array
```

* Returns the `Mac` addresses of all network cards upon successful call.

```php
array(4) {
  ["lo"]=>
  string(17) "00:00:00:00:00:00"
  ["eno1"]=>
  string(17) "64:00:6A:65:51:32"
  ["docker0"]=>
  string(17) "02:42:21:9B:12:05"
  ["vboxnet0"]=>
  string(17) "0A:00:27:00:00:00"
}
```
## swoole_cpu_num()

Obtain the number of CPU cores on the local machine.

```php
function swoole_cpu_num(): int
```

* Returns the number of CPU cores if successful. For example:

```shell
php -r "echo swoole_cpu_num();"
```
## swoole_last_error()

Get the last error code from the Swoole underlying layer.

```php
function swoole_last_error(): int
```

You can use `swoole_strerror(swoole_last_error(), 9)` to convert the error code to error information. For a complete list of error information, please refer to the [Swoole error code list](/other/errno?id=swoole).
## swoole_mime_type_add()

Add new MIME type to the built-in MIME type table.

```php
function swoole_mime_type_add(string $suffix, string $mime_type): bool
```
## swoole_mime_type_set()

Modify a specific MIME type. Return `false` if failed (e.g. if it does not exist).

```php
function swoole_mime_type_set(string $suffix, string $mime_type): bool
```
- Delete a certain MIME type, return `false` on failure (such as if it does not exist).

```php
function swoole_mime_type_delete(string $suffix): bool
```
## swoole_mime_type_get()

Get the MIME type corresponding to the filename.

```php
function swoole_mime_type_get(string $filename): string
```
### swoole_mime_type_exists()

Check if the MIME type corresponding to the given suffix exists.

```php
function swoole_mime_type_exists(string $suffix): bool
```
## swoole_substr_json_decode()

Zero-copy JSON deserialization, all parameters except `$offset` and `$length` are consistent with [json_decode](https://www.php.net/manual/en/function.json-decode.php).

!> Available in Swoole version >= `v4.5.6`, starting from version `v4.5.7` it requires [--enable-swoole-json](/environment?id=common-parameters) to be added at compile time. For usage scenarios, refer to [Swoole 4.5.6 supports zero-copy JSON or PHP deserialization](https://wenda.swoole.com/detail/107587)

```php
function swoole_substr_json_decode(string $packet, int $offset, int $length, bool $assoc = false, int $depth = 512, int $options = 0)
```

  * **Example**

```php
$val = json_encode(['hello' => 'swoole']);
$str = pack('N', strlen($val)) . $val . "\r\n";
$l = strlen($str) - 6;
var_dump(json_decode(substr($str, 4, $l), true));
var_dump(swoole_substr_json_decode($str, 4, $l, true));
```
## swoole_substr_unserialize()

Zero-copy PHP unserialization, all parameters are the same as [unserialize](https://www.php.net/manual/en/function.unserialize.php) except for `$offset` and `$length`.

!> Available starting from Swoole version >= `v4.5.6`. For usage scenarios, refer to [Swoole 4.5.6 supports zero-copy JSON or PHP unserialization](https://wenda.swoole.com/detail/107587)

```php
function swoole_substr_unserialize(string $packet, int $offset, int $length, array $options= [])
```

  * **示例 (Example)**

```php
$val = serialize('hello');
$str = pack('N', strlen($val)) . $val . "\r\n";
$l = strlen($str) - 6;
var_dump(unserialize(substr($str, 4, $l)));
var_dump(swoole_substr_unserialize($str, 4, $l));
```
## swoole_error_log()

Outputs error information to the log. `$level` is the [log level](/consts?id=日志等级).

!> Available in Swoole version >= `v4.5.8`

```php
function swoole_error_log(int $level, string $msg)
```
## swoole_clear_error()

Clear the error on the socket or the error code on the last error.

!> Available in Swoole version >= `v4.6.0`

```php
function swoole_clear_error()
```
## swoole_coroutine_socketpair()

Coroutine version of [socket_create_pair](https://www.php.net/manual/en/function.socket-create-pair.php).

!> Swoole version >= `v4.6.0` is required

```php
function swoole_coroutine_socketpair(int $domain , int $type , int $protocol): array|bool
```
## swoole_async_set

This function can set options related to asynchronous I/O.

```php
function swoole_async_set(array $settings)
```

- enable_signalfd: enable or disable the use of `signalfd` feature
- enable_coroutine: switch built-in coroutine, [see details here](/server/setting?id=enable_coroutine)
- aio_core_worker_num: set the minimum number of AIO processes
- aio_worker_num: set the maximum number of AIO processes
## swoole_error_log_ex()

Write logs with specified level and error code.

```php
function swoole_error_log_ex(int $level, int $error, string $msg)
```

!> Available since Swoole version `v4.8.1`
## swoole_ignore_error()

Ignore error log of specified error code.

```php
function swoole_ignore_error(int $error)
```

!> Available since Swoole version >= `v4.8.1`