---
title: 'chrome os 更新流程'
date: 2021-10-14T09:48:26+08:00
tags: ['chromium']
categories: ['developer']
draft: false
---

# chromeos 更新流程梳理

chrome os 成功更新流程中 `updat_engine_client` 以下简称(client)与 omaha server (以下简称 server)存在四次应答，
下面就这四次应答进行详述。整个流程中 request 和 response 都是使用 xml 格式。

requestid 每次或许不同，但 sessionId 是相同的
找出四次的 InstallPlan 的区别
以下流程将插槽 A 当作当前分区,硬盘为机械硬盘(`/dev/sda`)

## 第一次应答

1. client 端： 由`updat_engine_client` 发起更新检查请求，请求中包含了 updatcheck 的 tag 表明需要 server 端是否有新的更新包发布。
2. server 端： 检查启动参数中更新负载存放位置中的更新包 json 信息，确定 appid 以及更新方式(delta 更新还是 full 更新),appid 验证通过之后回复 client status ok 表示可以更新。
3. client 端: 收到可以更新的信息,确定下载方式(p2p/httpserver),更新地址，更新方式(delta/full), 生成安装计划(InstallPlan)

## 第二次应答

1. client 端：第二次向 server 开始下载更新请求， eventtype 为 13(`UPDATE_DOWNLOAD_STARTED`)
2. server 端： 确认请求，发送 status ok
3. client 端： 得到 ok 结果生成开始下载流程
   - 生成安装计划 InstallPlan
   - 标记 A 分区为活动分区(当前分区)
   - 运行系统的 `/usr/sbin/chromeos-setgoodkernel` (作用?)
   - 标记另一个分区为 unbootloader,并且删除之前更新的完成标记和属性
   - 开始第一次传输，检查是否使用 proxy
   - 当可以收到数据时重置 url 快速失败计数为 0 (url failure count)
   - 验证 public key
   - 准备分区 hash
   - 对新 root 分区计算 sha256 加密
   - 对新 kernel 分区计算 sha256 加密
   - 打开 `/dev/sda5` 不带 `O_DSYNC`
   - 缓存写入
   - 将 1000 个操作应用在 root 分区
   - 打开 `/dev/sda4` 不带 `O_DSYNC`
   - 缓存写入
   - 将 32 个操作应用在 root 分区 (数字的计算？)
   - 完成所有操作 operations
   - 通过 libcurl 发送传输终止请求
   - 部署下载完成
   - 使 Full Payload Attempter Number 增加 1
   - 收集更新直方图
   - 完成 chromeos download action

## 第三次应答

1. client 端：第三次向 server 发送更新下载完成请求，eventtype 为 14 (`UPDATE_DOWNLOAD_FINISHED`)
2. server 端：确认请求，发送 status ok
3. client 端：开始文件系统验证， 生成 InstallPlan。包含了目标分区和 hash
   - 计算 root 的 hash
   - 计算 kernel 的 hash
   - 完成文件系统验证，开始更新后行为
   - 运行挂载了的`/dev/sda5`中的 postinst (postinst at /tmp/.org.chromium.Chromium.N38Z8V/postinst))
   - 将新的 postinst 格式化为 data
   - 写入新的 lsb-release 文件
   - 设置 boot 目标 `/dev/sda5` 分区 5 插槽 B
   - 更新同步分区操作
   - 将插槽 B 设置为活动分区
   - 安装后操作完成

## 第四次应答

1. client 端：第三次向 server 发送更新完成请求，eventtype 为 3 (`UPDATE_COMPLETE`)
2. server 端：确认请求，发送 status ok
3. client 端：设置 cgroup cpu shares 为 1024
   - 重置记录的 hash 为已经准备好了的分区
   - 更新 timestamp
   - 更新成功应用，等待重启
