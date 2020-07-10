## 问题关键点： 存验证码的key的唯一性

1. session是用户唯一的，但是只是存在单台服务器上，所以不适合分布式服务器，所以还是需要redis（也有`egg-session-redis`）
2. csrfToken可以作为key的一部分，但是`ctx.cookie.get('csrfToken')`取不到，好像是没有set就get不到，就算`ctx.header.cookie`里面有，所以只能在header里面切割`ctx.header.cookie.split(';')[0].split('=')[1]`简单粗暴，或者用正则`/([^=]+)=([^;]+);?\s*/g`
   更新：ctx.cookie.get不到是因为默认取已加密的，如果要取未加密的应该加`{ encrypt: true }`参数
3. 验证码的key`captchaKey`，可能会在多个地方用到，就想存在控制器的constructor里，这里也有坑
   首先constructor里要先于this调用super，但是spuer要传ctx参数，可是也传不了this.ctx因为等于是super在后面调用了，想起来constructor其实也会接受一个ctx，所以问题解决

```js
constructor (ctx) {
    super(ctx)
    this.captchaKey = 'captcha' + ctx.header.cookie.split(';')[0].split('=')[1]
}
```

## 后台

```js
const captcha = require('svg-captcha')

constructor (ctx) {
    super(ctx)
    this.captchaKey = 'captcha' + ctx.header.cookie.split(';')[0].split('=')[1]
}

// 可以设置宽高，单位像素，但是注意宽不要小于高
// 验证码有效期限为600秒10分钟
async captcha () {
    const { ctx, app } = this
    const option = { size: 4, noise: 1, width: 78, height: 38 }

    const width = parseInt(ctx.query.width)
    const height = parseInt(ctx.query.height)
    if (!isNaN(width) && !isNaN(height) && width > height) {
        option.width = width
        option.height = height
    }

    const cap = await captcha.create(option)
    if (!await app.redis.set(this.captchaKey, cap.text.toLowerCase(), 'EX', 600)) ctx.throw(200, '获取图形验证码失败')

    ctx.body = 'data:image/svg+xml;utf8,' + encodeURIComponent(cap.data)
}

async veriCaptcha (captcha = '', msg = '图形验证码错误') {
    const { ctx, app } = this
    if (await app.redis.get(this.captchaKey) !== captcha) ctx.throw(200, msg)
}
```

## 前台

```html
<el-form-item>
	<el-input placeholder="请输入验证码" v-model="form.captcha">
    <img :src="captchaSvg" style="display:block;width:78px;height:38px;padding:0" slot="append" @click.stop="getCaptcha"/>
  </el-input>
</el-form-item>
```

```js
getCaptcha (width = '', height = '') {
  this.axios.get(`/api/captcha?width=${width}&height=${height}`).then(res => (this.captchaSvg = res), err => console.log(err))
}
```



## 参考

[原来浏览器原生支持JS Base64编码解码](https://www.zhangxinxu.com/wordpress/2018/08/js-base64-atob-btoa-encode-decode/)
[学习了，CSS中内联SVG图片有比Base64更好的形式](https://www.zhangxinxu.com/wordpress/2018/08/css-svg-background-image-base64-encode/)
[SVG 图像入门教程](https://www.ruanyifeng.com/blog/2018/08/svg.html)