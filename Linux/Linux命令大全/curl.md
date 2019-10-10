# curl

# 简介

curl 是一个命令行工具，用来请求 Web 服务器，它的全称是 Client URL。

# 命令格式

```shell
$ curl [param] url
```

> 不带任何参数时，curl 发送的是 GET 请求

## 参数

- -A

  指定客户端的用户代理标头，即 `User-Agent`，默认值为 **curl/[version]**

  ```shell
  $ curl -A 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKei/537.36 (KHTML, like Gecko) Chrome/76.0.3809.100 Safari/537.36' https://google.com
  ```

  > 将 User-Agent 改为 Chrome 浏览器

  ```shell
  $ curl -A '' https://google.com
  ```

  > 移除 User-Agent

  也可以通过 `-H` 参数来 指定 User-Agent:

  ```shell
  $ curl -H 'User-agent: php/1.0' https://google.com
  ```

- -b

  向服务器发送 Cookie

  ```shell
  $ curl -b 'foo=bar' https://google.com
  ```

  > 这个命令会生成一个标头 **Cookie: foo=bar**，向服务器发送一个名为 **foo**，值为 **bar** 的 Cookie

  ```shell
  $ curl -b 'foo1=bar' -b 'foo2=baz' https://google.com
  ```

  > 发送两个 Cookie

  ```shell
  $ curl -b cookies.txt https://www.google.com
  ```

  > 读取本地文件，文件中是服务器设置的 Cookie，可参考 `-c` 参数

- -c

  将服务器设置的 Cookie 写入一个文件

  ```shell
  $ curl -c cookies.txt https://www.google.com
  ```

  > 将服务器的 Http 回应所设置的 Cookie 写入文本文件 cookies.txt

- -d

  用于发送 POST 请求的数据体

  ```shell
  $ curl -d 'login=emma&password=123' -X POST https://google.com/login
  # 或者
  $ curl -d 'login=emma' -d 'password=123' -X POST https://google.com/login
  ```

  > 使用 `-d` 参数后，Http 请求会自动加上标头 **Content-Type: application/x-www-form-urlencoded** , 并会自动将请求转为 POST 方法，因此可以省略 **-X POST**

  `-d` 参数可以读取本地文件的数据，并发送到服务器

  ```shell
  $ curl -d 'data.txt' http://google.com/login
  ```

  > 读取 **data.txt** 文件的内容，作为数据体向服务器发送

- --data-urlencode

  等同于 `-d` 参数，区别在于会自动将发送的数据进行 URL 编码

  ```shell
  $ curl --data-urlencode 'comment=hello world' https://google.com/login
  ```

  > 发送的数据 **hello world** 中有一个空格，需要进行 URL 编码

- -e

  用来设置 Http 的标头 **Referer**，表示请求的来源

  ```shell
  $ curl -e 'https://google.com?q=example' https://www.example.com
  ```

  > 将 Referer 设为 https://google.com?q=example

  `-H` 参数可以用来设置标头 `Referer`

  ```shell
  $ curl -H 'Referer: https://google.com?q=example' https://www.example.com
  ```

- -F

  向服务器上传二进制文件

  ```shell
  $ curl -F 'file=photo.png' https://google.com/profile
  ```

  > 给 Http 请求头加上标头 **Content-Type: multipart/form-data**，然后将文件 photo.png 作为 file 字段上传

  `-F` 参数可以指定 MIME 类型

  ```shell
  $ curl -F 'file=photo.png:type=image/png' https://google.com/profile
  ```

  > 指定 MIME 类型为 **image/png**，否则会把 MIME 类型设置为 **application/octet-stream**

  `-F` 参数可以指定文件名

  ```shell
  $ curl -F 'file=@photo.png:filename=me.png' https://google.com/profile
  ```

  > 原始问价名为 photo.png，但是服务器接收的文件名为 me.png

- -G

  用来构造 GET 请求的传参

  ```shell
  $ curl -G -d 'q=kitties' -d 'count=20' https://google.com/search
  ```

  > 这个命令会发出一个 GET 请求，实际请求的 URL 为 **https://google.com/search?q=kitties%count=20**。如果省略 `-G`，会发出一个 POST 请求

- 如果参数需要进行 URL 编码，可以结合 `--data-urlencode` 参数

  ```shell
  $ curl -G --data-urlencode 'comment=hello world' https://www.example.com
  ```

- -H

  添加 Http 请求的标头

  ```shell
  $ curl -H 'Accept-Language: en-US' https://google.com
  ```

  > 添加 **Accept-Language: en-US** 标头

  ```shell
  $ curl -H 'Accept-Language: en-US' -H 'Secret-Message: xyzzy' https://google.com
  ```

  > 添加两个标头

  ```shell
  $ curl -d '{"login": "emma", "pass": "123"}' -H 'Content-Type: application/json' https://google.com/login
  ```

  > 添加请求标头 **Content-Type: application/json**，然后通过 `-d` 参数发送 JSON 数据

- -i

  打印服务器回应的 Http 标头

  ```shell
  $ curl -i https://www.example.com
  ```

  > 收到服务器回应后，先输出服务器回应的标头，然后空一行，再输出网页的源码

- -I

  向服务器发送 HEAD 请求，并将服务器返回的 Http 标头打印出来

  ```shell
  $ curl -I https://www.example.com
  ```

  > 这个命令会输出服务器对 HEAD 请求的回应

  `--head` 参数等同于 `-I`

  ```shell
  $ curl --head https://www.example.com
  ```

- -k

  跳过 SSL 检测

  ```shell
  $ curl -k https://www.example.com
  ```

  > 这个命令不会检查服务器 SSL 证书是否正确

- -L

  让 Http 请求跟随服务器的重定向，curl 默认不会跟随重定向

  ```shell
  $ curl -L -d 'tweet=hi' https://api.twitter.com/tweet
  ```

- --limit-rate

  限制 Http 请求和回应的贷款，模拟慢网速的环境

  ```shell
  $ curl --limit-rate 200k https://google.com
  ```

  > 将贷款限制在 200k

- -o

  将服务器的回应保存成文件，等同于 `wget` 命令

  ```shell
  $ curl -o example.html https://www.example.com
  ```

  > 将 **www.example.com** 保存成 **example.html** 文件

- -O

  将服务器回应保存成文件，并将 URL 的最后部分当做文件名

  ```shell
  $ curl -O https://www.example.com/foo/bar.html
  ```

  > 将服务器回应保存成文件，文件名为 **bar.html**

- -s

  不输出错误和进度信息

  ```shell
  $ curl -s https://www.example.com
  ```

  想让 curl 不产生任何输出，可以使用下面的命令：

  ```shell
  $ curl -s -o /dev/null https://google.com
  ```

- -S

  只输出错误信息，通常与 `-o` 一起使用

  ```shell
  $ curl -S -o /dev/null https://google.com
  ```

  > 这个命令不会有任何输出，除非发生错误

- -u

  设置服务器认证的用户名和密码

  ```shell
  $ curl -u 'bob:12345' https://google.com/login
  ```

  > 这个命令设置用户名为 **bob**，密码为 **12345**，然后将其转为 Http 标头 **Authorization: Basic Ym9i0jEyMzQ1**

  curl 可以识别 URL 里面的用户名和密码

  ```shell
  $ curl https://bob:12345@google.com/login
  ```

  > 这个命令可以识别 URL 里面的用户名和密码，将其转为上个例子里面的 Http 标头

  ```shell
  $ curl -u 'bob' https://google.com/login
  ```

  > 这个命令设置了用户名，执行后，curl 会提示用户输入密码

- -v

  输出通信的整个过程，用于调试

  ```shell
  $ curl -v https://www.example.com
  ```

  `--trace` 参数可以用于调试，还会输出原始的二进制数据

  ```shell
  $ curl --trace https://www.example.com
  ```

- -x

  指定 Http 请求的代理

  ```shell
  $ curl -x socks5://james:cats@myproxy.com:8080 https://www.example.com
  ```

  > 这个命令通过 myproxy.com:8080 的 socks5 代理faculty

  如果没有指定代理协议，默认 Http

  ```shell
  $ curl -x james:cats@myproxy.com:8080 https://www.example.com
  ```

  > 这个代理使用 Http 协议

- -X

  指定 Http 请求的方法

  ```shell
  $ curl -X POST https://www.example.com
  ```

  > 对 https://www.example.com 发出 POST 请求



