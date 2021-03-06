#  安装vim

```shell
yum -y install vim
```

# 一些问题

## 解决分隔符问题

```shell
:set fileformat=unix
```

# 命令历史

以:和/开头的命令都有历史纪录，可以首先键入:或/然后按上下箭头来选择某个历史

# 移动命令

```shell
w 光标移动到下一个单词首
e 光标移动到下一个单词尾
^ 移动到本行第一个非空白字符上。
0（数字0）移动到本行第一个字符上
$ 移动到行尾 3$ 移动到下面3行的行尾

gg 移动到文件头。
G  移动到文件尾。

Ctrl + f 向下滚动一屏
Ctrl + b 向上滚动一屏

f（find）命令也可以用于移动，fx将找到光标后第一个为x的字符，3fd将找到第三个为d的字符。
F 同f，反向查找。

跳到指定行，冒号+行号，回车，比如跳到240行就是 :240回车

Ctrl + e 向下滚动一行
Ctrl + y 向上滚动一行
Ctrl + d 向下滚动半屏
Ctrl + u 向上滚动半屏
```

# 拷贝剪切和粘贴

```shell
yy 拷贝当前行
nyy 拷贝当前后开始的n行，比如2yy拷贝当前行及其下一行。
:1,10 co 20 将1-10行插入到第20行之后。
:1,$ co $ 将整个文件复制一份并添加到文件尾部。

dd 剪切当前行（删除当前行）
ndd 剪切当前行之后的n行。利用p命令可以对剪切的内容进行粘贴
:1,10d 将1-10行剪切。利用p命令可将剪切后的内容进行粘贴。
:1,10 m 20 将第1-10行移动到第20行之后。

p在当前光标后粘贴,如果之前使用了yy命令来复制一行，那么就在当前行的下一行
粘贴。 
shift + p 在当前行前粘贴
u 撤销编辑
```



# 查找命令

```shell
/text 正向查找text，按n健查找下一个，按N健查找前一个。
?text 反向查找text，按n健查找下一个，按N健查找前一个。

1. vim中有一些特殊字符在查找时需要转义 .*[]^%/?~$
2. 查找很长的词，如果一个词很长，键入麻烦，可以将光标移动到该词上，按*或#键即可以该单词进行搜索，*相当于/搜索。而#命令相当于?搜索。

# 查找设置
:set nu 临时设置行号
:set ignorecase 忽略大小写的查找
:set noignorecase 不忽略大小写的查找
:set hlsearch 高亮搜索结果，所有结果都高亮显示，而不是只显示一个匹配。
:set nohlsearch 关闭高亮搜索显示
:set nohlsearch 关闭当前的高亮显示，如果再次搜索或者按下n或N键，则会再次高亮。
:set incsearch 逐步搜索模式，对当前键入的字符进行搜索而不必等待键入完成。
:set wrapscan 重新搜索，在搜索到文件头或尾时，返回继续搜索，默认开启。
```



# 插入命令

```shell
i 在当前位置生前插入
I 在当前行首插入
a 在当前位置后插入
A 在当前行尾插入
o 在当前行之后插入一行
O 在当前行之前插入一行
```

# 文件命令

## 打开多个文件

```shell
vim file1 file2 file3...
```

## 新窗口打开文件

```shell
:open file
```

# 退出命令

```shell
:wq 保存并退出
ZZ 保存并退出
:q! 强制退出并忽略所有更改
:e! 放弃所有修改，并打开原来文件
```