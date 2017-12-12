


 How to use git in Linux?
<!-- vim-markdown-toc GFM -->

* [在同一台电脑上配置多个git账户](#在同一台电脑上配置多个git账户)

<!-- vim-markdown-toc -->

###在同一台电脑上配置多个git账户

1.首先在~/.ssh目录下执行

```
ssh-keygen -t rsa -C "miaoying.new@qq.com"
```
其中 -C "miaoying.new@qq.com" 可以不加。如果加上，则在最后生成的myself_id_rsa.pub文件内容的末尾会带上miaoying.new@qq.com；如果不加，则myself.id.rsa.pub文件内容的末尾会加上当前设备的登录用户名和设备名。

根据提示输入文件名（我输入的是myself_id_rsa，文件名随意取），之后可以看到生成了两个文件：
```
myself_id_rsa   myself_id_rsa.pub
```
其中，myself_id_rsa存放的是私钥，myself_id_rsa.pub存放的是公钥。



2.将公钥添加到github的SSH keys列表里，即表示该github账户可以允许含有该SSH的设备进行读写操作，把该SSH文件拷贝到其他设备上，其他设备也可以对项目进行读写操作。



3.配置好后，该设备上就有两个github账户，需要对项目进行账户指定，即允许哪些用户对项目进行git操作，例如项目Demo，只允许用户名为zhangsan，邮箱为zhangsan@qq.com进行操作，那么在Demo项目根目录下执行 (用户名和邮箱随意取，因为git项目信任的是SSH key，而不是用户名)

```
git config user.name zhangsan
git config user.email zhangsan@qq.com
```

另外，同一台设备上可以生成多个SSH，也就是说以上操作可重复执行多次。

