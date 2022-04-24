---
tags: #linux #tmux
---

# Linux 开机自启

建议用 `systemd` 来托管服务，而不是在 `rc.local` 启动。

不过因为 `rc.local` 比较简单，这里记录一下用法。

`rc.local` 在 `/etc/rc.local` ，是 `/etc/rc.d/rc.local` 的软链接。

要让系统可以执行 `rc.local` 还要先 `chmod +x /etc/rc.d/rc.local`。

该文件相当于系统开机时执行的.sh文件，里面可以直接写命令。需要注意的是，执行的上下文是 `root` ，而且**没有环境变量**，如果执行的命令需要环境变量，可以在脚本中设置，也可以用 `su` 切换用户来执行，例如：
```bash
su pi -c "xxx"
```

**不要让 `rc.local` 挂起**
`rc.local` 是一个脚本，是按顺序执行的，执行完一个程序后才会执行下一个程序，如果某程序不是后台程序，就应该加&让程序运行在后台，否则 `rc.local` 会挂起。

**挂起会导致系统启动被阻塞**
