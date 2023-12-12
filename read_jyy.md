.c -- --> .i -- --> .s -- --> .o -- --> .out

工具指南: man xxx

echo $x:
    $# 表示参数个数。
    $0 是脚本本身的名字。
    $1 是传递给该shell脚本的第一个参数。
    $2 是传递给该shell脚本的第二个参数。
    $@ 表示所有参数，并且所有参数都是独立的。
    $$ 是脚本运行的当前进程ID号。
    $? 是显示最后命令的退出状态，0表示没有错误，其他表示有错误。

man: help && read manual

gcc --help:
    -o <输出文件> ：指定输出文件
    -E 只执行编译预处理
    -S 将代码转换为汇编
    -c 只进行编译操作，不进行链接操作
    -Wall 显示警告信息
    -l库的首字母 链接gcc非默认库函数

vim:

tmux:
    ![Alt text](20231017170324.png)
    ![Alt text](20231017170339.png)
    ![Alt text](20231017170412.png)

objdump:
    | less

git:
    git clone: 1.(-54) --> apt-get install libssl-dev
               2.(443) --> git config --global --unset http.proxy
    git branch: 显示现在所有分支 (-m xxx): 重命名当前分支为xxx
    git checkout -b pa0: 创建新的分支pa0
    git status: 检查日志
    git diff: 和上一次提交的差异报告
    git add: 提交到暂存区
    git commit: 暂存区提交到库
    git log: 查看历史纪录

bash shell
    PATH --> .bashrc

ccache: (which gcc)

make:
    (time) make (-j?): 多少时间和多少个cpu

fzf: choose file
    vim $(fzf) vim打开fzf选择的文件

grep: g(all file) / re(正则式) / p(print)

find:
    find . -name "*.c" | xargs grep --color -nse '\<main\>' 在.C文件里面找到main （\分隔符，| xargs管道前一个输出作为下一个输入）

tree:
    ![Alt text](9M@2~13@MQ(%7DR)K%5D6%25XY3B8.png)
    ![Alt text](<LK8IW%L(`B(0_H%D}%KSBZV.png>)