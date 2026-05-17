---
title: Soft hyphen (0xAD) - 看不见的连字符
date: 2017-12-16T16:55:36+08:00
tags: ["soft-hyphen", "popup-dict"]
draft: false
---

今天在阅读一篇英文文章 [The Last JSON Spec](https://www.tbray.org/ongoing/When/201x/2017/12/14/RFC-8259-STD-90) 时，发现自己写的划词翻译工具 [popup-dict](https://github.com/bianjp/popup-dict) 对很多词无效，比如 "ir­ra­tional"。

细查之下，发现 "ir­ra­tional" 虽然看起来只有 10 个字符，但 "ir­ra­tional".length 却返回 12。第三个字符的 char code 是 173（16 进制表示就是 0xAD），这不是字母 "r"。整个字符串不匹配英文单词/合成词/句子的正则表达式，因此被 popup-dict 忽略。

<!--more-->

搜索发现，0xAD 是 [soft hyphen](https://en.wikipedia.org/wiki/Soft_hyphen)，中文大概翻译为软连字符。这个字符用于在排版时显式建议换行位置，一般不可见，但如果需要换行，就可以从这个字符处换行，并渲染为一个可见的连字符（渲染成什么样与语言有关，HTML 的 lang 属性）。主流浏览器基本都支持。

[测试发现](https://codepen.io/bianjp/pen/dJYVpO)，soft hyphen 对中文无效。相关的 [zero-width space](https://en.wikipedia.org/wiki/Zero-width_space)  及 [&lt;wbr&gt;](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/wbr) 也都对中文无效。所以也就对英文有点用了。

了解了 soft hyphen，那么划词翻译无效的问题也就好解决了，匹配前直接去掉就好了。

```python
text = text.replace('\xad', '')
```
