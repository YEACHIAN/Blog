## 不要使用toLocaleString

toLocaleString = toLocaleDateString + toLocaleTimeString 虽然好看，但是不可逆（从String还原成Date），还不如使用

```js
new Date().toGMTString() // Mon, 11 Nov 2019 08:44:47 GMT

new Date().toISOString() // 2019-11-11T08:44:55.976Z
new Date().toJSON() // 2019-11-11T08:47:45.854Z

new Date().toTimeString() // 16:45:05 GMT+0800 (中国标准时间)

new Date().toString() // Mon Nov 11 2019 16:45:14 GMT+0800 (中国标准时间)
new Date() // Mon Nov 11 2019 16:47:16 GMT+0800 (中国标准时间)

new Date().toUTCString() // Mon, 11 Nov 2019 08:45:26 GMT
```

## 解构赋值

对于一维对象，可以进行变量改名，和设置默认值

```js
const person = { name: 'yy', age: 22 }
const { name: n, age = 18, sex = 'male' } = person
console.log(n, age, sex) // yy 22 male
```

处理嵌套对象

```js
const persons = { yy: { n: 'yy', a: 22 }, zz: { n: 'zz', a: 21 } }
const { yy: { n: name, a: age } } = persons
console.log(name, age) // yy 22
```

参考: [深入理解ES6--5.解构：更方便的数据访问](https://juejin.im/post/5ae555c1518825671a638b82)

## reduce去重

本质：打标记

```js
/**
 * @description
 * 数组去重
 * 核心思想: 打标记
 * @param {Array} arr
 * @returns {Array} 返回去重后的数组
 */
function arrDedup(arr) {
    if (!Array.isArray(arr)) throw new Error('非数组')
    const mark = []
    return arr.reduce((cur, next) => {
        if (!mark[next]) {
            mark[next] = true
            cur.push(next)
        }
        return cur
    }, [])
}
```

## 两个数组是否相等

如果是无序的怎么处理

```js
/**
 * @description
 * 无序相等
 * [1, 2] == [2, 1]
 * @param {Array} arra
 * @param {Array} arrb
 * @returns {Number} 不同之处有几处
 */
function unorderEqual(arra, arrb) {
    if (!Array.isArray(arra) || !Array.isArray(arrb)) throw new Error('非数组')
    // [] [""] 应该相等
    // 去除空元素
    arra = arra.filter(x => x)
    arrb = arrb.filter(x => x)
    const arr = this.arrDedup([...arra, ...arrb])
    return Math.abs(arr.length - arra.length)
}
```

## goto 的替代品 label 标记

js 没有 goto 语句，标记只能和 break 或 continue 一起使用，最方便的就是用于跳出最外层循环

- continue 只能标记**循环**
- break 可以标记**任何语句**

label 用于 continue

```js
loop:
for (let i = 0; i < 3; i++) 
   for (let j = 0; j < 3; j++)
      if (i == 1 && j == 1)
		continue loop
      else 
		console.log(i, j)
/**
结果:  跳过 i == 1 && j == 1
0 0
0 1
0 2
1 0
2 0
2 1
2 2
*/
```

label 用于 break

```js
loop:
for (let i = 0; i < 3; i++) 
   for (let j = 0; j < 3; j++)
      if (i == 1 && j == 1)
		break loop
      else 
		console.log(i, j)
/**
结果:  在 i == 1 && j == 1 处截止
0 0
0 1
0 2
1 0
*/
```

## 关于时间口语化表达的纠结

在今天凌晨，说昨天23点59分，可以说是一天前吗？

```js
/**
 * @description
 * 时间转换为更加口语化的形式
 * @param {String|Number} before 传string就是可以转为时间格式的字符串 传number就是时间戳 两者都可以放入new Date()
 */
const computeTime = before => {

  /**
   * Date对象直接相减就是时间戳的相减
   */
  before = new Date(before)
  const mid = new Date() - before

  /**
   * 相差多少天前是 向上(ceil) 取整
   * 比如 前天晚上 可以说 两天前 Math.ceil(((new Date('2019/10/16 00:00') - new Date('2019/10/14 23:59')) / 86400000)) 1.0006944444444446 -> 2
   * 比如 昨天晚上 可以说一天前 Math.ceil((new Date('2019/10/16 00:00') - new Date('2019/10/15 23:59')) / 86400000) 0.0006944444444444445 -> 1 跟下面的情况一样 差距太小不能说一天前 而是说 而是更小单位前
   * Math.ceil((new Date('2019/10/15 12:12') - new Date('2019/10/15 12:11')) / 86400000) 0.0006944444444444445 
   */
  const dayDiff = mid / 86400000

  /**
   * 30天或以上直接显示日期 YYYY年MM月DD日
   * date是month中的天 day是week中的天
   * const hours = before.getHours()
   * const minutes = before.getMinutes()
   * const secends = before.getSeconds()
   * const millisecends = before.getMilliseconds()
   */
  if (parseInt(dayDiff) >= 30) {
    const year = before.getFullYear()
    const month = ('00' + (before.getMonth() + 1)).slice(-2)
    const date = ('00' + before.getDate()).slice(-2)
    return `${year}年${month}月${date}日`
  }

  else if (1 < dayDiff && dayDiff < 7) return Math.ceil(dayDiff) + '天前'

  else if (7 <= parseInt(dayDiff) && dayDiff < 30) return Math.ceil(dayDiff / 7) + '周前'

  else if (mid / 3600000 > 1 && mid / 60000 > 60) return Math.ceil(mid / 3600000) + '小时前'

  else if (1 < mid / 60000 && mid / 60000 < 60) return Math.ceil(mid / 60000) + '分钟前'

  else return '刚刚'
}
```

## 为什么for...in和for...of可以用const但是for不行?

首先看看`var`, `let`, `const`三者的区别

| 名称  | 重新定义 | 重新赋值 |
| ----- | -------- | -------- |
| var   | ✅        | ✅        |
| let   | ❌        | ✅        |
| const | ❌        | ❌        |

```js
const arr = [1,2,3]
for (const i in arr) console.log(i) // ok
for (const i of arr) console.log(i) // ok
for (const i = 0; i < arr.length; i++) console.log(i) // error
```

for...in for...of可以理解为是函数, 每次迭代相当于调用一次函数，生成一个新的作用域，相当于定义了多个作用域中值不同的const i，没有重新赋值，而 for 循环只生成了一次作用域

## 任何 async 函数都会返回一个 promise

```js
async function f () {
    console.log('await func')
}
f() // Promise {<resolved>: undefined}
await f() // 直接把 resolve 的 undefined 取出来了
```

## 何如优雅地处理 aysnc/await 的异常？

async/await + then(promise) + Go语言风格错误处理[err, data]

```js
const promisify = (name = '', opts = {}) => 
  new Promise((success, fail) => 
    wx[name]({ ...opts, success, fail })
  )

const [err, shopCart] = await promisify('getStorage', { key: 'shopCart' })
.then(({ data: shopCart }) => [null, shopCart], err => [err])

console.log(err, shopCart)
```

错误只要不 throw 那都好说
参考: [async/await 优雅的错误处理方法](https://juejin.im/post/5c49eb28f265da613a545a4b)
参考: [13篇文章，教你学会ES6知识点](https://juejin.im/post/5af153e96fb9a07aa34a35eb#heading-3)