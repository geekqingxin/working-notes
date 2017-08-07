# 1. 基本命令

## 1.1 初始化仓库

    …or create a new repository on the command line

    echo "# working-notes" >> README.md
    git init
    git add README.md
    git commit -m "first commit"
    git remote add origin https://github.com/liuyf8688/working-notes.git
    git push -u origin master

    …or push an existing repository from the command line

    git remote add origin https://github.com/liuyf8688/working-notes.git
    git push -u origin master

## 1.2 撤消commit

    git log

    git reset --hard commit_id

## 1.3 修改已提交的注释

    git commit --amend

# XXX. 问题

## XXX.1 解决Windows命令行乱码问题

    # 1. 让git支持utf-8编码

    git config --global core.quotepath false
    
    git config --global gui.encoding utf-8

    git config --global i18n.commit.encoding utf-8

    git config --global i18n.logoutputencoding utf-8

    export LESSCHARSET=utf-8

    # 2. 修改Git\mingw64\share\git\completion\git-completion.bash文件

    # 在方便末尾处添加一行
    alias ls="ls --show-control-chars --color"