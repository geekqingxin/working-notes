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