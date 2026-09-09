---
title: "git传输时使用的两种协议ssh和http的区别"
source: "https://blog.csdn.net/Castlehe/article/details/119530573"
author:
  - "[[吨吨不打野]]"
published: 2021-08-12
created: 2026-09-01
description: "文章浏览阅读2.7w次，点赞41次，收藏148次。本文详细解析了Git协议的演变，比较了SSH与HTTP在Git中的应用，重点讲解了如何通过SSH秘钥进行安全连接，并解决GitHub SSH警告和密钥更换问题。"
tags:
  - "clippings"
---
## 1\. 简单介绍

在我一开始使用git来管理github上的项目时，所有可以搜到的入门教程，基本开始都会让你设置下面这样

```bash
$ git config --global user.name "your name"
$ git config --global user.email "your_email@youremail.com"
bash
```

很多都只有设置全局用户名和密码，而没有提及ssh key的方式。

实际上，不管时设置用户名和密码，还是生成ssh key，其实都是Git的文件传输协议。

---

## 2\. Git、 SVN 和Github、Gitee、Gitlab区别

经常可以看到和git相关的几个词语，包括：github、gitlab以及gitee，简要介绍一下区别：

- Git是一种版本控制系统，是一种工具，用于代码的存储和版本控制。
- GitHub是一个基于Git实现的在线代码仓库，是目前全球最大的代码托管平台，可以帮助程序员之间互相交流和学习。
- GitLab是一个基于Git实现的在线代码仓库软件，是由 `GitLabInc.`开发，可以用GitLab自己搭建一个类似于GitHub一样的仓库，但GitLab有完善的管理界面和权限控制，一般用于在企业、学校等内部网络搭建Git私服。
- Gitee.com(码云) 是 `OSCHINA.NET` 推出的代码托管平台,支持 Git 和 SVN,提供免费的私有仓库托管。
- GitHub和GiLlab两个都是基于Web的Git远程仓库，它们都提供了分享开源项目的平台，为开发团队提供了存储、分享、发布和合作开发项目的中心化云存储的场所。
- GitHub如果使用私有仓库，是需要付费的，GitLab可以在上面搭建私人的免费仓库
- GitLab让开发团队对他们的代码仓库拥有更多的控制，相对于GitHub，它有不少的特色：
	- 允许免费设置仓库权限
		- 允许用户选择分享一个project的部分代码
		- 允许用户设置project的获取权限，进一步提升安全性
		- 可以设置获取到团队整体的改进进度
		- 通过innersourcing让不在权限范围内的人访问不到该资源

另外，git和svn都属于开源版本控制系统 (version control system，VCS)

- SVN是Subversion的简称，是一个开放源代码的集中式版本控制系统,支持大多数常见的操作系统。比Git早些出来，目前来说，大多是开发人员都是比较熟悉这款工具的。TortoiseSVN这款辅助软件相信很多人都用过。
- Git：是一个开源的分布式版本控制系统，可以有效、高速的处理从很小到非常大的项目版本管理。Git是为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。是一款比较进阶的代码控制器。

SVN的特点：

- SVN的部署比较方便，拥有一个服务器主仓库，开发人员都相当于是客户端，设计理念上比较简洁，容易让人理解和接受，适合中小型代码项目。
- 使用的时候，首先开发人员都需要从预先的url仓库获取代码到本地，然后大家集中向url仓库提交代码。最终的版本就是这个仓库。
- 提交代码的时候先建议update，否则可能多个开发人员对同一个文件提交引起代码冲突。
- 如果中央服务器宕机，开发人员将不能及时进行代码提交，如果中央服务器瘫痪可能导致数据和日志丢失。

Git的特点：

- Git设计的原理比SVN要复杂一些，开发人员学习和理解的成本略高，它不是一个集中式的管理方式，引入了分布、分支等多个概念。
- 分布式也就是说当从服务器检索好代码到本地的时候，本地会自动创建一个仓库，本地的仓库和服务器的仓库的地位是一样的。
- 提交代码的时候一般是先提交(commit)到本地仓库，然后在push到服务器仓库进行合并（merge）。
- 当没有网络的环境下，依然可以方便本地仓库的提交代码和查看代码的历史日志。服务器就算宕机和故障，代码恢复也是轻而易举。
- 可以在本地灵活创建分支，然后提交到服务器上，多个分支来进行版本分化。

建议：

- 其实无论是哪一种版本控制器，都能够满足基本的协同需求
- 首先是看团队成员对版本的熟悉程度来选择。
- 其次建议代码量过大的程序还是使用Git安全性比较高，同时一般情况下，两种版本控制器都会用
- SVN一般用来管理项目，放设计图，各种文档和资料，这种东西没有必要弄得那么麻烦，而且本来就是需要大家共享且改动不大。其次Git专门用来管理代码。

---

参考：

- [Git，GitHub与GitLab有什么区别？](https://zhuanlan.zhihu.com/p/124085062)
- [github和gitlab的区别是什么](https://www.php.cn/tool/git/471279.html)
- [Git、GitHub、GitLab三者之间的联系以及区别](https://www.cnblogs.com/leeyongbard/p/9777498.html)
- [SVN和Git的区别](https://www.cnblogs.com/ai10999/p/11449426.html)
- [【VCS】-常见的版本控制系统（VCS）](https://blog.csdn.net/zt15732625878/article/details/54237391)

---

## 3\. Git协议

### 3.0为什么需要有git传输协议？

为了可以实现协作功能，需要一个远程的Git仓库，

- 虽然也可以从个人仓库上进行推送(push)和拉取(pull)来修改内容，但是这样容易混淆他人的进度，不建议这样。
- 故与他人合作的最佳方法即是建立一个你与合作者们都有权利访问，且可从那里推送和拉取资料的公用仓库。
- 与远程的git公用仓库通信，需要遵循一定的协议
- Git 可以使用四种不同的协议来传输资料：本地协议（Local），HTTP 协议，SSH（Secure Shell）协议及 Git 协议。

下面简要介绍本地协议和Git协议，着重介绍常用的http协议和ssh协议。

---

### 3.1 http协议

官方文档介绍有两种，

- Git 1.6.6 版本之前的一般被称为 哑 HTTP 协议，
- Git 1.6.6 版本引入了一种新的、更智能的协议，让 Git 可以像通过 SSH 那样智能的协商和传输数据， 新版本的 HTTP 协议一般被称为 智能 HTTP 协议，也是目前最流行的一种协议。

智能 HTTP 的运行方式和 SSH 及 Git 协议类似，只是运行在标准的 HTTP/S 端口上并且可以使用各种 HTTP 验证机制， 这意味着使用起来会比 SSH 协议简单的多，可以使用 HTTP 协议的用户名/密码授权，免去设置 SSH 公钥。

- 其像 SSH 协议一样提供传输时的授权和加密， 而且只用一个 URL 就可以都做到，省去了为不同的需求设置不同的 URL。
- 如果你要推送到一个需要授权的服务器上（一般来讲都需要），服务器会提示你输入用户名和密码。 从服务器获取数据时也一样。
- **优点** ：
	- 不同的访问方式只需要一个 URL 以及服务器只在需要授权时提示输入授权信息，这两个简便性让终端用户使用 Git 变得非常简单。
		- 相比 SSH 协议，可以使用用户名／密码授权是一个很大的优势，这样用户就不必须在使用 Git 之前先在本地生成 SSH 密钥对再把公钥上传到服务器。 对非资深的使用者，或者系统上缺少 SSH 相关程序的使用者，HTTP 协议的可用性是主要的优势。 与 SSH 协议类似，HTTP 协议也非常快和高效。
		- 另一个好处是 HTTPS 协议被广泛使用，一般的企业防火墙都会允许这些端口的数据通过。
- **缺点** ：
	- 在一些服务器上，架设 HTTPS 协议的服务端会比 SSH 协议的棘手一些。 除了这一点，用其他协议提供 Git 服务与智能 HTTP 协议相比就几乎没有优势了。
		- 在 HTTP 上使用需授权的推送，管理凭证会比使用 SSH 密钥认证麻烦一些

---

### 3.2 ssh协议

架设 Git 服务器时常用 SSH 协议作为传输协议。 因为大多数环境下服务器已经支持通过 SSH 访问 —— 即使没有也很容易架设。 SSH 协议也是一个验证授权的网络协议；并且，因为其普遍性，架设和使用都很容易。

通过 SSH 协议 克隆 版本库，你可以指定一个 `ssh:// 的 URL` ：

```bash
$ git clone ssh://[user@]server/project.git
# 或者使用一个简短的 scp 式的写法：
$ git clone [user@]server:project.git
bash123
```
- **优点**
	- 首先，SSH 架设相对简单 —— SSH 守护进程很常见，多数管理员都有使用经验，并且多数操作系统都包含了它及相关的管理工具。
		- 其次，通过 SSH 访问是安全的 —— 所有传输数据都要经过授权和加密。
		- 最后，与 HTTPS 协议、Git 协议及本地协议一样，SSH 协议很高效，在传输前也会尽量压缩数据。
- **缺点**
	- 缺点在于它不支持匿名访问 Git 仓库。
		- 如果你使用 SSH，那么即便只是读取数据，使用者也 必须 通过 SSH 访问你的主机， 这使得 SSH 协议不利于开源的项目，毕竟人们可能只想把你的仓库克隆下来查看。
		- 如果你只在公司网络使用，SSH 协议可能是你唯一要用到的协议。 如果你要同时提供匿名只读访问和 SSH 协议，那么你除了为自己推送架设 SSH 服务以外， 还得架设一个可以让其他人访问的服务。

### 3.3 本地协议

最基本的就是 `本地协议（Local protocol）` ，其中的 `远程版本库` 就是同一主机上的另一个目录，或者是使用共享文件系统进行的文件管理和访问（一般在同一局域网络中）

> 共享文件系统，常见的有NFS和Samba，
> 
> - NFS主要用于Linux/Unix平台下，
> - 而Samba用于将linux/Unix平台下的文件映射到Window系统网络邻居上，用于实现Linux/Unix到Window平台的共享，当然，它也可以实现Linux/Unix平台之间的文件共享

使用共享文件系统，就可以从本地版本库克隆（clone）、推送（push）以及拉取（pull）。 克隆一个版本库或者增加一个远程到现有的项目中，使用版本库路径作为 URL。例如：

```bash
$ git clone /srv/git/project.git
$ git clone file:///srv/git/project.git
bash
```

只是资源表示url的方式和常见的http形式有所不同。

- 优点：
	- 简单，直接使用现有的文件权限和网络访问权限。
- 缺点：
	- 共享文件系统比较难配置，并且比起基本的网络连接访问，不方便从多个位置访问（只有公司可以访问，家里不行，除非开vpn）
		- 使用类似于共享挂载的文件系统时，这个方法不一定是最快的。 访问本地版本库的速度与你访问数据的速度是一样的。 在同一个服务器上，如果允许 Git 访问本地硬盘，一般的通过 NFS 访问版本库要比通过 SSH 访问慢。
		- 这个协议并不保护仓库避免意外的损坏。 每一个用户都有“远程”目录的完整 shell 权限，没有方法可以阻止他们修改或删除 Git 内部文件和损坏仓库。

---

### 3.4 git协议

**Git 协议** ：是包含在 Git 里的一个特殊的守护进程；它监听在一个特定的端口（9418），类似于 SSH 服务，但是访问无需任何授权。

- 要让版本库支持 Git 协议，需要先创建一个 git-daemon-export-ok 文件 —— 它是 Git 协议守护进程为这个版本库提供服务的必要条件 —— 但是除此之外没有任何安全措施。 要么谁都可以克隆这个版本库，要么谁也不能。
- 这意味着，通常不能通过 Git 协议推送。 由于没有授权机制，一旦你开放推送操作，意味着网络上知道这个项目 URL 的人都可以向项目推送数据。 不用说，极少会有人这么做。

**优点** ：Git 协议是 Git 使用的网络传输协议里最快的。 如果你的项目有很大的访问量，或者你的项目很庞大并且不需要为写进行用户授权，架设 Git 守护进程来提供服务是不错的选择。 它使用与 SSH 相同的数据传输机制，但是省去了加密和授权的开销。

**缺点** ：Git 协议缺点是缺乏授权机制。

- 把 Git 协议作为访问项目版本库的唯一手段是不可取的。
- 一般的做法里，会同时提供 SSH 或者 HTTPS 协议的访问服务，只让少数几个开发者有推送（写）权限，其他人通过 git:// 访问只有读权限。
- Git 协议也许也是最难架设的。 它要求有自己的守护进程，这就要配置 xinetd、systemd 或者其他的程序，这些工作并不简单。 它还要求防火墙开放 9418 端口，但是企业防火墙一般不会开放这个非标准端口。 而大型的企业防火墙通常会封锁这个端口。

---

## 4\. http和ssh协议

`CLI` 命令行界面（ Command Line Interface for batch scripting）

为什么生成ssh秘钥就可以使用git，但是还要另外再配用户名和密码，使用http协议？

根据 [Github docs-在 Git 中设置用户名](https://docs.github.com/cn/github/getting-started-with-github/getting-started-with-git/setting-your-username-in-git) 可知：

> - Git 使用用户名将提交与身份关联。
> - Git 用户名与您的 GitHub 用户名不同。(是两种系统的用户名，但是可以起一样的，只要你愿意)
> - 可以使用 git config 命令更改与您的 Git 提交关联的名称。
> - 设置的新名称将在从命令行推送到 GitHub 的任何未来提交中显示。 如果您想要将真实姓名保密，则可以使用任意文本作为您的 Git 用户名。  
> 	使用 git config 更改与 Git 提交关联的名称仅影响未来的提交，不会更改用于过去提交的名称。

```bash
# 为计算机上的每个仓库设置 Git 用户名 使用--global参数
$ git config --global user.name "your name"
# 确认正确设置
$ git config --global user.name
> Mona Lisa

$ git config --global user.email "your_email@youremail.com"

# 为一个仓库设置 Git 用户名
$ git config user.name "Mona Lisa"
# 确认正确设置了用户名
$ git config user.name
> Mona Lisa
bash12345678910111213
```

使用git设置名字，是因为commit时候会使用，可以在gitlab/github每个版本的提交中看到某个版本的提交者是谁，那个就是git设置的名字，而github的名字就是用户名。类似：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/57ac4343d895c797cae4335248fc7a12.png)

在日常使用的时候，除了授权，输入用户名密码或者输入SSH key的时候有区别，其他时候感觉不到有什么区别。

---

参考：

- [官方文档-4.1 服务器上的 Git - 协议](https://git-scm.com/book/zh/v2/%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E7%9A%84-Git-%E5%8D%8F%E8%AE%AE)
- 博客园： [gitlab两种连接方式:ssh和http配置介绍](https://www.cnblogs.com/kevingrace/p/6114810.html)
- 简书： [github上git clone https与ssh的区别](https://www.jianshu.com/p/71fbf000b0e7)

（一般的博客都写的是使用区别，一个需要输入用户名密码，另一个不需要。。。官方文档里有偏向原理性的解释）

学习git最好的两个文档：

1. [官方gitbook](https://git-scm.com/book/zh/v2)
2. [github官方文档](https://docs.github.com/cn/github)

## 5\. ssh使用

之前一直是用户名密码登录，但是今天收到了这样的提示：  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d637092004cb543ef84b04afde482b27.png)  
没办法，就换一下吧。

参考： [github关于使用ssh连接的文档](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/about-ssh)

### 第一步，检查现有的ssh keys

在CLI中输入：

```bash
$ ls -al ~/.ssh
# Lists the files in your .ssh directory, if they exist
bash
```

如果是mac或者linux，可以直接找到这个文件  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/25ec15b403fde53533cc2b4c165abeb3.png)  
  
打开其中的 `known_hosts` ，这里就是我机器上存放key的地方，会有类似：id\_rsa.pub、id\_ecdsa.pub、id\_ed25519.pub这样的字眼。

### 第二步 新生成一个key

如果不想使用已有的key，可以新生成一组，比如：

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
ssh-keygen -t ed25519 -C "hs8023hfp@gmail.com"
bash
```

其中，ed25519是一种数字签名算法，如果系统 OpenSSH 版本低于 6.5(2014 年的古早版本),不支持这个算法，则可以使用下面代码来生成

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
bash1
```
- 生成后，会提示： `Generating public/private ed25519 key pair.`
- 然后还会提示： ` Enter a file in which to save the key (/Users/you/.ssh/id_ed25519): [Press enter]` ，问你这个key要存在哪里，直接按下Enter，就会保存在默认位置。
- 然后还会提示：
	- `Enter passphrase (empty for no passphrase): [Type a passphrase]`  
		`Enter same passphrase again: [Type passphrase again]`
		- 这个 passphrase表示 密语，是为了保护私钥的安全. 如果设置了passphrase, 当载入私钥时需要输入, 否则不需要输入.如果你认为你自己的私钥安全, 就可以不用输入了.
		- 所以直接两个回车就好了

### 第三步，将生成的key添加到ssh- agent 中

```bash
# 在后台启动 ssh-agent
$ eval "$(ssh-agent -s)"
> Agent pid 59566
# pid表示这个ssh-agent进程的进程ID

# 如果您使用的是 macOS Sierra 10.12.2 或更高版本，则需要修改您的~/.ssh/config文件以自动将密钥加载到 ssh-agent 中并将密码存储在您的钥匙串中。

#首先，检查~/.ssh/config文件是否存在于默认位置。
$ open ~/.ssh/config
> The file /Users/you/.ssh/config does not exist.

# 不存在，创建该文件。
$ touch ~/.ssh/config

# 打开~/.ssh/config文件，添加以下内容。如果SSH 密钥文件的名称或路径与示例代码不同，请修改文件名或路径以匹配您当前的设置。
// 关于这个文件的内容，如果你首次对某个机器进行配置，那么就按照下面写
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
// 如果是非首次配置（即已经有一台机器配过，现在要为别的机器配置，那么config的文件内容要改为）
Host *
  IgnoreUnknown UseKeychain // 加上这句
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519

# 将您的 SSH 私钥添加到 ssh-agent 并将您的密码存储在钥匙串中。如果您使用不同的名称创建了密钥，或者如果您要添加具有不同名称的现有密钥，请将命令中的id_ed25519替换为您的私钥文件的名称。
$ ssh-add -K ~/.ssh/id_ed25519
> Identity added: /Users/huangshan/.ssh/id_ed25519 (hs8023hfp@gmail.com)
bash123456789101112131415161718192021222324252627282930
```
- 参考：[.ssh/config: “Bad configuration option: UseKeychain” on Mac OS Sierra 10.12.6](https://stackoverflow.com/questions/47455300/ssh-config-bad-configuration-option-usekeychain-on-mac-os-sierra-10-12-6)

### 第四步，向github添加生成的ssh key

直接打开刚刚的 `id_ed25519.pub` ，复制全部内容  
或者使用以下代码：

```bash
$ pbcopy < ~/.ssh/id_ed25519.pub
# Copies the contents of the id_ed25519.pub file to your clipboard
bash
```

然后打开网页的github，找到setitngs ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/d5bb323115f30bd8935cc10cc38f4204.png)  
然后 `SSH and GPG keys.`，  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/a4a804d537bc0e6f102f31ad9221aa3b.png) 然后就是 `New SSH key` 或者 `Add SSH key.`  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8a14e1c03f3f327d2ec95e8f292ee0d7.png)  
起个名字，把刚刚复制的内容粘贴到key里，然后点击 `Add SSH key.` 。就好了，可能还会让输入密码确认一下。  
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ab79d923a48e1ca23d281161247db2b5.png)

### 第五步，测试这个ssh连接

```bash
$ ssh -T git@github.com
# Attempts to ssh to GitHub

# 然后会提示
> The authenticity of host 'github.com (IP ADDRESS)' can't be established.
> RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
> Are you sure you want to continue connecting (yes/no)?
bash1234567
```

注意，上面的 `RSA key fingerprint` 并不是指自己刚刚配置的ssh key，而是用于测试的公用key，包括以下：

```bash
# 当前的R SA key fingerprint
SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8 (RSA)

# 下面的从2021.9.14起支持
SHA256:p2QAMXNIC1TJYWeIOttrVc98/R1BUFWu3/LiyKgUfQM (ECDSA)
SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU (Ed25519)

# 下面这个2021.9.16会停用
SHA256:br9IjFspm1vxR3iA35FWE+4VTyz1hYVLIE2t1/CeyWQ (DSA)
bash123456789
```

然后核对之后没问题，就输入 `yes` 就好了。然后就会提示：

```bash
Warning: Permanently added 'github.com,192.30.255.112' (RSA) to the list of known hosts.
Hi \`username\`! You've successfully authenticated, but GitHub does not provide shell access.
bash
```

这个时候，就算添加好了。

---

如果提示：

```bash
Warning: Permanently added the RSA host key for IP address '140.82.114.3' to the list of known hosts.
Hi your_github_id! You've successfully authenticated, but GitHub does not provide shell access.
bash
```

那么需要重新：

```bash
git remote add origin https://github.com/XXX/XXX.git
# 或者
git remote set-url origin git@github.com:lut/EvolutionApp.git
bash123
```

参考：

- [Github Authentication Failed - … GitHub does not provide shell access](https://stackoverflow.com/questions/26953071/github-authentication-failed-github-does-not-provide-shell-access)

## 🌈6. 后续

### 1\. 警告

2023.3.28 今天突然收到了这样的一个警告错误：

```bash
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
SHA256:uNiVztksCsDhcc0u9e8BujQXVUpKZIDTMczCvj3tD2s.
Please contact your system administrator.
Add correct host key in /Users/XXXX/.ssh/known_hosts to get rid of this message.
Offending RSA key in /Users/XXXX/.ssh/known_hosts:1
Host key for github.com has changed and you have requested strict checking.
Host key verification failed.
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
bash1234567891011121314151617
```

参考： [“Warning: Remote Host Identification Has Changed” — Did GitHub change their RSA key?](https://stackoverflow.com/questions/75830783/warning-remote-host-identification-has-changed-did-github-change-their-rsa)

### 2\. 错误原因

因为github更新了自己的key

### 3\. 解决方案

报错中的

```bash
SHA256:uNiVztksCsDhcc0u9e8BujQXVUpKZIDTMczCvj3tD2s
bash1
```

实际上来自这里： [GitHub’s SSH key fingerprints](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/bbd3fdead9fe2ab8455bfd9cec8f92ed.png)

---

根据 ["Warning: Remote Host Identification Has Changed" — Did GitHub change their RSA key?](https://stackoverflow.com/questions/75830783/warning-remote-host-identification-has-changed-did-github-change-their-rsa)中提供的：github的文章- [We updated our RSA SSH host key](https://github.blog/2023-03-23-we-updated-our-rsa-ssh-host-key/) 可知：

有两种步骤：

1. 运行以下命令，删除旧的key
	```bash
	$ ssh-keygen -R github.com
	bash1
	```
	![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/019de191262a11c334f2e44ad4c70df8.png)

---

2. 手动更新 `~/.ssh/known_hosts` 文件，添加新的key（下面这个新的key和旧的长得很像，但是实际不一样）
	```bash
	github.com ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCj7ndNxQowgcQnjshcLrqPEiiphnt+VTTvDP6mHBL9j1aNUkY4Ue1gvwnGLVlOhGeYrnZaMgRK6+PKCUXaDbC7qtbW8gIkhL7aGCsOr/C56SJMy/BCZfxd1nWzAOxSDPgVsmerOBYfNqltV9/hWCqBywINIR+5dIg6JTJ72pcEpEjcYgXkE2YEFXV1JHnsKgbLWNlhScqb2UmyRkQyytRLtL+38TGxkxCflmO+5Z8CSSNY7GidjMIZ7Q4zMjA2n1nGrlTDkzwDCsw+wqFPGQA179cnfGWOWRVruj16z6XyvxvjJwbz0wQZ75XK5tKSb7FNyeIEs4TT4jk+S4dhPeAUC5y+bDYirYgM4GC7uEnztnZyaVWQ7B381AK4Qdrwt51ZqExKbQpTUNn+EjqoTwvqNj4kqx5QUCI0ThS/YkOxJCXmPUWZbhjpCg56i+2aB6CmK2JGhn57K5mj0MNdBXA4/WnwH6XoPWJzK5Nyu2zB3nAZp+S5hpQs+p1vN1/wsjk=
	bash1
	```
	如果嫌弃这个比较长，不想复制，可以直接命令行（我运行下面的命令会报错，所以我是手动复制的）：
	```bash
	$ curl -L https://api.github.com/meta | jq -r '.ssh_keys | .[]' | sed -e 's/^/github.com /' >> ~/.ssh/known_hosts
	bash1
	```

---

3. 然后就可以像之前一样，测试连接(详见5.ssh使用中的 第五步)：
	```bash
	$ ssh -T git@github.com
	> You've successfully authenticated, but GitHub does not provide shell access.
	bash
	```

然后就可以正常 `git push` 了！