# <center>命令行与Git</center>  

### 1. 命令行
* ##### Unix（部分在Windows PowerShell可用）
`pwd`：当前目录  
`ls`: list，列出当前目录下的所有文件或文件夹  
`cd`: 移动到目录  
`rm`: 删除文件（或目录）  
`mkdir`: 创建目录  
`rmdir`: 删除目录（只能是空的，完全删除非空目录请用`rm -rf 目录名`）  
`cp`: copy，`copy 源 目的`；源中含有目录时使用选项-R  
`mv`: move，语法同cp  
`history`: 执行命令的历史记录  
`上下键`: 在历史命令中移动  
`!字母`: 执行最近的该字母开头的命令  
`q`: 退出  
`ps`: 列出当前所有进程  
`top`: 列出当前占用CPU最多的进程  
`ctrl-C`: 强制终止正在运行的程序  
`kill`: 杀死进程`kill <PID>`，-9表示强制杀死  
`sudo kill -9 <PID>`: 伪root杀死进程  
`cat > 文件名`: 输入此命令后向“文件名”中写入，用<kbh>ctrl-D</kbh>结束  
`more 文件名`: 分屏输出文件内容  
`tail 选项 文件名`: 输出文件尾部内容。`-f`表示循环运行`-n 行数`  
`wc`: wordcount，计算单词个数，输出为`行数 单词数 字符数`  
**grep**  
`grep 选项 文本 文件`: 查找文本在文件中出现的位置`-n`表示带着行号作输出  
`命令 | grep 文本`: 在命令的输出中搜索指定文本  
**重定向**  
`命令 < in > out`: 将文件in作为命令的输入，out作为文件的输出  
`命令1 | 命令2`: 将命令1的输出作为命令2的输入  
`命令 &`: 表示命令后台运行  
**设置环境变量**  
`set`: 显示所有环境变量  
`环境变量=XXX`: 将环境变量设置为XXX（临时性更改）  
`PATH=$PATH:XXX`: 向PATH中添加XXX（临时性更改）  
创建文件`.profile`，并添加上一条指令，即可实现永久性更改  
**网络配置**  
`ifconfig`: 显示当前所有网络连接  
`ifconfig 连接1 dowm`: 关闭连接1  
`ifconfig 连接1 up`: 启用连接1  
`ifconfig 连接1 add <iPv4Addr> netmask 255.255.255.0`: 在连接1中新建一个网段  
`ifconfig 连接1 delete <iPv4Addr>`: 在连接1中删除一个网段  
`ifconfig 连接1 <iPv4Addr>`: 修改网段  
`ping`  
`netstat`: 列举这台机器上的连接  
`netstst -l`: 列出这台机器上在做监听的  
`netstat -lt`: 仅列举TCP监听  
`netstat -lu`: 仅列举UDP监听  
`netstat -s`: 列出统计数据  
`netstat -p`: 列出进程信息  
`netstat -pc`: 持续更新进程信息，<kbh>ctrl-c</kbh>结束  
`netstat -r`: 列出路由表  
`netstat -ap | grep ssh`:   
`netstat -i`: 输出接口信息  
**网络调试**  
`nc -l -p <port>`: 在指定端口进行监听  
`telnet <iPv4Addr> <port>`: 连接到指定地址的指定端口  
通过上述步骤进行连接之后可以以文本形式进行测试  
**ssh连接**  
`ssh 地址 -p 端口`: 以ssh连接到指定地址（ssh端口为默认时可省略-p选项）；默认以当前机器上的用户名进行登录。  
`ssh 用户@地址`  
确认当前身份: 看命令提示符，`ifconfig`，`whoami`，`w`（所有用户信息）  
>上述登陆方式仍然是不安全的，下面为安全的登陆方式  
1. 创建新用户: `sudo adduser 名称`（可能为`newuser`，取决于unix版本），并设置密码  
2. 生成密钥对: `ssh-keygen -f ~/.ssh/filename`
3. 将filename.pub传输至服务器: `sftp 用户名@地址`连接ftp服务器，`put filename.pub`，上传文件`bye`退出ftp连接
4. 在服务器新建.ssh目录，将filename.pub移入其中；新建authorized_keys文件并写入filename.pub的内容
5. 在客户端使用密钥对登录: `ssh -i ~/.ssh/filename 用户名@地址`
6. 删除服务端的用户密码: （需要超级用户）`sudo passwd -d 名称`，此时该用户不能再通过密码进行登录
7. 简化登陆命令: `cp /etc/ssh/ssh_config ~/.ssh`复制文件，`mv ssh_config config`重命名，在文件中添加如下内容：
```ssh
Host 主机名
    user 用户名
    port 端口号
    hostname ipv4地址
    identityfile ~/.ssh/filename
```
此后只需要`ssh 主机名`即可登录。Windows下可用putty
* ##### Windows（CMD）
`dir`: 同`ls`  
`cd`  
`del`: 同`rm`  
`mkdir`  
`rd` :同`rmdir`  
`copy`: 同`cp`  
`move`: 同`mv`  

### 2. 版本管理

* ##### 版本控制系统（VCS, Version Control System）及其分类
1. 人工式 VCS
2. 本地式 LVCS（如RCS）
数据库位于本地
3. 集中式 CVCS（常用SVN）
数据库位于一台单独的服务器上，可控性高，流畅性低，故障成本高
4. 分布式 DVCS（常用Git）
每个客户端都有一份完整的数据库拷贝
* ##### 分支模型（Branching Model）/ 工作流（Workflow）：产品级分支模型
    * 常驻分支：development(from master), production(master)
    * 活动分支：feature(from development), hotfix(from master), release(from development)
    * 涉及四种环境：开发环境、测试环境、预发布环境、生产环境
* ##### Git
    * 帮助命令
    `git help <command>`
    `git <command> -h`
    `git <command> --help`
    `man git-<command>`
    * 配置信息
    `git config --global user.name "用户名"`  
    `git config --global user.email 邮箱`  
        > 配置级别  
        > \-\-local 【默认，高优先级】，只影响本地仓库  
        > \-\-global 【中优先级】，影响所有当前用户的git仓库  
        > \-\-system 【低优先级】，影响全系统的git仓库  
    * 初始化库
    `git init`: 初始化目录（创建.git文件夹）
    `git status`: 查看当前仓库信息  
        > git-status对状态的跟踪  
        > 内容状态：工作目录，暂存区，提交区（，stash区）  
        > 文件状态：未跟踪，已跟踪  
    * 添加文件到暂存区  
    `git add`: 添加文件到暂存区，并进行跟踪  
        > .gitignore文件保存了add命令忽略的文件  
    * 删除文件  
    `git rm --cached`: 仅从暂存区删除  
    `git rm`: 从暂存区和工作目录同时删除  
    `git rm $(git ls-files --deleted)`: 删除所有被跟踪，但在工作目录下被删除的文件  
    * 提交文件  
    `git commit -m '备注'`: 暂存区->提交区  
    `git commit -a -m '备注'`: 工作目录->提交区  
    `git log`: 输出提交信息  
    `git log --oneline`: 简化内容输出
    * 设置命令别名
    `git config --global alias.shortname <fullcommand>`  
    这一语句使得`git shortname`与`git fullcommand`等价   
    * 显示版本差异  
    `git diff`: 显示工作目录与暂存区的差异  
    `git diff -cached <reference>`: 暂存区与某次提交的差异，默认为HEAD（当前提交）  
    `git diff <reference>`: 工作目录与某次提交之间的差异  
    `git diff <reference1> <reference2>`: 两次提交之间的差异  
    * 撤销本地（工作目录）修改  
    `git checkout -- <filename>`: 将文件内容从暂存区复制到工作目录  
    * 撤销暂存区内容  
    `git reset HEAD <filename>`: 将文件从上次提交复制到暂存区  
    * 撤销全部改动  
    在上一条的基础上执行以下命令  
    `git checkout HEAD -- <filename>`  
    * 分支操作  
    `git branch <name>`: 创建分支，在当前HEAD位置新建一个引用  
    `git branch -d <name>`: 删除分支  
    `git branch -v`: 显示所有分支的信息  
    `git checkout <branchname>`: 切换HEAD到指定分支  
    `git checkout -b <branchname>`: 创建新分支并切换HEAD到这一分支  
    `git checkout <reference>`: 将HEAD切换到指定提交  
    `git checkout -`: 回到上一个分支  
    `git reset --soft <reference>`: 将当前分支、HEAD指针退回至指定提交，保留暂存区、工作目录内容不变  
    `git reset --mixed <reference>`: 同soft，但复制提交内容到暂存区  
    `git reset --hard <reference>`: 同mixed，但复制内容到工作目录  
    `git reflog`: 按照经过各个commit的倒序输出所有commit  
    `A^`: A的上次提交，与`A~1`等价  
    `A~n`: A之前的第n次提交  
    `git stash`: 保存当前的工作目录与暂存区状态，并返回到干净的工作空间（相当于一个栈）  
    `git stash save '标识'`  
    `git stash list`: 查看所有stash  
    `git stash apply <stashsave>`  
    `git stash drop <stashname>`: 删除指定stash记录  
    `git stash pop` = `git stash apply` + `git stash drop`  
    `git merge <branch1> <branch2 HEAD可省略>`: 合并分支1到分支2，产生一次新的提交，并移动分支2；当1、2位于同一条线上且2在1之后时，默认仅移动分支1，不产生新的提交  
    `git cat-file -p HEAD`: 显示某个对象的具体信息  
    `git merge <branch1> <branch2> --no-ff`: 强制分支合并产生新的提交  
    `git rebase master`: “变基”，将HEAD与共同节点之间的（含HEAD）提交记录在master上“重演”，并移动HEAD到最新的节点上【不要对master进行rebase！！】  
    `git rebase --onto master <reference>`: 将指定提交（不含）与HEAD（含）之间的提交记录在master上“重演”  
    `git tag 名称 <reference>`: 为指定对象打上指定标签，后续可以使用这一标签代表这一对象  
    * 远程操作  
    `git init ~/git-server --bare`: 将指定目录初始化为一个裸仓库  
    `git push 仓库 分支`: 提交本地历史到远程（提交历史的复制）  
    `git remote add 别名 仓库地址`: 为仓库起别名  
    `git fetch 仓库`: 获取远程仓库的提交历史  
    `git pull` = `git fetch` + `git merge`  
    `git clone 仓库地址 <?>`: 克隆一个远程仓库作为一个本地仓库  
    `git config --global http.sslVerify false`

