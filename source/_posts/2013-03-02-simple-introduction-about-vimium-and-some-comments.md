---
layout: post
title: simple introduction about vimium and some comments
tag: productivity
categories: ["productivity"]
---

Vimium 是Chrome下的一个插件，可以让你用类似vim的方式用全键盘的方式浏览网页。其所有快捷键如下(web页面下按？也可以查看):

Vimium Help

```bash
vigating the page
j, <c-e>: Scroll down
k, <c-y>: Scroll up
h:the Scroll left
l: Scroll right
gg: Scroll to the top of the page
G:Scroll Scroll to the bottom of the page
zH: Scroll all the way to the left
zL: Scroll all the way to the right
d: Scroll a page down
u: Scroll a page up
r: Reload the page
gs: View page source
yy: Copy the current URL to the clipboard
yf: Copy a link URL to the clipboard
post: Open the clipboard's URL in the current tab
P: Open the clipboard's URL in a new tab
gu: Go up the URL hierarchy
i: Enter insert mode
gi: Focus the first (or n-th) text box on the page
f: Open a link in the current tab
F: Open a link in a new tab
<a-f>: Open multiple links in a new tab
o: Open URL, bookmark, or history entry
O: Open URL, bookmark, history entry, in a new tab
T: Search through your orpen tabs
b: Open a bookmark
B: Open a bookmark in a new tab
[[: Follow the link labeled previous or <
]]: Follow the link labeled next or >
gf: Cycle forward to the next frame on the page
```

查找:

```bash
/: Enter find mode
n: Cycle forward to the next find match
N: Cycle backward to the previous find match
Navigating history
H: Go back in history
L: Go forward in history
Manipulating tabs
K, gt: Go one tab right
J, gT: Go one tab left
g0: Go to the first tab
g$: Go to the last tab
t: Create new tab
x: Close current tab
X: Restore closed tab
Miscellaneous
?: Show help
```

觉得比较有用的几个快捷键:

- 滚到网页底部和顶部: 和vim一样 G，gg
- 向下(上)移一页: d/u (vim 得加 ctrl键)
- 拷贝当前地址栏url: yy, chrome对应的快捷键是command + alt + c(- 但是最新的chrome不知道为什么和develop工具的快捷键冲突了)
- 拷贝网页中某链接到剪贴板中:yf 之后选择你需要拷贝的链接，用 p键在当前页面打开，大P在新的tab中打开
- 打开书签、地址或者历史记录: o/O在当前页面/新页面打开，如果只想打开书签中记录的url，可以用b/B
- 查找模式: / 后输入搜索关键字，用n/N在搜索结果中向前或者向后查询，用的不多……，比浏览器默认的搜索稍微方便点
- 查找tab: T，输入关键字，跳转到相关的tab，这个很方便
- tab间移动: K/gt 移到右边的tab， J/gT移到左边的tab。不是很方便，一般都用浏览器默认的快捷键，shift+tab或者是alt+command+方向键
- 创建/关闭/恢复前一个关闭掉的tab: t/x/X, 用的也不多，浏览器默认的快捷键用习惯了

使用vimium有好处也有不好的地方，我自己的使用感觉如下:

好处:

系统无关，在任何系统下只要是使用chrome浏览器，不需要再去记忆快捷键;
对于键盘控来说，无疑是一个好东西，比默认的浏览器快捷键要强大，基本可以免去触摸板和鼠标操作；

坏处:

页面如果没有load, vimium也不会被加载，所有的快捷键也都失效了，这个时候还是需要浏览器默认的快捷键才可以继续撸键盘；
有时候vimium的js会和某些页面的js冲突，比如googlereader页面，而且在浏览器上调试的时候如果遇到js冲突还是挺讨厌的，
这种情况就得禁用vimium;
Firefox下据说有更强大的工具，叫做vimoperator,听说功能更加强大，不过我基本只用，所以没怎么尝试过。Anyway，如果你只相信啪啪啪的，可以试试这个插件。
