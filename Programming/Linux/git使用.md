# Git配置

###  1.配置用户名和邮箱

```shell
git config --global user.name "Ingorant"
git config --global user.email "942597493@qq.com"
```

###  2.生成密钥对

```shell
ssh-keygen -t rsa -C "942597493@qq.com"
```

### 3.查看你生成的公钥并添加到github

```shell
cat ~/.ssh/id_rsa.pub
# 若是Linux，可使用xsel复制到剪切板
cat ~/.ssh/id_rsa.pub | xsel -ib
```

### 4.验证

```shell
ssh -T git@github.com
```
**Note:**

验证之后clone 一下自己的项目，激活“钥匙”







# git使用

### 仓库到本地

```shell
#先跳转到Ingorant目录
git clone 链接 
```

###  本地提交到仓库

```shell
#在哪个路径下其实无所谓 
git status	#查看仓库状态
git add filename 	#添加文件
git commit -m "说明"	#提交
git push

git pull

```

