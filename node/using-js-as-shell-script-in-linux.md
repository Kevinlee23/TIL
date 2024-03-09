### 在 linux 系统下，将 js 文件 当做 shell 脚本执行

#### 1. 通过#！注释来指定当前脚本使用的解析器

`#! /usr/bin/env node` 表示由 nodejs 解析

#### 2. 赋予脚本文件执行权限

`chmod +x /home/usr/bin/scriptName.js`

#### 3. 在 PATH 环境变量中指定的某个目录下，创建一个软链文件，文件名与我们希望用的终端命令同名

`sudo ln -s /home/user/bin/scriptName.js /usr/local/bin/cmdName`

#### result

处理完毕后，我们就能在任何目录下使用 `cmdName` 这个命令