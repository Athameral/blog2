---
title: sshd_nonroot
date: 2024-09-07 16:36:24
tags: [SSH]
---

# 以非特权用户运行sshd

## 为什么不能直接运行

- UsePAM
- 读写权限
- 特权端口（Privileged Ports）
- 密码登录
- PID File