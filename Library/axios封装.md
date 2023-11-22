### 封装 Axios

#### 为什么要封装 axois

- 利于不同环境的配置
- 请求前参数处理
- 请求后数据处理
- 异常处理
- 请求错误后的重试
- 接口的日志收集
- 避免重复请求
- ...

```javascript
// utils
export const sessionCache = {
  set(key, value) {
    if (!sessionStorage) {
      return;
    }
    if (key != null && value != null) {
      sessionStorage.setItem(key, value);
    }
  },
  get(key) {
    if (!sessionStorage) {
      return null;
    }
    if (key == null) {
      return null;
    }
    return sessionStorage.getItem(key);
  },
  setJSON(key, jsonValue) {
    if (jsonValue != null) {
      this.set(key, JSON.stringify(jsonValue));
    }
  },
  getJSON(key) {
    const value = this.get(key);
    if (value != null) {
      return JSON.parse(value);
    }
  },
  remove(key) {
    sessionStorage.removeItem(key);
  },
};

// 参数处理
export function tansParams(params) {
  let result = "";
  for (const propName of Object.keys(params)) {
    const value = params[propName];
    var part = encodeURIComponent(propName) + "=";
    if (value !== null && value !== "" && typeof value !== "undefined") {
      if (typeof value === "object") {
        for (const key of Object.keys(value)) {
          if (
            value[key] !== null &&
            value[key] !== "" &&
            typeof value[key] !== "undefined"
          ) {
            const params = propName + "[" + key + "]";
            var subPart = encodeURIComponent(params) + "=";
            result += subPart + encodeURIComponent(value[key]) + "&";
          }
        }
      } else {
        result += part + encodeURIComponent(value) + "&";
      }
    }
  }
  return result;
}
```

```javascript
// 判断是否是正式环境, 具体实现根据项目不同而不同
const isProd = () => import.meta.env.VITE_APP_ENV === "production";

const service = axois.create({
  baseURL: isProd ? baseURL__prod : baseURL__dev, // 请求的基本路由
  timeout: 1000, // 超时时间
  withCredentials: true, // 是否允许携带 cookie
});

// 配置通用请求头
const headers = {
  "Content-Type": "application/json",
  // 处理项目的鉴权, 如果我们使用了第三方的库, 如 pinia, reduc等, 如果想直接在此处获取配置，是有问题的，所以可以动态配置在请求拦截 use 中
  // Authorization: getToken(),
};

// 请求前参数处理
service.interceptors.request.use(
  (config) => {
    config.headers = {
      ...headers,
      ...config.headers,
    };

    // 是否需要设置token
    const isToken = (config.headers || {}).isToken === false;
    if (!isToken) {
      // tokenCallback 通指获取token的方法, 看项目是使用什么来储存token
      config.headers["Authorization"] = "Bearer " + tokenCallback();
    }

    // 是否需要防止重复提交
    const isRepeatSubmit = (config.headers || {}).repeatSubmit === false;
    if (
      !isRepeatSubmit &&
      (config.method === "post" || config.method === "put")
    ) {
      const requestObj = {
        url: config.url,
        data:
          typeof config.data === "object"
            ? JSON.stringify(config.data)
            : config.data,
        time: new Date().getTime(),
      };

      const sessionObj = sessionCache.getJSON("sessionObj");
      if (
        sessionObj === undefined ||
        sessionObj === null ||
        sessionObj === ""
      ) {
        sessionCache.setJSON("sessionObj", requestObj);
      } else {
        const s_url = sessionObj.url; // 请求地址
        const s_data = sessionObj.data; // 请求数据
        const s_time = sessionObj.time; // 请求时间
        const interval = 1000; // 间隔时间(ms)，小于此时间视为重复提交
        if (
          s_data === requestObj.data &&
          requestObj.time - s_time < interval &&
          s_url === requestObj.url
        ) {
          const message = "数据正在处理，请勿重复提交";
          console.warn(`[${s_url}]: ` + message);
          return Promise.reject(message);
        } else {
          sessionCache.setJSON("sessionObj", requestObj);
        }
      }
    }

    // get请求映射params参数
    if (config.method === "get" && config.params) {
      let url = config.url + "?" + tansParams(config.params);
      url = url.slice(0, -1);
      config.params = {};
      config.url = url;
    }

    return config;
  },
  (error) => {
    // 错误处理
    console.log(error);
    Promise.reject(error);
  }
);

const errorCode = {
  401: "无效的会话，或者会话已过期，请重新登录。",
  403: "当前操作没有权限",
  404: "访问资源不存在",
  default: "系统未知错误，请反馈给管理员",
};

// 请求后数据处理
service.interceptors.response.use(
  (res) => {
    // 未设置状态码则默认成功状态
    const code = res.data.code || 200;
    // 获取错误信息
    const msg = errorCode[code] || res.data.msg || errorCode["default"];

    if (code === 401) {
      loginTimeoutCallback(); // 登录状态回调, 根据项目不同设置
      return Promise.reject(msg);
    } else if (code === 500) {
      showMessage({ type: "danger", message: msg });
      return Promise.reject(msg);
    } else if (code === 601) {
      showMessage({ type: "warning", message: msg });
      return Promise.reject(msg);
    } else {
      return res.data;
    }
  },
  (error) => {
    message.error("请求异常，请稍后重试！");
    return Promise.reject(error);
  }
);
```
