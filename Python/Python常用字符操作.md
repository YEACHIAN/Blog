## 判断类型

| 方法               | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| string.isspace()   | 如果 string 中只包含空格，则返回True                         |
| string.isalnum()   | 如果 string 至少有一个字符并且所有字符都是字母或数字则返回True |
| string.isalpha()   | 如果 string 至少有一个字符并且所有字符都是字母则返回True     |
| string.isdecimal() | 如果 string 只包含数字则返回True，全角数字                   |
| string.isdigit()   | 如果 string 只包含数字则返回True，全角数字、（1）、\u00b2    |
| string.isnumeric() | 如果 string 只包含数字则返回True，全角数字、汉字数字         |
| string.istitle()   | 如果 string 是标题化的（每个单词的首字母大写）则返回True     |
| string.islower()   | 如果 string 中包含至少一个区分大小写的字符，并且所有这些（区分大小写的）字符都是小写，则返回True |
| string.isupper()   | 如果 string 中包含至少一个区分大小写的字符，并且所有这些（区分大小写的）字符都是大写，则返回True |





## 查找和替换

| 姓名                                                    | 说明                                                         |
| :------------------------------------------------------ | :----------------------------------------------------------- |
| string.startswith(str)                                  | 检查字符串是否是以 str 开头，是则返回 True                   |
| string.endSwith(str)                                    | 检查字符串是否是以 str 结束，是则返回 True                   |
| string.find(str, start=0, end=len(string)               | 检查 str 是否包含在 string 中，如果 start 和 and 指定范围，则检查是否包含在指定范围内，如果是返回开始的索引值，否则返回-1 |
| string.rfind(str, start=0, end=len(string)              | 类似 find() 函数，不过是从右边开始查找                       |
| string.index(str, start=0, end=len(string))             | 跟 find() 方法类似，只不过如果 str 不再 string 会报错        |
| string.rindex(str, start=0, end=len(string))            | 类似于 index(),不过是从右边开始                              |
| string.replace(old_str, new_str, num=string.count(old)) | 把 string 中的 old_str 替换成 new_str，如果 num 指定，则替换不超过 num 次 |

## 大小写转换
| 方法                | 说明                             |
| ------------------- | -------------------------------- |
| string.capitalize() | 把字符串的第一个字符大写         |
| string.title()      | 把字符串的每个单词首字母大写     |
| string.lower()      | 转换 string 中所有大写字符为小写 |
| string.upper()      | 转换 string 中的小写字母为大写   |
| string.swapcase()   | 翻转 string 中的大小写           |


## 文本对齐

| 方法                 | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| string.ljust(width)  | 返回一个原字符串左对齐，并使用空格填充至长度 width 的新字符串 |
| string.rjust(width)  | 返回一个原字符串右对齐，并使用空格填充至长度 width 的新字符串 |
| string.center(width) | 返回一个原字符串居中，并使用空格填充至长度 width 的新字符串  |

## 去除空白字符

| 方法            | 说明                               |
| --------------- | ---------------------------------- |
| string.lstrip() | 揭掉 string 左边（开始）的空白字符 |
| string.rstrip() | 揭掉 string 右边（开始）的空白字符 |
| string.strip()  | 揭掉 string 左右两边的空白字符     |


## 拆分和链接 

| 方法                     | 说明                                                         |
| ------------------------ | ------------------------------------------------------------ |
| string.partition(str)    | 把字符串 string 分成一个 3 元素的元组（str前面，str，str后面） |
| string.rpartition(str)   | 类似于 partition() 函数，不过是从右边开始查找                |
| string.split(str="",num) | 以 str 为分隔符切片 string，如果 num 有指定值，则仅分隔 num+1 个子字符串，str 默认包含'\r','\t','\n' 和空格 |
| string.splitlines()      | 按照行（'\r','\n','\r\n'）分隔，返回一个包含各行作为元素的列表 |
| string.join(seq)         | 以 string 作为分隔符，将seq中所有的元素（的字符串表示）合并为一个新的字符串 |