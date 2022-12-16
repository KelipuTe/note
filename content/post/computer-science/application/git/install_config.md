---
draft: false
date: 2021-11-19 08:00:00 +0800
lastmod: 2021-11-19 08:00:00 +0800
title: "在 Windows 环境安装和配置 Git"
summary: "在 Windows 环境安装 Git；配置用户名和邮箱；配置 github、gitee、阿里云云效、腾讯 coding 的 SSH；"
toc: true

categories:
- application(应用)

tags:
- computer-science(计算机科学)
- Git
---

> CPU AMD64(x86_64)<br/>
> Windows 11 家庭版<br/>
> git 2.34.0

### 下载和安装

去 [Git 官方网站](https://git-scm.com) 下一个 Windows 环境的安装包。这里下载到的安装包是 Git-2.34.0-64-bit.exe。运行安装包，一路下一步即可。本体安装应该是自动配好的，可以输出版本检查一下有没有安装好。

```
> git version
git version 2.34.1.windows.1
```

### 全局配置用户名和邮箱

```
> git config --global user.name "xxx"
> git config --global user.email "xxx@xxx.com"
```

命令执行后会在 `C:\Users\用户名\` 目录下创建一个 `.gitconfig` 文件用于保存配置。

### 记住密码

永久记住密码：

```
> git config --global credential.helper store
```

临时记住密码：

```
> git config –global credential.helper cache
> git config –global credential.helper 'cache –timeout=3600'
```

这两条命令会在目录 `C:\Users\用户名\` 下创建一个 `.gitconfig`文件用于保存配置。第一次提交任然需要输入用户名和密码。提交成功后，同一个用户就不需要再输入了。提交成功后会在目录 `C:\Users\用户名\` 下创建一个 `.git-credentials` 文件用于保存密码。

### Github 配置 SSH

打开 [Github 的账号设置页面](https://github.com/settings/profile) ，在左侧边栏里找到 **Access->SSH and GPG keys**。最后要把生成的 SSH 的公钥配置在这里。

#### 第一步，在自己的终端上生成 SSH。

- SSH 目录一般都在 `~/.ssh`，对应到 win11 上就是 `C:\Users\用户名\.ssh` 目录。
- 进入 `~/` 目录，对应到 win11 上就是 `C:\Users\用户名\` 目录。
- 这里建议用 Git 提供的那个 Git Bash 命令行窗口操作。它会自动的把 `~` 目录对应到 win11 的用户目录。
- 然后使用命令生成 SSH，一般一路回车即可。

```
> ssh-keygen -t rsa -C "xxx@xxx.com" -f ~/.ssh/github_rsa
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/Administrator/.ssh/github_rsa
Your public key has been saved in /c/Users/Administrator/.ssh/github_rsa.pub
The key fingerprint is:
SHA256:xxx xxx@xxx.com
The key's randomart image is:
+----[RSA 256]----+
|                 |
+----[SHA256]-----+
```

- `-t rsa`，表示使用 rsa 算法。
- `-C "xxx@xxx.com"`，指定 SSH 的名字，不一定非要是邮箱，但是一般用邮箱。
- `-f ~/.ssh/github_rsa`，指定了生成出来的文件的目录和文件名。

如果不指定文件名，则会用默认的文件名，如果想要配置多个 SSH 就需要指定文件名。

#### 第 2 步，获取和配置公钥。

生成 SSH 的步骤完成后，用 `cat` 获取公钥。

```
> cat ~/.ssh/github_rsa.pub
ssh-rsa 一大串 xxx@xxx.com
```

然后把上面 cat 到的公钥内容贴到 Github 配置 SSH 公钥的地方。

#### 第 3 步，配置 SSH。

如果上面用的是默认的文件名，那其实是不需要配置的，直接默认配置就生效了。

但是如果指定文件名，就需要进行下面的配置。使用 `ssh-add` 添加私钥。配置多个 SSH 就要执行多次。

```
> ssh-add ~/.ssh/github_rsa
Identity added: /c/Users/Administrator/.ssh/github_rsa (xxx@xxx.com)
```

如果遇到报错：`Could not open a connection to your authentication agent.`。

先执行 `ssh-agent bash`，这命令没有输出，然后再执行 `ssh-add` 命令。

- `ssh-add -l`，查看私钥。
- `ssh-add -D`，删除全部私钥。

私钥添加完成后，在 `~/.ssh/` 目录创建文件名为 config 的配置文件。

在 config 文件添加以下配置内容，配置多个 SSH 就要写多个。

```
# Github
# Host，识别模式，和主机名一样就行
Host github.com
# HostName，主机名
# Port，端口号，如果不是默认的 22 就需要指定
HostName github.com
# IdentityFile，秘钥文件路径
IdentityFile ~/.ssh/github_rsa
# User，用户名
User xxx
```

#### 第 4 步，测试。

如果配置成功，则执行 `ssh -T` 命令后会，有下面提示成功的输出。

```
> ssh -T git@github.com
Hi xxx! You've successfully authenticated, but GitHub does not provide shell access.
```

### 配置多个 SSH

这里给 Github 和 Gitee 两个代码仓库配置 SSH。

第 1 步、第 2 步是一样的。都是生成 SSH 然后到对应的仓库去配置。

第 3 步、第 4 步不一样。`ssh-add` 需要执行多次，把每个 SSH 都添加。config 文件的配置也需要配多个。

如果有第 3 个、第 4 个、或者更多的仓库，以此类推即可。

#### Gitee 配置 SSH

Gitee 的帮助中心有关于 [配置 SSH 的文档](https://gitee.com/help/articles/4181)

打开 [Gitee 的账号设置页面](https://gitee.com/profile/account_information) ，在左侧边栏里找到 **安全设置->SSH 公钥**。最后要把生成的 SSH 的公钥配置在这里。

生成 SSH，这里除了用的是 ed25519 算法，其他的差不多。

```
> ssh-keygen -t ed25519 -C "xxx@xxx.com" -f ~/.ssh/gitee_rsa
Generating public/private ed25519 key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/kelipute/.ssh/gitee_rsa
Your public key has been saved in /c/Users/kelipute/.ssh/gitee_rsa.pub
The key fingerprint is:
SHA256:xxx xxx@xxx.com
The key's randomart image is:
+--[ED25519 256]--+
|                 |
+----[SHA256]-----+
```

生成 SSH 的步骤完成后，用 `cat` 获取公钥。

```
> cat .ssh/gitee_rsa.pub
ssh-ed25519 一大串 xxx@xxx.com
```

然后把上面 cat 到的公钥内容贴到 Gitee 代码仓库配置 SSH 的地方。

使用 `ssh-add` 添加私钥。

```
> ssh-add ~/.ssh/gitee_rsa
Identity added: /c/Users/Administrator/.ssh/gitee_rsa (xxx@xxx.com)
```

在 config 文件添加以下配置内容。

```
# Gitee
Host gitee.com
HostName gitee.com
IdentityFile ~/.ssh/gitee_rsa
User xxx
```

测试。

```
> ssh -T git@gitee.com
The authenticity of host 'gitee.com (212.64.62.183)' can't be established.
ED25519 key fingerprint is SHA256:一大串
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'gitee.com' (ED25519) to the list of known hosts.
Hi xxx! You've successfully authenticated, but GITEE.COM does not provide shell access.
```

#### 阿里云云效配置 SSH

打开 [云效的个人设置页面](https://account-devops.aliyun.com/settings/profile) ，在左侧边栏里找到 **SSH 公钥**。最后要把生成的 SSH 的公钥配置在这里。

生成 SSH，阿里云云效要求使用 ed25519 算法，其他的差不多。

```
> ssh-keygen -t ed25519 -C "xxx@xxx.com" -f ~/.ssh/yun2xiao4_ed25519
Generating public/private ed25519 key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/Users/Administrator/.ssh/yun2xiao4_ed25519
Your public key has been saved in /c/Users/Administrator/.ssh/yun2xiao4_ed25519.pub
The key fingerprint is:
SHA256:xxx xxx@xxx.com
The key's randomart image is:
+--[ED25519 256]--+
|                 |
+----[SHA256]-----+
```

生成 SSH 的步骤完成后，用 `cat` 获取公钥。

```
> cat ~/.ssh/yun2xiao4_ed25519.pub
ssh-ed25519 一大串 xxx@xxx.com
```

然后把上面 cat 到的公钥内容贴到阿里云云效代码仓库配置 SSH 的地方。

使用 `ssh-add` 添加私钥。

```
> ssh-add ~/.ssh/yun2xiao4_ed25519
Identity added: /c/Users/Administrator/.ssh/yun2xiao4_ed25519 (xxx@xxx.com)
```

在 config 文件添加以下配置内容。

```
# Gitee
Host codeup.aliyun.com
HostName codeup.aliyun.com
IdentityFile ~/.ssh/yun2xiao4_ed25519
User xxx
```

#### 腾讯 Coding 配置 SSH

Coding 的帮助中心有关于 [配置 SSH 的文档](https://coding.net/help/docs/repo/ssh/config.html)

打开 [Coding 的个人账户页面](https://xxx.coding.net/user/account/setting/basic) ，在左侧边栏里找到 **SSH 公钥**。最后要把生成的 SSH 的公钥配置在这里。


