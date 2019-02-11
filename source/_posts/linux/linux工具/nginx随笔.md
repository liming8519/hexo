---
title: nginx随笔
tags:
  - tool
categories:
  - linux
date: 2019-02-11 01:00:00
---
> nginx笔记
<!-- more -->

## sendfile
- 读取文件并socket传输的普通过程
0. read(file,tmp_buf, len);
   write(socket,tmp_buf, len);
1. 系统调用 read()产生一个上下文切换，从 user mode 切换到 kernel mode，然后 DMA 执行拷贝，把文件数据从硬盘读到一个 kernel buffer 里。
2. 数据从 kernel buffer拷贝到 user buffer，然后系统调用 read() 返回，这时又产生一个上下文切换，从kernel mode 切换到 user mode。
3. 系统调用write()产生一个上下文切换，从 user mode切换到 kernel mode，然后把步骤2读到 user buffer的数据拷贝到 kernel buffer，不过这次是个不同的 kernel buffer，这个 buffer和 socket相关联。
4. 系统调用 write()返回，产生一个上下文切换，从 kernel mode 切换到 user mode ，然后 DMA 从 kernel buffer拷贝数据到协议栈。
- sendfile的协议栈传输过程
0. sendfile(socket,file, len);
1. 系统调用sendfile()通过 DMA把硬盘数据拷贝到 kernel buffer，然后数据被 kernel直接拷贝到另外一个与 socket相关的 kernel buffer。这里没有 user mode和 kernel mode之间的切换，在 kernel中直接完成了从一个 buffer到另一个 buffer的拷贝。
2. DMA 把数据从 kernelbuffer 直接拷贝给协议栈，没有切换，也不需要数据从 user mode 拷贝到 kernel mode，因为数据就在 kernel 里。
- 适用场景
sendfile 是将 in_fd 的内容发送到 out_fd 。而 in_fd 不能是 socket ， 也就是只能文件句柄。 所以当 Nginx 是一个静态文件服务器的时候，开启 SENDFILE 配置项能大大提高 Nginx 的性能。 但是当 Nginx 是作为一个反向代理来使用的时候，SENDFILE 则没什么用了，因为 Nginx 是反向代理的时候。 in_fd 就不是文件句柄而是 socket，此时就不符合 sendfile 函数的参数要求了。

## accept_mutex
1. 当一个新连接到达时，如果激活了accept_mutex，那么多个Worker将以串行方式来处理，其中有一个Worker会被唤醒，其他的Worker继续保持休眠状态；如果没有激活accept_mutex，那么所有的Worker都会被唤醒，不过只有一个Worker能获取新连接，其它的Worker会重新进入休眠状态，这就是「惊群问题」。
2. OS may wake all processes waiting on accept() and select(), this is called thundering herd problem. This is a problem if you have a lot of workers as in Apache (hundreds and more), but this insensible if you have just several workers as nginx usually has. Therefore turning accept_mutex off is as scheduling incoming connection by OS via select/kqueue/epoll/etc (but not accept()).

## multi_accept
1. 在Nginx获得有新连接的通知之后,接受尽可能多的连接。

