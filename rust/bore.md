---
tags: #rust #network #bore
---

# [bore](https://github.com/ekzhang/bore)
`Rust` 中的一个现代、简单的 `TCP` 隧道，它将本地端口暴露给远程服务器，绕过标准 `NAT` 。这就是它所做的一切：不多也不少。

## 安装
> 需要有rust环境，rust环境安装参照 [[install-rust]] 。
```bash
cargo install bore-cli
```
*安装的时候会从 `github` 拉 `crates.io-index` ，要上 `github` 啊，可恶，根本上不去。*


## 服务端使用
先在云服务器把防火墙打开想要开放的端口，以及开放 `bore` 的对外端口 `7835`。

客户端是通过连接 `7835` 来进行握手，然后才从其它端口建立连接。

然后直接运行 `bore server` 就可以开始监听了。
```
$ bore server
2022-04-21T07:08:49.256539Z  INFO bore_cli::server: server listening addr=0.0.0.0:7835
```


## 客户端使用
确认要映射的端口，服务端地址
```
$ bore local 22 --to hostname
```
直接这样连的话，端口是随机分配的，要指定端口需要使用 `--port <port>` 命令行参数。
```
$ bore local 22 --to jie65535.top --port 9022
2022-04-21T09:11:39.835236Z  INFO bore_cli::client: connected to server remote_port=9022                              
2022-04-21T09:11:39.835322Z  INFO bore_cli::client: listening at jie65535.top:9022
```

## systemd 服务
用 `cargo install bore-cli` 安装后执行 `sudo ln -s ~/.cargo/bin/bore /usr/bin` 创建软连接。

然后在 `/etc/systemd/system` 目录下新建 `bore.service`，添加以下内容
```
[Unit]
Description=bore service
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=bore server
Restart=always
RestartSec=60s
User=root

[Install]
WantedBy=default.target
```
接着执行 `sudo systemctl enable bore.service` 即可开机自启。
用 `sudo systemctl start bore.service` 来启动服务。
用 `sudo systemctl status bore.service` 查看状态。
这些都是 `systemd` 的基操了，不多赘述。
成功的话状态会输出以下信息
```
$ sudo systemctl status bore.service
● bore.service - bore service
   Loaded: loaded (/etc/systemd/system/bore.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2022-04-23 16:20:01 CST; 2min 55s ago
 Main PID: 241914 (bore)
    Tasks: 3 (limit: 11738)
   Memory: 288.0K
   CGroup: /system.slice/bore.service
           └─241914 /usr/bin/bore server

Apr 23 16:20:01 iZwz9ejck3djoyghr9h865Z systemd[1]: Started bore service.
Apr 23 16:20:01 iZwz9ejck3djoyghr9h865Z bore[241914]: 2022-04-23T08:20:01.902721Z  INFO bore_cli::server: server listening addr=0.0.0.0:7835
```

## 服务端帮助
```
bore-server 0.3.0
Runs the remote proxy server

USAGE:
    bore server [OPTIONS]

OPTIONS:
    -h, --help                   Print help information
        --min-port <MIN_PORT>    Minimum TCP port number to accept [default: 1024]
    -s, --secret <SECRET>        Optional secret for authentication [env: BORE_SECRET]
    -V, --version                Print version information
```

## 客户端帮助
```
bore-local 0.3.0
Starts a local proxy to the remote server

USAGE:
    bore local [OPTIONS] --to <TO> <LOCAL_PORT>

ARGS:
    <LOCAL_PORT>    The local port to expose

OPTIONS:
    -h, --help                 Print help information
    -l, --local-host <HOST>    The local host to expose [default: localhost]
    -p, --port <PORT>          Optional port on the remote server to select [default: 0]
    -s, --secret <SECRET>      Optional secret for authentication [env: BORE_SECRET]
    -t, --to <TO>              Address of the remote server to expose local ports to
    -V, --version              Print version information
```