算法一

```js
var items = [
    ['a', 'A'],
    ['b', 'B'],
    ['c', 'C']
]

var result = {
    'a-b-c': { remain: 10, price: 1000 },
    'a-B-c': { remain: 10, price: 1000 },
    'A-B-C': { remain: 10, price: 1000 }
}

var selected = ['a']

/**
 * @description
 * 过滤掉x中包含y掉部分
 * @param x {Array}
 * @param y {Array}
 */
var difference = (x, y) => x.filter(x => !y.includes(x))

var itemStatus = {}
items.forEach(currItems => {
    var tempSelected = difference(selected, currItems)
    // console.log(selected, currItems, tempSelected)
    currItems.forEach(item => {
        var tempSku = [ item, ...tempSelected ]
        for (const skuKey in result) {
            const sku = skuKey.split('-')
            if (difference(tempSku, sku).length) {
                itemStatus[item] = true
                break
            }
        }
    })
})

console.log(itemStatus)

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

算法二

```js
var items = [
    ['a', 'A'],
    ['b', 'B'],
    ['c', 'C']
]

var result = {
    'a-b-c': { remain: 10, price: 1000 },
    'a-B-c': { remain: 2, price: 1100 },
    'A-B-C': { remain: 10, price: 1000 }
}

var data = {}

var selected = ['a']

var getRemainByKey = selected => {
    const key = selected.join('-')

    if (data[key]) return data[key]

    if (selected.length == items.length)
        return result[key] ? data[key] = result[key].remain : data[key] = 0

    let remain = 0
    const tempSelected = []
    for (const currItems of items) {
        const exist = currItems.find(item => item == selected[0])
        if (exist && selected.length)
            tempSelected.push(selected.shift())
        else {
            currItems.forEach(item => {
                remain += getRemainByKey([item, ...selected, ...tempSelected])
            })
            break
        }
    }
    return data[key] = remain
}

console.log(getRemainByKey(selected))
```

算法三

```js
var result = {
    'a-b-c': { remain: 10, price: 1000 },
    'a-B-c': { remain: 2, price: 1100 },
    'A-B-C': { remain: 10, price: 1000 }
}
var items = [['a', 'A'], ['b', 'B'], ['c', 'C']]
var data = {}

function getRemainByKey (selected) {

    const key = selected.join('-')
    if (typeof data[key] != 'undefined') return data[key]
    if (selected.length === items.length) return result[key] ? data[key] = result[key].remain : data[key] = 0

    let remain = 0
    const tempSelected = []
    for (const currItems of items) {
        const exist = currItems.some(item => item === selected[0]) && selected.length
        if (!exist) {
            currItems.forEach(item => {
                remain += getRemainByKey([...tempSelected, item, ...selected]) // 顺序很重要!
            })
            break
        }
        tempSelected.push(selected.shift())
    }
    return data[key] = remain
}

console.log(getRemainByKey(['a']))



/**
 * @description
 * [a, b] -> [[ a ], [ b ], [a, b]]
 * @param arr {Array} 一维数组
 */
function powerset(arr) {
    const ps = [[]]
    for (let i = 0; i < arr.length; i++) {
        for (let j = 0, len = ps.length; j < len; j++) {
            ps.push([...ps[j], ...arr[i]])
        }
    }
    return ps
}

/**
 * @description
 * 生成所有组合的库存组合
 * @param stockData {Object} 库存数据 {'a-b-c': { remain: 10, price: 1000 }}
 */
function stockComb (stockData) {
    if (typeof stockData != 'object')
        throw new Error('不是对象')
    const res = {}
    for (const skuKey in stockData) {
        const { remain, price } = stockData[skuKey]
        powerset(skuKey.split('-')).forEach(sku => {
            const key = sku.join('-')
            if (res[key]) {
                res[key].price = (res[key].price << 0) + price
                res[key].remain = (res[key].remain << 0) + remain
            } else
                res[key] = { remain, price }
        })
    }
    return res
}
```

还有一些在 [#50](https://github.com/TENCHIANG/blog/issues/50) 这里，数组去重和两数组无序相等

## 参考

[sku 多维属性状态判断算法](https://keelii.com/2016/12/22/sku-multi-dimensional-attributes-state-algorithm/)
[商品规格SKU组合查询算法总结](https://quincychen.cn/sku_algorithm/)