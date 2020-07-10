## **1.全部替换**

string.replace（）函数仅替换第一次出现的情况。
可以在正则表达式的末尾添加/ g来替换所有出现的内容。

```
var example = "potato potato";
console.log(example.replace(/pot/, "tom"));  // "tomato potato"
console.log(example.replace(/pot/g, "tom"));  // "tomato tomato"
```

## **2.提取唯一值**

仅仅通过`Set`对象和`Spread`运算符就可以创建一个唯一值的数组。

```
var entries = [1, 2, 2, 3, 4, 5, 6, 6, 7, 7, 8, 4, 2, 1]
var unique_entries = [...new Set(entries)];
console.log(unique_entries); // [1, 2, 3, 4, 5, 6, 7, 8]
```

## **3.将数字转换为字符串**

只需要使用带空引号集的串联运算符即可

```
var converted_number = 5 + "";
console.log(converted_number); // 5
console.log(typeof converted_number);  // string
```

## 4.将字符串转换为数字

我需要的只是+运算符。

```
the_string = "123";
console.log(+the_string); // 123

the_string = "hello";
console.log(+the_string); // NaN
```

**注意**：因为它仅适用于“字符串数字”。

## 5.随机排列数组中的元素

每天我都像喝大了～

```
var my_list = [1, 2, 3, 4, 5, 6, 7, 8, 9];
var my_list = [1, 2, 3, 4, 5, 6, 7, 8, 9];
console.log(my_list.sort(()=>Math.random() - 0.5)); //[5, 7, 6, 4, 3, 8, 2, 1, 9]
```

## **6.碾平多维数组**

```
var entries = [1, [2, 5], [6, 7], 9];
var flat_entries = [].concat(...entries); // [1, 2, 5, 6, 7, 9]
```

## **7.短路条件**

来看这个例子：

```
if (available) {
    addToCart();
}

//并通过简单地使用变量和函数来缩短它
available && addToCart()
```

### 8.动态属性名

```
const dynamic = 'flavour';
var item = {
    name: 'Coke',
    [dynamic]: 'Cherry'
}
console.log(item);  // { name: "Coke", flavour: "Cherry" }
```

### 9.使用长度调整/清空数组

调整数组长度：

```
var entries = [1, 2, 3, 4, 5, 6, 7];  
console.log(entries.length); // 7  
entries.length = 4;  
console.log(entries); // [1, 2, 3, 4]
```

清空数组：

```
var entries = [1, 2, 3, 4, 5, 6, 7]; 
console.log(entries.length);  // 7  
entries.length = 0;   
console.log(entries);  // []
```

## 参考

[9 Extremely Powerful JavaScript Hacks](https://dev.to/razgandeanu/9-extremely-powerful-javascript-hacks-4g3p)