- https://github.com/lecepin/Debugging-env-browser
- https://webpack.js.org/configuration/dev-server/#devserverproxy
- https://vitejs.dev/config/server-options.html#server-proxy
- https://github.com/chimurai/http-proxy-middleware#options

#### 1. 模拟一个需要登陆态的后端接口

backend-server.js， http://localhost:3001

```js
const http = require("http");

// 用于存储用户的会话信息
const sessions = {};

// 生成随机的会话ID
function generateSessionId() {
  return Math.random().toString(36).substring(2, 15);
}

const app = http.createServer((req, res) => {
  // res.setHeader("Access-Control-Allow-Origin", "http://localhost:3000");
  // res.setHeader("Access-Control-Allow-Credentials", "true");

  if (req.url === "/login") {
    // 模拟登录逻辑
    const sessionId = generateSessionId();
    sessions[sessionId] = { loggedIn: true };

    // 设置会话ID的Cookie
    // 模拟多 cookie key 场景
    res.setHeader("Set-Cookie", [
      `sessionId=${sessionId}; SameSite=Strict`,
      `sessionId2=${sessionId}; SameSite=Strict`,
    ]);
    res.statusCode = 200;
    res.setHeader("Content-Type", "text/plain");
    res.end("login successful");
  } else if (req.url === "/logout") {
    // 模拟登出逻辑
    const cookies = req.headers.cookie;
    if (cookies && cookies.includes("sessionId")) {
      const sessionId = cookies.split("sessionId=")[1].split(";")[0];
      delete sessions[sessionId];
    }

    // 清除会话ID的Cookie
    res.setHeader(
      "Set-Cookie",
      "sessionId=; Path=/; HttpOnly; SameSite=Strict; Expires=Thu, 01 Jan 1970 00:00:00 GMT"
    );
    res.statusCode = 200;
    res.setHeader("Content-Type", "text/plain");
    res.end("Logout successful");
  } else if (req.url === "/api/a") {
    // 验证登录状态
    const cookies = req.headers.cookie;

    if (
      cookies &&
      cookies.includes("sessionId") &&
      cookies.includes("sessionId2")
    ) {
      const sessionId = cookies.split("sessionId=")[1].split(";")[0];
      const sessionId2 = cookies.split("sessionId2=")[1].split(";")[0];

      if (
        sessionId == sessionId2 &&
        sessions[sessionId] &&
        sessions[sessionId].loggedIn
      ) {
        res.statusCode = 200;
        res.setHeader("Content-Type", "application/json");
        res.end(JSON.stringify({ content: "lalala", success: true }));
        return;
      }
    }

    // 未登录
    res.statusCode = 401;
    res.setHeader("Content-Type", "text/plain");
    res.end("Not logged in, please log in first");
  } else {
    res.statusCode = 404;
    res.setHeader("Content-Type", "text/plain");
    res.end("Not Found");
  }
});

app.listen(3001, () => {
  console.log("需要登陆认证 - Server is running on http://localhost:3001");
});
```

# 2. 实现方法

启动前端项目地址 http://localhost:3000，模拟后端 http://localhost:3001。

可实现方法：

1. 系统级别代理工具（如 Charles、Whistle）直接修改 3001 的 CORS 及将登陆后的 3001 的 cookies 逐个写入代理规则。
2. 使用 webpack 或者 vite 等内置的 proxy 的配置重写，方式同 1。唯一好处再于不是系统级的，不对系统网络造成影响。
3. 直接 Electron，将 CORS 及 Cookies 的 SameSite 设置为 None 来解决。极为方便。

#### 2.1 vite 实现

vite.config.js

```js
export default {
  server: {
    proxy: {
      "/api": {
        target: "http://localhost:3001",
        changeOrigin: true,
      },
    },
  },
  plugins: [
    {
      name: "append-backend-cookies",
      configureServer(server) {
        server.middlewares.use((req, res, next) => {
          req.headers["Cookie"] = "sessionId=wzqa6tjqgl;sessionId2=wzqa6tjqgl";
          next();
        });
      },
    },
  ],
};
```

前端请求：

```js
fetch("/api/a")
  .then((data) => data.text())
  .then((data) => alert(data))
  .catch((e) => alert("Err:" + e));
```

缺点，繁琐，不断的修改 cookies。

#### 2.2 webpack OR umi

config.js OR webpack.config.js

```js
export default {
  {
    '/api/': {
      target: "http://localhost:3001",
      changeOrigin: true,
      secure:false,
      onProxyReq: (proxyReq) => {
        proxyReq.setHeader(
          'Cookie',
          'sessionId=wzqa6tjqgl;sessionId2=wzqa6tjqgl',
        )
      },
    },
  }
}
```

#### 2.3 electron

```js
function createWindow() {
  const win = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      webSecurity: false,
    },
  });

  win.loadURL("https://localhost:3000");
  // win.webContents.openDevTools();
}

function disableSamesiteCookies(filter = ["*://*/*"]) {
  session.defaultSession.webRequest.onHeadersReceived(
    { urls: filter },
    (details, callback) => {
      const newCookies = [];

      details?.responseHeaders?.["set-cookie"]?.map((item) =>
        newCookies.push(item.split("; ")[0] + "; Secure; SameSite=None")
      );
      details.responseHeaders["set-cookie"] = newCookies;

      callback({ cancel: false, responseHeaders: details.responseHeaders });
    }
  );
}

app.whenReady().then(() => {
  createWindow();
  disableSamesiteCookies();

  //  ……
});
```
