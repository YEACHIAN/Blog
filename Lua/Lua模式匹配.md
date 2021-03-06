## Lua模式匹配 Patterns

* 是精简版的正则表达式
* 支持模式匹配的函数：string.find、string.gmatch,、string.gsub、string.match
* 魔术字符（magic characters）：`^$()%.[]*+-?`（类似关键字）

### 字符类 Character Class

* **x**：表示字符 x 本身（非魔术字符）
* **.**（点）：表示任何字符 
* **%a**：字母
* **%c**：控制字符
* **%d**：数字
* **%g**：可打印字符（除空白符）
* **%s**：空白符
* **%l**：小写字母（lower，[a-z]不一定表示小写, %l 可移植性更好）
* **%u**：大写字母（upper）
* **%p**：标点符号
* **%w**：字母和数字
* **%x**：十六进制数
* **%x**：表示字符 x 本身（x为非数字字符，一般用来对魔术字符的转义）
* **%z**：字符 0（模式里不能包含0，请用 %z 代替）
* 字符类构成的集合（方括号括起来）

### 字符类集合

* 字符类集合：**方括号**括起来的字符类集合（也是字符类，格式 **[set]**，无需任何分隔符）
  * 不加方括号：数量和顺序都要一致
  * 加方括号：数量和顺序不要求一直
* **\-** 表示从多少到多少（范围）
  * **[0-7]** 表示八进制数字
  * 范围尽量只用简单字符（无 %）
* **^** 表示**补集**（相反）
* % 开头的都是小写，**大写表示补集**（如 `%S` 表示所有非空格的字符）

### 模式条目 Pattern Item

* 模式条目：字符类配单个字符，或有特殊后缀表示字符数量匹配多个字符
  * 没有方括号的字符类，有方括号的多个字符类，都是模式条目
* **\***：零个或多个，可有可无（贪心法则）
* **+**：一个或多个，起码有一个（贪心法则）
* **-**：零个或多个（非贪心，匹配尽可能短的字符，满足条件即可）
* **?**：零个或一个（非贪心，最多一个）
* **%n**：n 的范围为 1 到 9，表示匹配第 n 个符合条件的字符（第 n 个**捕获**）
* **%bxy**：x 和 y 是不一样的字符，读到 x 则 +1 读到 y 则 -1，y 到 0 为止（平衡，最外层）
  * **%b()** 匹配最外层的括号
* **%f[set]**：边境模式，匹配 set 之前的一个空串，且该空串的前一个字符不属于 set（非  Lua5.1 特性）

### 模式 Pattern

* 模式指模式条目的序列，就是一个普通字符串
* **^**：在模式开头，表示从字符串开头就得匹配（以xxx开头）
* **$**：在模式结尾，表示匹配到字符串结尾（以xxx结尾）
* 所以表示的是严格模式，除此之外，^ $ 出现在其它地方则只表示它们自身

### 捕获 Captures

* 捕获：模式内，用**小括号**括起来的子模式
  * 捕获可以理解为就是子模式
* 捕获成功的子字符串会保存起来，由左括号的次序编号（从 1 开始）
  * 模式 `"(a*(.)%w(%s*))"`
  * 匹配到字符串 `"a*(.)%w(%s*)"` ，编号为 1
  * `"."` 匹配到的保存到 2，`"%s*"` 保存到 3
* 例外：空的捕获 **()**，会保存字符串的位置（数字而非字符串）
  * 模式 `"()aa()"` 匹配字符串 `"flaaap"` 将产生两个捕获 3 和 5

### string.gsub

* `string.gsub (s, pattern, repl [, n])`
* 返回字符串副本，找到的模式，已被 repl 指定的字符串替换 n 次（返回被替换的次数）
* repl 可以是字符串，表和函数
  * 字符串：%0 表示整个匹配 %n 表示第 n 个匹配
  * 表：每次匹配时，把第1个捕获作为 key 查表，查不到则就用捕获作为结果；如果模式未指定捕获，则将整个匹配作为键（0 号捕获）
  * 函数：每次匹配时，将所有捕获传参；如果模式未指定捕获，则将整个匹配作为参数（不匹配则不调用？）
  * 如果表查找和函数调用返回的是数字或字符串，则做为替换字符串，否则不替换（返回 false 或 nil）