title: JavaScript 学习之正则篇
date: 2017-01-07 11:40:16
tags:
- JavaScript
---
> 正则表达式可以极大的提高复杂文本分析的效率，快速匹配出复杂的字符串。

<!--more-->
#### 基础语法
| 字符 | 描述 |
| :--: | :--- |
| `\` | 将下一个字符标记为一个特殊字符、或一个原义字符。例如，“n”匹配字符“n”。“\n”匹配一个换行符。序列“\”匹配“\”而“(”则匹配“(”。|
| `^` | 匹配输入字符串的开始位置。 |
| `$` | 匹配输入字符串的结束位置。 |
| `{n}` | n是一个非负整数。匹配确定的n次。例如，“o{2}”不能匹配“Bob”中的“o”，但是能匹配“food”中的两个o。|
| `{n,}` | n是一个非负整数。至少匹配n次。例如，“o{2,}”不能匹配“Bob”中的“o”，但能匹配“foooood”中的所有o。“o{1,}”等价于“o+”。“o{0,}”则等价于“o*”。|
| `{n,m}` | m和n均为非负整数，其中n<=m。最少匹配n次且最多匹配m次。例如，“o{1,3}”将匹配“fooooood”中的前三个o。“o{0,1}”等价于“o?”。请注意在逗号和两个数之间不能有空格。|
| `*` | 匹配前面的子表达式零次或多次。例如，zo*能匹配“z”、“zo”以及“zoo”。*等价于{0,}。|
| `+` | 匹配前面的子表达式一次或多次。例如，“zo+”能匹配“zo”以及“zoo”，但不能匹配“z”。+等价于{1,}。|
| `?` | 匹配前面的子表达式零次或一次。例如，“do(es)?”可以匹配“do”或“does”中的“do”。?等价于{0,1}。|
| `?` | 当该字符紧跟在任何一个其他限制符（*,+,?，{n}，{n,}，{n,m}）后面时，匹配模式是非贪婪的。非贪婪模式尽可能少的匹配所搜索的字符串，而默认的贪婪模式则尽可能多的匹配所搜索的字符串。例如，对于字符串“oooo”，“o+?”将匹配单个“o”，而“o+”将匹配所有“o”。|
| `.` | 匹配除“\n”之外的任何单个字符。要匹配包括“\n”在内的任何字符，请使用像“(.｜\n)”的模式。|
| `(pattern)` | 匹配pattern并获取这一匹配的子字符串。该子字符串用于向后引用。要匹配圆括号字符，请使用 \\( 和 \\)。|
| `x｜y` | 匹配x或y。例如，“z｜food”能匹配“z”或“food”。“(z｜f)ood”则匹配“zood”或“food”。 |
| `[xyz]` | 字符集合（character class）。匹配所包含的任意一个字符。例如，“[abc]”可以匹配“plain”中的“a”。其中特殊字符仅有反斜线\保持特殊含义，用于转义字符。其它特殊字符如星号、加号、各种括号等均作为普通字符。脱字符^如果出现在首位则表示负值字符集合；如果出现在字符串中间就仅作为普通字符。连字符 - 如果出现在字符串中间表示字符范围描述；如果如果出现在首位则仅作为普通字符。|
| `[^xyz]` | 排除型（negate）字符集合。匹配未列出的任意字符。例如，“[^abc]”可以匹配“plain”中的“plin”。 |
| `[^a-z]` | 排除型的字符范围。匹配任何不在指定范围内的任意字符。例如，“[^a-z]”可以匹配任何不在“a”到“z”范围内的任意字符。 |
| `\w` | 匹配包括下划线的任何单词字符。等价于 [a-zA-Z0-9_]。|
| `\W` | 匹配任何非单词字符。等价于 [^a-zA-Z0-9_]。|
| `\s` | 匹配任何空白字符，包括空格、制表符、换页符等等。等价于 [ \f\n\r\t\v]。|
| `\S` | 匹配任何非空白字符。等价于 [^ \f\n\r\t\v]。|
| `\d` | 匹配一个数字字符。等价于 [0-9]。|
| `\D` | 匹配一个非数字字符。等价于 [^0-9]。|
| `\b` | 匹配一个单词边界，也就是指单词和空格间的位置。例如， ‘er\b’ 可以匹配”never” 中的 ‘er’，但不能匹配 “verb” 中的 ‘er’。|
| `\B` | 匹配非单词边界。’er\B’ 能匹配 “verb” 中的 ‘er’，但不能匹配 “never” 中的 ‘er’。|
| `\<` | 表示词首。 例如\<abc表示以abc为首的词。|
| `\>` | 表示词尾。 例如abc\>表示以abc结尾的词。|

#### 优先级
优先级为从上到下从左到右，依次降低：

| 运算符 | 说明 |
| :---- | :---: |
| '\' | 转义符 |
| '(), (?:), (?=), []' | 括号和中括号 |
| '*、+、?、{n}、{n,}、{n,m}' | 限定符 |
| '^、$、\任何元字符' | 定位点和序列 |
| '｜' | 选择 |

#### 创建正则表达式
- 字面量
```javascript
var expression = /pattern/flags ;
//pattern 是要匹配的字符串模式
//flags用来标记正则表达式的行为: i 不区分大小写；g 表示全局搜索 ；m 表示多行模式
var reg = /ab/i; //表示匹配 字符串 'ab' 不区分大小写
```
- RegExp对象的构造函数
```javascript
var expression = new RegExp(pattern,flags)
//flags 和直接量语法一致
//pattern 可以是字符串模式，也可以是一个标准的正则表达式，后者必须省略 flags
//两种写法均可
var reg = new RegExp('ab','i')  
var reg = new RegExp(/ab/i)
```

#### RegExp的方法
- exec()
```
    该方法对一个指定的字符串执行一个正则表达式，执行正则表达式的匹配，返回一个数组。没有匹配到则返回 null，返回的数组是Array的实例，而且返回值还包含另外2个属性：index： 匹配到的字符位于原始字符串的基于0的索引值 和 input： 原始字符串`。
```
```javascript
    var myRe = /ab*/g;
    var str = 'abbcdefabh';
    var oo = myRe.exec(str);
    console.log(oo);
    //[ 'abb', index: 0, input: 'abbcdefabh' ]
    oo = myRe.exec(str);
    console.log(oo);
    //[ 'ab', index: 7, input: 'abbcdefabh' ]
```
>当正则表达式使用 "g" 标志时，可以多次执行 exec 方法来查找同一个字符串中的成功匹配。当你这样做时，查找将从正则表达式的 lastIndex 属性指定的位置开始。
不加"g" 标志的时候，每次都是从 0 开始，所以各种属性也不会改变

- test()
```
    接收一个字符串参数，regexObj.exec(str),如果包含正则表达式的一个匹配结果返回true，否则false
```
```javascript
    var pattern = /java/i;
    var result = pattern.test('javascript');
    console.log(result); //true
```

- toString()
```
    RegExp 对象覆盖了 Object 对象的 toString() 方法，并没有继承 Object.prototype.toString()。对于 RegExp 对象，toString 方法返回一个该正则表达式的字面量。
```
```javascript
    myExp = new RegExp("a+b+c");
    console.log(myExp.toString());  // 结果: /a+b+c/

    foo = new RegExp("bar", "g");
    console.log(foo.toString());   // 结果: /bar/g
```

#### 用于模式匹配的String方法
- String.search()
```
    参数：一个正则表达式。
    返回：它返回匹配到的位置索引，或者在失败时返回-1。
```
```javascript
    //查找连续2个数字的位置
    'a1b2c33d4'.search(/(\d){2}/)
    // 5
```

- String.replace()
```
    一个在字符串中执行查找匹配的String方法，并且使用替换字符串替换掉匹配到的子字符串。    
```
```javascript
    '121212'.replace(/1/g,',')
    //",2,2,2"
```

- String.match()
```
    参数：一个正则表达式。一个在字符串中执行查找匹配的String方法，它返回一个数组或者在未匹配到时返回null。
    设置g则返回所有匹配结果，否则数组的第一个元素是匹配的字符串，剩下的是圆括号中的子表达式
```
```javascript
    var matchResult = '121212'.match(/1/g)
    console.log(matchResult);
    //["1", "1", "1"]
    matchResult = '121212'.match(/1/)
    console.log(matchResult);
    //[ '1', index: 0, input: '121212' ]
```

- String.split()
```
    一个使用正则表达式或者一个固定字符串分隔一个字符串，并将分隔后的子字符串存储到数组中
```
```javascript
    //以数字分割字符串
    'a1b2c33d4'.split(/\d*/)
    //["a", "b", "c", "d", ""]
```

#### 实例
[常用javascript正则表达式](http://www.jianshu.com/p/45cccb9ca3e9)
