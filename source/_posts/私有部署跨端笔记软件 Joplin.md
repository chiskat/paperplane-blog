---
title: 私有部署跨端笔记软件 Joplin
date: 2025-03-20 14:00:00
tags: 
- DOC
- DevOps
categories: 
- DOC
---

拖了接近一年的一篇文章，介绍我近期一直在使用的笔记软件——Joplin。
它可能不是最好的，下文讲到了很多缺点，但它确实有自身的独特之处，值得一试。



# 为什么选择 Jopin

笔记软件有很多，例如 OneNote、印象笔记、石墨文档、有道云笔记、语雀，甚至企业微信、钉钉都可以用来写笔记。

但是，这些软件基本都属于商业软件，有商业化内容甚至广告，需要注册、实名制；更何况数据被厂商所掌控，一旦服务器出岔子或者倒闭，数据就有丢失的风险。

而且，作为开发者，我们还有针对 Markdown 格式、代码高亮、云同步等功能的硬性要求，最好还能支持插件，能私有化数据。这些功能对于商业笔记软件而言，属于小众需求，厂商出于成本考虑，也不会提供这些。

<br />

Joplin 就是这样一款开源、可私有部署的笔记软件。它原生支持 Markdown 语法，也可以使用类似 Word 一样的所见即所得模式，甚至可以同屏显示两者；它提供了安卓、iOS、PC、macOS 多个客户端，所有笔记和附件都能云端同步。

即使你没有服务器，Joplin 也能利用 S3 对象存储、WebDAV、OneDrive 等进行同步；比如你可以注册一个坚果云，使用坚果云免费的 WebDAV；Joplin 官方还提供了付费的云同步服务。
而且，Joplin 的服务端支持多账号、端到端加密，也就是说，你甚至可以和朋友共享一个服务端，使用各自的账号登录并同步，即使服务器持有者也无法读取其他人的笔记。

此外，电脑上的 Joplin 客户端可以自由安装插件；Joplin 官方还提供了浏览器上类似于印象笔记的 “剪藏” 插件，方便你快速保存喜欢的网页内容。

Joplin 官网：https://joplinapp.org
GitHub 仓库：https://github.com/laurent22/joplin
我部署的 Joplin 后端：https://joplin.p01.cc



# 服务端的安装和配置方式

Joplin 需要 PostgreSQL 数据库来存储账户、笔记和附件。推荐的方式是使用 Docker 来部署，如果你没有数据库，那么可以一同把数据库也通过 Docker 来管理。

使用 Docker Compose 部署：

```yaml
services:
  joplin:
    container_name: joplin
    image: joplin/server:latest
    depends_on:
      - postgres
    restart: always
    volumes:
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
    environment:
      - APP_PORT=<端口号>
      - APP_BASE_URL=<网址>
      - DB_CLIENT=pg
      - POSTGRES_CONNECTION_STRING=postgresql://postgres:<密码>@postgres:5432/<数据库名>?schema=public
      - MAX_TIME_DRIFT=0
      - TZ=Asia/Shanghai

  # Joplin 依赖一个 PostgreSQL 或 MySQL 数据库
  # 如果你已经有可用的数据库了，下面这段则不需要
  postgres:
    image: postgres:latest
    container_name: postgres
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: <密码>
      TZ: Asia/Shanghai
    volumes:
      - <数据库持久化目录>:/var/lib/postgresql/data
```

运行后，通过 Nginx 把网站对外暴露，然后通过浏览器访问，即可进行初始化。

初始化时，需要填写账户和邮箱，完成注册后，系统会给我们分配一个管理员账号。



# 客户端安装、使用和定制

本文主要介绍 PC / macOS 的 Joplin 客户端。
可以在官网下载页（https://joplinapp.org/download/）下载并安装；
或者，在官网安装帮助页（https://joplinapp.org/help/install/）有更细分的版本可供选择。

<br />

安装完成后，需要做的最重要的就是配置同步。
（一般安装完成后会进入配置向导，或者可手动通过菜单 → “偏好设置” → “同步” 进行设置）

Joplin 本身被设计成联机使用，因此它支持多种同步方式；
而我们自己部署了它的服务端，因此 “同步目标” 选择 “Joplin 服务器 (Beta)”。

然后，在 “Joplin 服务器 URL” 中填入我们自己部署的 Joplin 服务的完整网址，并在 “Joplin 服务器邮箱” 和 “Joplin 服务器密码” 中输入我们之前创建的账号的邮箱和密码，完成后，点击 “检查同步配置” 按钮，如果配置成功，会提示 “成功！同步配置看起来没问题。”



## 快捷键和样式 bug





## 【必做】优化编辑器样式

默认情况下，尤其是对 Windows 用户而言，Joplin 的文本编辑器在文本字体和代码块方面都有些问题，例如默认是宋体字，代码块也不是等宽字体，甚至设置里的定制字体也不是很好用；好在 Joplin 提供了自定义样式的功能，我们来把它改的更好一些：

打开 “选项” > “外观” > “显示高级选项”，然后点击第一个 “适用于已渲染 Markdown 的自定义样式表” 按钮；
此时会打开一个 CSS 文件，填入：

```css
/* For styling the rendered Markdown */
code {
  font-family: Fira Code, Consolas, Microsoft YaHei, 'Courier New', monospace !important;
}
pre.hljs {
  padding: 0.2em 0.5em !important;
}
.mce-content-body .joplin-editable {
  margin: 0 0.25em !important;
}
```

我这里的 `Fira Code` 是我自己用的代码字体，记得把它换成你自己使用的代码字体，或者，直接删掉这个字体名，使用系统内置的 `Consolas` 字体也不错。

保存此文件，我们继续；
点击第二个 “适用于 Joplin 全域应用样式的自定义样式表” 按钮，这也会打开一个 CSS 文件，填入：

```css
/* For styling the entire Joplin app (except the rendered Markdown, which is defined in `userstyle.css`) */
.CodeMirror.CodeMirror * {
  font-family: Fira Code, Consolas, Microsoft YaHei, 'Courier New', monospace !important;
}
.CodeMirror pre.CodeMirror-line, .CodeMirror pre.CodeMirror-line-like {
  padding-left: 0.5em !important;
  padding-right: 0.5em !important;
}
```

同理，代码字体可以改成你喜欢用的。

经过这样的定制，编辑器中不再会显示宋体字，代码字体也会变成常用的等宽字体，且代码块也不再会有遮挡问题。



# 特色功能：生成分享网页



# 特色功能：网页剪藏



# 扩展插件

