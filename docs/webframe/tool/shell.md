---
title: 我常用的前端提效 shell 命令
date: 2020-07-06
tags:
    - shell
    - 工具
---

## curl
`curl` 是常用的命令行工具，用来请求 Web 服务器。它的名字就是客户端（client）的 `URL` 工具的意思。`curl` 功能非常强大，它的命令可以直接放到 `postman` 使用，`postman` 也是支持 `curl` 的请求方式。

### 常用的结果参数
`curl` 的参数包括很多，这里只说几个常用的，如果遇到复杂情况可以参考文档。

不知道是不是还有的小伙伴不知道 `postman` 直接支持 `curl` 命令,在 `postman` 中点击 `code` 就会出现对应请求的`curl`命令

![](http://img.xiaogangzai.cn/shell_20200718_1.jpg)
- `-X` 参数指定 `HTTP` 请求的方法。
- `-H` 参数添加 `HTTP` 请求的标头。
- `-d` 参数用于发送 `POST` 请求的数据体。使用 `-d` 参数以后，`HTTP` 请求会自动加上标头`Content-Type` : `application/x-www-form-urlencoded`。并且会自动将请求转为 `POST` 方法，因此可以省略`-X POST`
- `-b` 参数用来向服务器发送 `Cookie`。

### curl 项目中应用
如果对 `curl` 熟悉的小伙伴完全可以替代 `postman` 等工具，小伙伴可以直接模拟请求。

因为在 `BFF` 项目中，好多时候前端也参与开发，我们也会直接调用后端的接口，有时候报错不知道是不是自己参数写错了，或者 `cookie` 有问题，找问题调试不方便，在 `local` 环境下，我们会直接打印出完整的 `curl` 请求，这时候可以直接看出错误，开发者只需要知道 `curl` 的一些参数就可以，还可以直接把 `curl` 命令复制到 `postman` 进行调试。看一下具体实现部分代码

``` javascript
 //只在本地环境输出
    if (ctx.app.config.env === 'local') {
      const str =
        curlString(url, {
          method,
          headers,
          body,
        }) + '\n';
      console.log('\x1b[32m%s\x1b[0m', str);
    }
    
/**
 * Builds a curl command and returns the string.
 * @param  {String} url               Endpoint
 * @param  {Object} options           Object with headers, etc. (fetch format)
 * @return {String}                   cURL command
 */
function curlString(url, options) {
  const method = options && options.method && typeof options.method === 'string' ? options.method.toUpperCase() : 'GET';

  const hasHeaders = options && options.headers && typeof options.headers === 'object';
  const hasBody = options && options.body;

  let curl = `\ncurl --request ${method} \\\n--url '${url}'`;

  if (hasHeaders) {
    curl +=
      ' \\\n' +
      Object.entries(options.headers)
        .filter(([key, value]) => value !== undefined)
        .map(([key, value]) => `--header '${key}: ${value}'`)
        .join(' \\\n');
  }

  if (hasBody) {
    curl += ` \\\n--data '${bodyToDataString(options)}'`;
  }

  return curl;
}

/**
 * Constructs a body string for use inside --data
 * @param  {Object} options           Object with headers, etc. (fetch format)
 * @return {String}                   cURL command data string
 */
function bodyToDataString(options) {
  let parsedData;
  try {
    parsedData = JSON.parse(options.body);
  } catch (e) {
    // fall back to original body if it could not be parsed as JSON
    parsedData = options.body;
  }

  // return an ampersand delimited string
  const headers = _.get(options, 'headers');
  const contentType = _.toLower(_.get(headers, 'content-type') || _.get(headers, 'Content-Type'));
  if (contentType === 'application/x-www-form-urlencoded') {
    if (typeof parsedData === 'string') {
      return parsedData;
    } else {
      return Object.entries(parsedData)
        .map(([key, val]) => `${key}=${val}`)
        .join('&');
    }
  } else {
    return JSON.stringify(parsedData);
  }
}
```


## vim 中的基本操作和配置
### 非 insert 模式
在 `vim` 打开文件后，还没有使用插入编辑，可以做哪些基本操作
1. `G` 快速移动到文件底部(常用于查看日志)
2. `gg` 快速移动到文件顶部
3. `0` 快速移动到行首
4. `$` 快速移动到行尾
4. `:13` 快速移动到特定行 
5. `ZZ` 光标移动到本屏中间
6. `dd` 剪切本行
7. `yy` 复制本行
8. `u` 撤销（undo缩写，撤销）
9. `p` 粘贴 （p指paste，粘贴）
10. 在 `mac` 系统下可以 `option+点击` 快速移动到想要的位置（也就是光标）

### insert 模式

前面说了多种移动方式，接下来结束几个常用的 insert 命令，我这里就结束一些常用简单的
1. i 在当前光标的前面进行编辑
2. o 快速进入 `insert` 模式，并定位到下一行编辑
3. esc 退出 `insert` 模式，与 <crtl-[>

## ping 
在网络中 `ping` 是一个十分强大的 `TCP/IP` 工具。

1. 用来检测网络的连通情况和分析网络速度

2. 根据域名得到服务器`IP`

3. 根据`ping`返回的`TTL`值来判断对方所使用的操作系统及数据包经过路由器数量。


`bytes`值：数据包大小，也就是字节。

`time`值：响应时间，这个时间越小，说明你连接这个地址速度越快。

`TTL`值：`Time To Live`,表示`DNS`记录在`DNS`服务器上存在的时间，它是 IP 协议包的一个值，告诉路由器该数据包何时需要被丢弃。可以通过 `Ping` 返回的 `TTL` 值大小，粗略地判断目标系统类型是 `Windows` 系列还是 `UNIX/Linux` 系列。

默认情况下，`Linux` 系统的` TTL `值为64或255，`WindowsNT/2000/XP` 系统的 `TTL` 值为 128，`Windows98` 系统的 `TTL` 值为32，UNIX 主机的 TTL 值为 255。

除了直接 `ping ip` ，还可以 `ping` 域名，会自动把域名解析为 `ip`。

### 应用
最常用的方式是直接ping ip地址，测试网络连通性

### 学会看懂出错提示信息
（1）No 
`Answer`：这种故障表明本机有一条通向中心主机的路由，但没有收到发给该中心主机的任何信息。原因可能是：中心主机没有工作、本机或中心主机网络配置不正确、本地或中心的路由器没有工作、通信线路有故障、中心主机存在路由选择问题，等等。 

（2）`Request Timed 
Out`：超时错误，被测试的机器不能正常连接，原因可能是该主机此时未连接（如已关机）、或到路由器的连接有问题、或路由器不能通过，或对方主机使用了防火墙软件禁止进行 `Ping` 测试等等。 

（3）`Unknown Host Name`：无法解析主机名字，可能是`DNS`设置不对，或者对方主机不存在

## telnet
`telnet` 经常可以确定远程服务的状态，比如确定远程服务器的某个端口是否能访问（端口连通性）。

`telenet`是`windows`标准服务，可以直接用;如果是`linux`或者`mac`，需要自己安装`telnet`
### 使用 telnet ip port

1）先用`telnet`连接不存在的端口

```shell
[root@localhost ~]# telnet 10.0.250.3 80
Trying 10.0.250.3...
telnet: connect to address 10.0.250.3: Connection refused #直接提示连接被拒绝
```

2）再连接存在的端口

```shell
[root@localhost ~]# telnet localhost 22
Trying ::1...
Connected to localhost. #看到Connected就连接成功了
Escape character is '^]'.
SSH-2.0-OpenSSH_5.3
a
Protocol mismatch.
Connection closed by foreign host.
```
## 总结
优秀和常用的 `shell` 命令有好多，我这里只写了几个非常常用，并且前端开发者也会经常用到的命令，希望对小伙伴们有一丢丢帮助。