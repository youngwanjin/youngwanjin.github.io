---
layout: wiki
title: vim 命令大全
categories: [wiki]
description: 记录vim常用命令
keywords: vim
---

----

vim一共有4个模式：

正常模式 (Normal-mode) 
插入模式 (Insert-mode)
命令模式 (Command-mode)
可视模式 (Visual-mode)
正常模式

启动vim后默认处于正常模式。不论位于什么模式，按下<Esc>键(有时需要按两下）都会进入正常模式。

插入模式

在正常模式中按下i, I, a, A等键，会进入插入模式。现在只用记住按i键会进行插入模式。在插入模式中，击键时会写入相应的字符。

命令模式

在正常模式中，按下:（英文冒号）键，会进入命令模式。在命令模式中可以执行一些输入并执行一些vim或插件提供的指令，就像在shell里一样。这些指令包括设置环境、文件操作、调用某个功能等等。

常用的命令有：q（退出）、q!（强制退出）、w（保存）、wq（保存并退出）。

可视模式

在正常模式中按下v, V, <Ctrl>+v，可以进入可视模式。可视模式中的操作有点像拿鼠标进行操作，选择文本的时候有一种鼠标选择的即视感，有时候会很方便。


### 1. 进入vim命令

 1. **vi/vim filename** : 打开或新建文件，将光标置于第一行行首

 2. **vi +n filename**: 打开文件，并将将光标置于第n行行首。

 3. **vi + filename**: 打开文件，将光标置于最后一行行首

 4.  **vi -r filename** ：在上次正在使用vi编辑时发生系统崩溃，恢复filename 

 5. **vi filename1 filename2** ：打开多个文件，依次进行编辑 。

    > **:n** 切换到下一个文件 (n=next)
    >**:N** 切换到上一个文件 



### 2. 光标移动命令

1. **h**: 光标左移一个字符
2. **l**: 光标右移一个字符
3. **space**: 光标右移一个字符
4. **backspace**：光标左移一个字符
5. **k/Ctrl+p**: 光标上移两行（垂直上移）
6. **j/Ctrl+n**: 光标下移两行（垂直下移）
7. **Enter**: 光标下移一行（自动到行首）
8. **w/W**：光标右移一个单词（包括标点）至字首
9. **b/B**：光标左移一个单词（包括标点）至字首
10. **e/E** ：光标右移一个单词（包括标点）至字尾
11. **) **: 光标移至句尾
12.  **( **：光标移至句首 
13. **}**：光标移至段落开头 
14.  **{**：光标移至段落结尾 
15.  **nG**：光标移至第n行首 
16.  **n$**：光标移至第n行尾 
17. **n+**：光标下移n行
18. **n-**：光标上移n行
19. **H** ：光标移至屏幕顶行
20. **M** ：光标移至屏幕中间行
21. **L** ：光标移至屏幕最后行
22. **0**：（零）光标移至当前行首
23. **$**：光标移至当前行尾
24. **^**:光标移至当前行首



### 3. 屏幕滚动命令

1. **Ctrl+u**：向文件首翻半屏
2. **Ctrl+d**：向文件尾翻半屏
3. **Ctrl+f**：向文件尾翻一屏
4. **Ctrl＋b**：向文件首翻一屏
5. **nz**：将第n行滚至屏幕顶部，不指定n时将当前行滚至屏幕顶部



### 4. 插入文本命令

1. **i** ：在光标前插入字符
2. **I** ：在当前行首插入字符
3. **a**：光标后插入字符
4. **A**：在当前行尾字符
5. **o**：在当前行之下新开一行插入字符
6. **O**：在当前行之上新开一行
7. **r**：替换当前光标所指字符
8. **R**：替换当前字符及其后的字符，直至按ESC键
9. **s**：从当前光标位置处开始，以输入的文本替代指定数目的字符
10. **S**：删除指定数目的行，并以所输入文本代替之
11. **ncw/nCW**：修改指定数目的字符
12. **nCC**：修改指定数目的行



### 5. 删除命令

1. **ndw/ndW**：删除光标处开始及其后的n-1个字

2. **do**：删至行首

3. **d$**：删至行尾

4. **ndd**：删除当前行及其后n-1行

5. **x/X**：删除一个字符，x删除光标后的，而X删除光标前的

6. **Ctrl+u**：删除输入方式下所输入的文本



### 6. 搜索及替换命令

1. **/pattern**：从光标开始处向文件尾搜索与pattern匹配的字符（向下搜索）

2. **?pattern**：从光标开始处向文件首搜索与pattern匹配的字符（向上搜索）

3. **n**：在同一方向重复上一次搜索命令

4. **N**：在反方向上重复上一次搜索命令

5. **:s/p1/p2/g**：将当前行中所有p1均用p2替代

6. **:n1,n2s/p1/p2/g**：将第n1至n2行中所有p1均用p2替代

7. **:g/p1/s//p2/g**：将文件中所有p1均用p2替换



### 7. 选项设置

1. **all**：列出所有选项设置情况

2. **term**：设置终端类型

3. **ignorance**：在搜索中忽略大小写

4. **list**：显示制表位(Ctrl+I)和行尾标志（$)

5. **number**：显示行号

6. **report**：显示由面向行的命令修改过的数目

7. **terse**：显示简短的警告信息

8. **warn**：在转到别的文件时若没保存当前文件则显示NO write信息

9. **nomagic**：允许在搜索模式中，使用前面不带“\”的特殊字符

10. **nowrapscan**：禁止vi在搜索到达文件两端时，又从另一端开始

11. **mesg**：允许vi显示其他用户用write写到自己终端上的信息



### 8. 命令模式

1. **:n1,n2 co n3**：将n1行到n2行之间的内容拷贝到第n3行下

2. **:n1,n2 m n3**：将n1行到n2行之间的内容移至到第n3行下

3. **:n1,n2 d** ：将n1行到n2行之间的内容删除

4. ** :w **：保存当前文件

5. **:e filename**：打开文件filename进行编辑

6. **:x**：保存当前文件并退出

7. **:q**：退出vi

8. **:q!**：不保存文件并退出vi

9. **:!command**：执行shell命令command

10. **:n1,n2 w!command**：将文件中n1行至n2行的内容作为command的输入并执行之，若不指定n1，n2，则表示将整个文件内容作为command的输入

11. **:r!command**：将命令command的输出结果放到当前行



### 9. 寄存器操作

1. **"?nyy**：将当前行及其下n行的内容保存到寄存器？中，其中?为一个字母，n为一个数字

2. **"?nyw**：将当前行及其下n个字保存到寄存器？中，其中?为一个字母，n为一个数字

3. **"?nyl**：将当前行及其下n个字符保存到寄存器？中，其中?为一个字母，n为一个数字

4. **"?p**：取出寄存器？中的内容并将其放到光标位置处。这里？可以是一个字母，也可以是一个数字

5. **ndd**：将当前行及其下共n行文本删除，并将所删内容放到1号删除寄存器中



### 10. vim 模式介绍

* 正常模式 (Normal-mode) 

* 插入模式 (Insert-mode)

* 命令模式 (Command-mode)

* 可视模式 (Visual-mode)

  

#### (1). **正常模式** 

启动vim后默认处于正常模式。不论位于什么模式，按下<Esc>键(有时需要按两下）都会进入正常模式

#### (2). 插入模式

在正常模式中按下i, I, a, A等键，会进入插入模式; 在插入模式中，击键时会写入相应的字符

#### (3). 命令模式

在正常模式中，按下:（英文冒号）键，会进入命令模式。在命令模式中可以执行一些输入并执行一些vim或插件提供的指令，就像在shell里一样。这些指令包括设置环境、文件操作、调用某个功能等

#### (4). 可视模式

在正常模式中按下v, V, <Ctrl>+v，可以进入可视模式, 可视模式中的操作有点像拿鼠标进行操作，选择文本的时候有一种鼠标选择的即视感，有时候会很方便。



----

[Ref1]( http://pizn.github.io/2012/03/03/vim-commonly-used-command.html )

[Ref2]( https://www.jianshu.com/p/e4230122610b )
