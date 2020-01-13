# 一、gitlab ssh 秘钥配置
#### 1、在本地生成rsa秘钥，默认生成到.ssh目录下
```shell
    ssh-keygen -t rsa -C "your_email@example.com"
```
#### 2、粘贴 .ssh/id_rsa.pub 文字到gitlab上
点头像-> Setting -> SSH Kes 

# 二、设置linux 命令高亮主题，.oh-my-zsh
#### 1、查看当前环境shell
```shell
   echo $SHELL
```
#### 2、查看系统自带哪些shell
```shell
   cat /etc/shells
```
> 如果没有/bin/zsh的话，安装zsh
 ```shell 
    yum install zsh # centos
    brew install zsh # mac
 ```
> 如果有/bin/zsh 直接设置成默认,设置完重启终端
```shell
    chsh -s /bin/zsh
```
#### 3、安装oh-my-zsh
 ```shell
    git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh
    cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
 ```
 
#### 4、修改主题，vim ~/.zshrc
```java
    ZSH_THEME="ys"
```
#### 5、安装自动补全插件 zsh-autosuggestions
```java
    cd ~/.oh-my-zsh/custom/plugins/
    git clone https://github.com/zsh-users/zsh-autosuggestions
    vi ~/.zshrc
    
    加上 
    plugins=(
        git
        zsh-autosuggestions
    )
    
    设置自动补全颜色
     vim .oh-my-zsh/custom/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh
     ${ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE='fg=13'}
     fg 的值可以切换颜色
```
#### 6、安装自动高亮插件
```java
    cd ~/.oh-my-zsh/custom/plugins/
    git clone https://github.com/zsh-users/zsh-syntax-highlighting.git
    vi ~/.zshrc
    
    请务必保证插件顺序，zsh-syntax-highlighting必须在最后一个。
    plugins=(
        git
        zsh-autosuggestions
        zsh-syntax-highlighting
    )
    然后在文件的最后一行添加：source ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```
#### 7、设置完以上内容之后，可能会发现git status 并没有高亮效果。
```shell
    设置下这个: 
    git config --global color.ui true
    
    设置完可以看下vim ~/.gitconfig 
```
