---
title: About Hexo
tags: Hexo
categories: IT
---
Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## Quick Start

### Create a new post

``` bash
$ hexo new post "My New Post" #或者
$ hex new "MyNew Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
# usally for local test
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/deployment.html)


### Relate info

* 一些工具
看推荐用的[sublime text](https://www.yejinmo.com/sublime-text-3-3103/), 需要先安装Package Control，然后安装Markdown相关的插件
[http://tongji.baidu.com/web/welcome/login](http://tongji.baidu.com/web/welcome/login)

* ref: [Markdown简单的世界](https://wizardforcel.gitbooks.io/markdown-simple-world/content/hexo-tutor-7.html)

* theme: [NexT](https://github.com/iissnan/hexo-theme-next)

ps: 
只需要在根下的_config.yml里替换掉theme就行, 其他的第三方可以在theme下的_config.yml里修改，比如说说评论和百度流量统计。
在部署的时候，留意github地址，比如https://swordhqm.github.io.
碰到问题，无法进入page是因为邮箱没有认证。
hexo 正常有三种，post，page，draft，其中draft的可以通过publish 回到post
hexo 代码块的高亮是通过[highlight.js](http://highlightjs.readthedocs.io/en/latest/css-classes-reference.html) 来完成高亮的

表格语法

```markdown
| 项目        | 价格   |  数量  |
| --------   | -----:  | :----:  |
| 计算机     | $1600 |   5     |
| 手机        |   $12   |   12   |
| 管线        |    $1    |  234  |
```

* create tags

hex new page "tags" --创建标签页面
```bash
title: Tagcloud
date: 2014-12-22 12:39:04
type: "tags"
comments: false # 关闭这个页面的多说或者 Disqus 评论
```

hex new page "My new Post" --设定tag 和 category
```bash
title: About Hexo
tags: Hexo
categories: Hexo
```

hex new page "categories"

```bash
title: categories
date: 2016-07-20 17:15:15
type: "categories"
comments: false
```

* 站内搜索和mp3
[url](http://jijiaxin89.com/2015/08/21/%E7%8E%A9%E8%BD%AChexo%E5%8D%9A%E5%AE%A2%E4%B9%8Bnext/)
