---
title: "输出shell脚本头部的注释做帮助信息"
date: 2020-07-17T12:47:47+08:00
---

博主[@reconquestio](https://twitter.com/reconquestio)发了一篇文章[Help message for shell scripts](https://samizdat.dev/help-message-for-shell-scripts/)展示一个技巧，将帮助信息写在 Bash 脚本脚本的头部，然后只要执行"脚本名 + help"，就能输出这段帮助信息。我翻译了一下，权做参考。

把所有的帮助信息写在脚本的最开始：

```bash
#!/bin/bash
###
### my-script — does one thing well
###
### Usage:
###   my-script <input> <output>
###
### Options:
###   <input>   Input file to read.
###   <output>  Output file to write. Use '-' for stdout.
###   -h        Show this message.
```

然后`help`函数这么写：

```bash
help() {
    sed -rn 's/^### ?//;T;p' "$0"
}
```

调用`help`函数：

```bash
if [[ $# == 0 ]] || [[ "$1" == "-h" ]]; then
    help
    exit 1
fi
```

完整的例子可以参考作者的[`gist`](https://gist.github.com/kovetskiy/a4bb510595b3a6b17bfd1bd9ac8bb4a5)
