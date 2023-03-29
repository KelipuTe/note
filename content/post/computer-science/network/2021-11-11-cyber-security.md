---
draft: true
date: 2022-05-01 08:00:00 +0800
lastmod: 2022-05-01 08:00:00 +0800
title: "cyber security（网络安全）"
summary: ""
toc: true

categories:
- network(网络)

tags:
- computer-science(计算机科学)
- network(网络)
---

- cyber security（网络安全）
  - secrecy（保密性）
  - integrity（完整性）
  - availability（可用性）
- threat model（威胁模型）
  - attack vector（攻击向量）
- authentication（身份验证）
  - what you know
    - 用户名和密码
    - brute force attack（暴力攻击）
  - what you have
    - 锁和钥匙
  - what you are
    - biometric authenticator（生物识别）
    - probabilistic（概率性的）
  - two-factor（双因素）
  - multi-factor（多因素）
- access control（访问控制）
  - permissions（权限）
  - ACL（access control lists、访问控制表）
  - bell lapadula model（贝尔-拉帕杜拉模型）
- malware（恶意软件）
- independent verification and validation（独立安全检查和质量验证）
- lsolation（隔离） 
- sandbox（沙盒）
- cyber attack（网络攻击）
  - social engineering（社会工程学）
    - phishing（钓鱼）
    - pretexting（假托）
  - trojan horses（木马）
  - NAND mirroring（NAND 镜像）
  - exploit（漏洞利用）
    - buffer overflow（缓冲区溢出）
    - bounds checking（边界检查）
      - canaries（金丝雀）
    - code injection（代码注入）
  - zero day vulnerability（零日漏洞）
  - worms（计算机蠕虫）
  - botnet（僵尸网络）
    - DDOS（distributed denial of service attack、分布式拒绝服务攻击）

### 网络安全

- 保密性：有权限，才能读取数据
- 完整性：有权限，才能访问或修改系统和数据
- 可用性：有权限，应该可以随时访问系统和数据

### 贝尔-拉帕杜拉模型

没有向上读的权限，没有向下写的权限。

### NAND 镜像

如果能连上目标计算机，就可以通过刷新整个内存，来绕过密码尝试次数限制。
