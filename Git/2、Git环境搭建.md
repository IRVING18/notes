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
