### Git

#### 工作常用

```git
// 每修改完push流程
git status // 查看修改了哪些文件
git add XX
git commit -m "v70.9:remove if(bWait) in QueueSubmit_VK"
git checkout -b "damondai" // 新建分支并切换过来
git tag -a v70.9
git push --set-upstream origin damondai
git push --tag


// push完可以删除本地临时分支
git branch -d damondai
```

#### 全局设置

`git config --global user.name xxx`：设置全局用户名，信息记录在`~/.gitconfig`文件中
`git config --global user.email xxx@xxx.com`：设置全局邮箱地址，信息记录在`~/.gitconfig`文件中
`git init`：将当前目录配置成git仓库，信息记录在隐藏的`.git`文件夹中

#### 个人常用

`git add XX` ：将XX文件添加到暂存区
`git commit -m "给自己看的备注信息"`：将暂存区的内容提交到当前分支
`git status`：查看仓库状态
`git log`：查看当前分支的所有版本
`git push -u (第一次需要-u以后不需要) `：将当前分支推送到远程仓库

`git push --force`：在本地进行了一些修改，还未提交，想要强制拉取覆盖本地修改

`git clone --recursive git@git.acwing.com:xxx/XXX.git`：将远程仓库XXX下载到当前目录下
`git branch`：查看所有分支和当前所处分支

#### 查看命令

`git diff XX`：查看XX文件相对于暂存区修改了哪些内容
`git status`：查看仓库状态
`git log`：查看当前分支的所有版本
`git log --pretty=oneline`：用一行来显示
`git reflog`：查看HEAD指针的移动历史（包括被回滚的版本）
`git branch`：查看所有分支和当前所处分支
`git pull `：将远程仓库的当前分支与本地仓库的当前分支合并

- `git pull origin branch_name`：将远程仓库的`branch_name`分支与本地仓库的当前分支合并

#### 删除命令

`git rm --cached XX`：将文件从仓库索引目录中删掉，不希望管理这个文件
`git restore --staged xx`：==将`xx`从暂存区里移除==
`git checkout -- XX或git restore XX`：==将`XX`文件尚未加入暂存区的修改全部撤销==

#### 代码回滚

`git reset --hard HEAD^` 或`git reset --hard HEAD~` ：将代码库回滚到上一个版本
`git reset --hard HEAD^^`：往上回滚两次，以此类推
`git reset --hard HEAD~100`：往上回滚100个版本
`git reset --hard 版本号`：回滚到某一特定版本

#### 远程仓库

`git remote add origin git@git.acwing.com:xxx/XXX.git`：将本地仓库关联到远程仓库
`git push -u (第一次需要-u以后不需要)` ：将当前分支推送到远程仓库
`git push origin branch_name`：将本地的某个分支推送到远程仓库
`git clone git@git.acwing.com:xxx/XXX.git`：将远程仓库`XXX`下载到当前目录下
`git push --set-upstream origin branch_name`：设置本地的`branch_name`分支对应远程仓库的`branch_name`分支
`git push -d origin branch_name`：删除远程仓库的`branch_name`分支
`git checkout -t origin/branch_name`: 将远程的`branch_name`分支拉取到本地
`git pull`：将远程仓库的当前分支与本地仓库的当前分支合并
`git pull origin branch_name`：将远程仓库的`branch_name`分支与本地仓库的当前分支合并
`git branch --set-upstream-to=origin/branch_name1 branch_name2`：将远程的`branch_name1`分支与本地的`branch_name2`分支对应

#### 分支命令

`git branch branch_name`：创建新分支
`git branch`：查看所有分支和当前所处分支
`git checkout -b branch_name`：创建并切换到`branch_name`这个分支
`git checkout branch_name`：切换到`branch_name`这个分支
`git merge branch_name`：将分支`branch_name`合并到当前分支上
`git branch -d branch_name`：删除本地仓库的`branch_name`分支
`git push --set-upstream origin branch_name`：设置本地的`branch_name`分支对应远程仓库的`branch_name`分支
`git push -d origin branch_name`：删除远程仓库的`branch_name`分支
`git checkout -t origin/branch_name` 将远程的`branch_name`分支拉取到本地
`git pull `：将远程仓库的当前分支与本地仓库的当前分支合并
`git pull origin branch_name`：将远程仓库的`branch_name`分支与本地仓库的当前分支合并
`git branch --set-upstream-to=origin/branch_name1 branch_name2`：将远程的`branch_name1`分支与本地的`branch_name2`分支对应

#### stash暂存

`git stash`：将工作区和暂存区中尚未提交的修改存入栈中
`git stash apply`：将栈顶存储的修改恢复到当前分支，但不删除栈顶元素
`git stash drop`：删除栈顶存储的修改
`git stash pop`：将栈顶存储的修改恢复到当前分支，同时删除栈顶元素
`git stash list`：查看栈中所有元素

### WinDbg

`g`：跳过当前中断，继续运行

`.ecxr`：切换到发生异常的线程中，即异常上下文

`kn/kv/kp`：查看当前线程的函数调用堆栈

`lm`：查看exe或dll二进制文件的详细信息，比如路径、时间戳等，例如`lm vm libcurl*`

`.reload`：加载pdb文件，例如`.reload /f libcurl.dll` 强制加载libcurl.dll库的pdb文件

`~`：查看当前进程的所有线程信息

`~ns`：切换到n号线程中

`!analyze`：详细分析当前异常，完整的命令为：`!analyze -v`

`.dump`：导出dump文件，`dump /ma D:\20230415.dmp`

`bp/bl/bc`：添加断点、罗列所有断点、清除断点

`r`：查看当前线程所有寄存器的值

`.effmach x86`：从任务管理器中导出的32位程序的dump文件需要手动切换到32位上下文

### vim

`G`：光标移动到最后一行

`:n` 或 nG：n为数字，光标移动到第n行

`gg`：光标移动到第一行，相当于1G

`/word`：向光标之下寻找第一个值为word的字符串

`?word`：向光标之上寻找第一个值为word的字符串

`n`：重复前一个查找操作

`N`：反向重复前一个查找操作

`:n1,n2s/word1/word2/g`：n1与n2为数字，在第n1行与n2行之间寻找word1这个字符串，并将该字符串替换为word2

`:1,$s/word1/word2/g`：将全文的word1替换为word2

`:1,$s/word1/word2/gc`：将全文的word1替换为word2，且在替换前要求用户确认

`v`：选中文本

`d`：删除选中的文本

`dd`: 删除当前行

`y`：复制选中的文本

`yy`: 复制当前行

`p`: 将复制的数据在光标的下一行/下一个位置粘贴

`u`：撤销

`Ctrl + r`：取消撤销

`大于号 >`：将选中的文本整体向右缩进一次

`小于号 <`：将选中的文本整体向左缩进一次

`:set paste` ：设置成粘贴模式，取消代码自动缩进

`:set nopaste`： 取消粘贴模式，开启代码自动缩进

`:set nu`： 显示行号

`:set nonu`： 隐藏行号

`gg=G`：将全文代码格式化

`:noh`： 关闭查找关键词高亮

### SSH

`ssh user@hostname`

#### 配置文件

创建文件 `~/.ssh/config`

然后在文件中输入：

```
Host myserver1
 HostName IP地址或域名
 User 用户名

Host myserver2
 HostName IP地址或域名
 User 用户名
```

之后再使用服务器时，可以直接使用别名`myserver1`、`myserver2`

#### 密钥登录

创建密钥：

`ssh-keygen`

然后一直回车即可。

执行结束后，`~/.ssh/`目录下会多两个文件：

`id_rsa`：私钥
`id_rsa.pub`：公钥
之后想免密码登录哪个服务器，就将公钥传给哪个服务器即可。

例如，想免密登录`myserver`服务器。则将公钥中的内容，复制到`myserver`中的`~/.ssh/authorized_keys`文件里即可。

也可以**使用命令一键添加公钥**：

`ssh-copy-id myserver`

### SCP

`scp 211:/home/djf/pmem1/a.txt .`：将211的`/home/djf/pmem1/a.txt`复制到当前目录下

`scp .\b.txt 211:/home/djf/pmem1/tmp`：将当前目录下的b.txt文件上传到211的`/home/djf/pmem1/tmp`目录下

`scp -r 211:/home/djf/pmem1 .`：将211目录`/home/djf/pmem1`下的所有文件复制到当前目录

### Linux

分割文件：`split -b 1M api_dump.txt smallfile`
