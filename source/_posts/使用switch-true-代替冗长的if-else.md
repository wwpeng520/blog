---
title: 使用switch(true)代替冗长的if-else
date: 2018-08-05 16:32:13
tags:
- js
---

今天偶然看到 switch(true) 的写法，第一感觉是有点奇怪，细细一想，发觉这个写法很妙，先上代码：

```bash
var num = 25;
switch (true) {
    case num < 0:
        alert("Less than 0.");
        break;
    case num >= 0 && num <= 10:
        alert("Between 0 and 10.");
        break;}
    case num >= 10 && num <= 20:
        alert("Between 10 and 20.");
        break;
    default:
        alert("More than 20.");
}
```

<!-- more -->
一般我们的写法都是如下所示。我们根据 expression 的值等于哪个值是做相应的操作，然后跳出 switch 语句。(此时 expression 的值都是确定的几个值)。switch 语句中的每一种情形（case）的含义是：“如果表达式（expression）等于这个值（value），则执行后面的语句（statement）”。这里的表达式我们会习惯性的认为是一个确定的值，而遇到不确定值或者一个取值范围时好像不适用了（其实不是的）。

```bash
switch (expression) {
    case value: statement
        break;
    case value: statement
        break;
    case value: statement
        break;
    default: statement
}
```

此种情况下如果使用 if-else 语句，代码结构如下。代码结构与可读性比 switch 差些，如果判断条件更多时，switch 就更有优势了。从根本上说，switch 语句就是为了让开发人员免于编写像下面这样的代码。

```bash
if (expression === 'case1') {
    // do sth...
} else if (expression === 'case1') {
    // do sth...
} else if (expression === 'case1') {
    // do sth...
} else {
    // do sth...
}

// 最常见的是有区间范围的情况
if (num === 0) {
    // do sth...
} else if (num < 10) {
    // do sth...
} else if (num < 100) {
    // do sth...
} else {
    // do sth...
}
```

所以初学者（我也是）会这样认为：

- 当 expression 是一个确定的值，且 expression 的值存在较多可能时，选择 switch；
- 当需要对 expression 的值判断多个范围（而不是具体值）时，选用 if-else 语句。

回到我们开头的那段代码，这个例子之所以给 switch 语句传递表达式 true，是因为每个 case 值都可以返回一个布尔值。这样，每个 case 按照顺序被求值，直到找到匹配的值或者遇到 default 语句为止。

这样我们不难发现上面的结论都是错的，switch 语句也可使用范围判定的语句，这样就可以代替冗长的 if-else 语句了。
