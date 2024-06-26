# Cloudflare worker adapter

由于国内糟糕的网络环境, 导致使用`wrangler dev`调试基本不太可能。使用cloudflare workers网页面板调试也比较复杂。所以这里提供一个简单的适配器，可以在本地调试。

可以使用vscode断点调试，加上`nodemon`可以实现热更新。

<img width="600" alt="image" src="https://user-images.githubusercontent.com/9513891/224906690-d9692649-ab5a-4c5a-98e2-49dc122d611a.png">


### Usage

```shell
npm i cloudflare-worker-adapter
```

### Example

https://github.com/TBXark/ChatGPT-Telegram-Workers

```js
import adapter, { bindGlobal } from 'cloudflare-worker-adapter'
import worker from '../main.js'
import { SqliteCache } from 'cloudflare-worker-adapter/cache/sqlite.js'
import fs from 'fs'
import HttpsProxyAgent from 'https-proxy-agent'
import fetch from 'node-fetch'


const config = JSON.parse(fs.readFileSync('./config.json', 'utf-8'))
const cache = new SqliteCache(config.database)

const proxy = config.https_proxy || process.env.https_proxy || process.env.HTTPS_PROXY
if (proxy) {
    console.log(`https proxy: ${proxy}`)
    const agent = new HttpsProxyAgent(proxy)
    const proxyFetch = async (url, init) => {
        return fetch(url, {agent, ...init})
    }
    bindGlobal({ 
        fetch: proxyFetch
    })
}

adapter.startServer(config.port, config.host, config.toml, {DATABASE: cache}, {server: config.server}, worker.fetch)
```

### Best Practice

如果你有在公网被调用的需求，使用内网穿透工具，将本地调试的端口映射到外网，这样就可以正常的交互了。

这里只对KV进行了基本的实现，如果你的项目中遇到的没有实现的类或者方法。可以自行实现之后使用`bindGlobal`进行绑定。同时欢迎把你的实现提交到这个项目中。
