---
title: homebrew常用命令
date: 2017-02-11 14:40:10
tags: [homebrew]
---

- 安装

```
ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/Go/install)"
```

- 搜索

```
brew search XXX
```

- 查询

```
brew info XXX
```

- 更新自己

```
brew update 
```

- 是否有新版本

```
brew outdated
```

列出所有安装的软件里可以升级的软件

- 升级软件

```
brew upgrade
```

升级所有可以升级的软件们

```
brew upgrade XXX
```

升级某个软件

- 清理

```
brew cleanup
```

清理不需要的版本极其安装包缓存