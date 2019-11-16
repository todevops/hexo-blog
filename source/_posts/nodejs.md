---
title: Node.js 安装指南
date: 2018-11-27
categories:
  - linux
tags:
  - linux
  - nodejs
---

Node.js 安装指南
<!-- more -->

https://github.com/creationix/nvm

To install or update nvm, you can use the install script using cURL:
```shell
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
```
or Wget:
```shell
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
```

```shell
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```