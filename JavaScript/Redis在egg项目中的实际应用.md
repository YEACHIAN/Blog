

## 依赖配置

```
npm install egg-redis --save
```

> config/plugin.js

`enable: true`启用redis

```
module.exports = {
  redis: { enable: true, package: 'egg-redis' }
}
```

> config/config.default.js

```
module.exports = {
	redis: {
        client: {
            port: process.env.REDIS_PORT || 6379,
            host: process.env.REDIS_HOST ||  '127.0.0.1',
            password: process.env.REDIS_PASSWORD ||  null,
            db: '0'
        }
    }
}
```

## 封装redis

其实没必要封装，封装只是为了使用方便和代码统一。

### service

> app/service/cache.js

```
'use strict';
const Service = require('egg').Service;

class CacheService extends Service {
  async set(key,value,seconds) {
    value = JSON.stringify(value);
    if(this.app.redis){
        if(!seconds){
            await this.app.redis.set(key,value);
        }else{
            await this.app.redis.set(key,value,'EX',seconds)
        }
    }
  }

  async get(key){
      if(this.app.redis){
          var data = await this.app.redis.get(key);
          if(!data) return;
          return JSON.parse(data)
      }
  }
}

module.exports = CacheService;
```

### controller

如果没有设置redis缓存，就去请求数据,再设置缓存

```
let data = await this.ctx.service.cache.get('user');
if(!data){
  data = await this.ctx.model.Nav.find({"username":'xxx'});
  await this.ctx.service.cache.set('user',data,60*60);
}
```