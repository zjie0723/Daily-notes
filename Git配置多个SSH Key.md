# Git配置多个SSH Key

打开git bash：

### 添加公钥

##### 生成第一个SSH Key

`ssh-keygen -t rsa -C "zhangjie123@qq.com” -f ~/.ssh/gitlab_rsa`

##### 生成第二个SSH Key

`ssh-keygen -t rsa -C "zhangjie456@test.com” -f ~/.ssh/github_rsa`

此时，.ssh目录下应该有4个文件：gitlab_rsa和gitlab_rsa.pub，github_rsa和github_rsa.pub，
分别将他们的公钥文件（gitlab_rsa.pub，github_rsa.pub）内容配置到对应的code仓库上。
</br>
配置公钥：登录github或你的代码托管平台。右上角你的账号登录个人信息处，点击settings，选SSH Keys。
</br>
Title自己定义，Key输入你的公钥文件复制的内容，最后点击Add SSH Key按钮即可。
</br>

### 添加私钥
```
ssh-add ~/.ssh/gitlab_rsa
ssh-add ~/.ssh/github_rsa

## 如果执行ssh-add时提示”Could not open a connection to your authentication agent”，
## 可以先执行命令:
ssh-agent bash

## 重新执行ssh-add命令
## 添加后我们可以通过 ssh-add -l 来确私钥列表
ssh-add -l

## 如果想删除私钥列表，可以通过 ssh-add -D 来清空私钥列表
ssh-add -D
```

### 修改配置文件

如果.ssh目录(就是私钥所在的文件夹)下无config文件，那么创建，
```
touch config  ## 创建一个config文件

vi config ## 编辑

## 在config文件添加以下内容

# github
Host github.com
Port 22
HostName github.com
PreferredAuthentications publickey
IdentityFile C:/Users/DELL/.ssh/github_rsa
User zhangjie456@test.com

# gitlab
Host gitlab.com
Port 22
HostName gitlab.com
PreferredAuthentications publickey
IdentityFile C:/Users/DELL/.ssh/gitlab_rsa
User zhangjie123@qq.com

# 配置文件参数
# Host : Host可以看作是一个你要识别的模式，对识别的模式，进行配置对应的的主机名和ssh文件（可以直接填写ip地址）
# Port: 端口号（默认22）
# HostName : 要登录主机的主机名（建议与Host一致）
# User : 登录名（如gitlab的username）
# IdentityFile : 指明上面User对应的identityFile路径

```

### 测试

`ssh -T git@github.com`

如果出现“Enter passphrase for key”提示，请输入密码。

输出“Hi zjie0723! You've successfully authenticated” 或者 “Welcome to GitLab”表示成功。

