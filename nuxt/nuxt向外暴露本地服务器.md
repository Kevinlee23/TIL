vue2, vue3 的项目是修改启动地址为 本地 ip

而在 nuxt 中是通过修改启动项来实现:

`package.json`

```json
{
  "script": {
    "serve": "nuxi dev --dotenv .env.development --host"
  }
}
```
