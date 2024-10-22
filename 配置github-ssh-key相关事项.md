* 可以生成多个id_rsa,通过重命名管理设备(我是因为笔记本与服务器为了免密登录，两者的id_rsa相同，该id_rsa已上传github，但只有笔记本能ssh成功连接github(可能github做了设备计数？))

```shell
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f ~/.ssh/id_rsa_github
$ ls ~/.ssh/
authorized_keys  config  id_rsa  id_rsa_github  id_rsa_github.pub  id_rsa.pub  known_hosts
```

4096表示ssh密钥长度

将公钥`id_rsa_github.pub`上传到github，密钥`id_rsa_github`配置到`~/.ssh/config`:

```shell
$ cat ~/.ssh/config 
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_github
$ ssh -T git@github.com
Hi BenkangPeng! You've successfully authenticated, but GitHub does not provide shell access.
```

如果遇到：

```shell
$ ssh -T git@github.com
Bad owner or permissions on /home/<your_usename>/.ssh/config
```

是因为SSH 对于文件和目录的权限非常严格，如果权限不符合要求，就会导致 SSH 连接失败。

修改相关文件权限：

```shell
chmod 600 ~/.ssh/config
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa_github
chmod 644 ~/.ssh/id_rsa_github.pub
```



如果是在docker中，这一套方法太麻烦了，直接用github的Personal access tokens：Setting->Developer Settings -> Personal access tokens -> Tokens(classic)

将生成的tokens@加在url前面就行了：

```shell
git clone https://<your-token>@github.com/BenkangPeng/notes.git
```

