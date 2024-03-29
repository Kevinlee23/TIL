### github 连接超时问题

#### 解决方案一, 修改dns

第一先查找 `github.com` 和 `github.global.ssl.fastly.net` 的 ip 地址
使用 ip 查询工具: https://www.ipaddress.com/

第二布配置 host 文件:

文件位置:

- windows `C:\windows\system32\drivers\etc`
- linux `/etc/hosts`

`host`

```txt
140.82.112.3 github.com
151.101.1.194 github.global.ssl.fastly.net
```

第三步, 刷新 host 文件

`windows`

```cmd
ipconfig /flushdns     #清除DNS缓存内容。
ps:ipconfig /displaydns #显示DNS缓存内容
```

`linux`

```shell
systemctl restart nscd
```

#### 解决方案二，使用代理

`git config --global http.proxy 127.0.0.1:ssrport`
`git config --global https.proxy 127.0.0.1:ssrport`
