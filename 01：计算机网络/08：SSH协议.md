### SSH 协议

------

[TOC]

##### 01：SSH 协议（Secure Shell protocol）：安全外壳协议

- SSH 协议：使用加密来保护客户端和服务器（C/S）之间的连接。所有用户身份验证、命令、输出和文件传输都经过加密（**建立隧道**），以防止网络中的攻击。
- 应用层协议，被设计用来在网络中安全地传输数据。
- 应用：服务器登录

##### 02：远程登录

- SSH 的默认端口是22，也就是说，你的**登录请求会送进远程主机的22端口**。

- ```sh
  ssh user@host
  # 示例
  ssh likang@10.234.11.1
  ```
  


###### 密码登录

1. 客户端填写用户名密码**发起远程登陆**；
2. 远端服务器收到登陆请求后，会**将本地的一个公钥发送给客户端**；
   - **中间人攻击**：如果攻击者插在用户与远程主机之间，用**伪造的公钥，获取用户的登录密码**。再用这个密码登录远程主机，那么SSH的安全机制就荡然无存，伪造公钥不需要远程主机的私钥解密；

3. 客户端收到公钥后，将自己的登陆信息用远端服务器的公钥加密，并将加密后的结果发送给远端服务器。
4. 远端服务器收到登陆密文后，用本地**私钥解密，拿到登陆信息到数据库比较**。登陆信息无误时，显示登陆成功。有误时就报错。
5. 登陆成功后，**客户端将会将远端服务器的公钥保存到本地**，等下次再登陆时，客户端检测到收到的公钥已经在本地存在（通常保存的目录：（$HOME/.ssh/known_hosts），就不会报警告。

###### 公钥登录【免密登录】

1. 客户端在**本地生成一对公私钥**；
2. 将客户端本地生成的**公钥手动添加到远端服务器上**；
3. 客户端发起登陆请求到远端服务器；
4. 远端服务器收到请求后，会本地**生成一串随机字符，并将该随机字符串发送给客户端**；
5. 客户端收到远端服务器的随机字符串后，**用本地私钥加密，并将密文传给远端服务器**；
6. 远端服务器将收到的密文**用保存的客户端公钥解密，并将解密结果与原随机字符串对比**，若一致的话证明客户端可信，允许登陆。
  - ssh-copy-id ip：将你的公共密钥传输到一个远程机器上的 authorized_keys 文件中；
    - ssh-copy-id -i id 公钥路径 ip
  - authorized_keys 文件
    - 远程主机将用户的公钥，保存在登录后的用户主目录的$HOME/.ssh/authorized_keys文件中。公钥就是一段字符串，只要把它追加在authorized_keys文件的末尾就行了。

##### 03：SSH 命令

- ssh [选项] [user@]host [command]
  - -A：开启认证代理连接转发功能；
  - -a：关闭认证代理连接转发功能；
  - -b：使用本机指定地址作为对应连接的源ip地址；
  - **-C**：请求压缩所有数据；
  - -F：指定ssh指令的配置文件；
  - -f：后台执行ssh指令；
  - -g：允许远程主机连接主机的转发端口；
  - -i：指定身份文件；
  - -l：指定连接远程服务器登录用户名；
  - **-N**：不执行远程指令；
  - -o：指定配置选项；
  - -p：指定远程服务器上的端口；
  - **-q**：静默模式；
  - -w：用于创建一个隧道设备，用于在两个主机之间传输数据；
  - -X：开启X11转发功能；
  - -x：关闭X11转发功能；
  - -y：开启信任X11转发功能。

```shell
# 使用 likang 用户名登录 ip 为 10.234.11.1 的主机后，执行 ll 命令
ssh likang@10.234.11.1 ll
```

##### 04：SSH 文件配置

- Host：定义连接主机的别名；

- Hostname：指定需要连接的主机名或IP；

- Port：指定连接目标主机的某个端口；

- User：指定用户连接；

- IdentityFile：使用密钥所在的位置；

- **AddKeysToAgent**：是否允许自动将密钥添加到 ssh-agent 中，相当于ssh-add；

  - **ssh-agent：缓存**，是一个用于管理 SSH 私钥的程序。使用 SSH 连接到远程主机时，需要提供相应的私钥以进行身份验证。可以帮助你在一个会话中只需解锁一次私钥，然后它就会在后台持续运行，并在需要时**自动提供私钥**，而无需每次都输入密码。

- ForwardAgent：表示是否允许**将本地系统上的 ssh-agent 连接通过SSH转发到目标服务器**，这样在目标服务器上可以使用本地系统上的SSH密钥，而不用将私钥复制到目标服务器上。

- **ProxyCommand**：指定一个**用于建立 SSH 连接的代理命令**；

  - ```shell
    # 当你尝试连接到 "internal-server.local" 时，ProxyCommand 参数会告诉 SSH 客户端在连接之前使用 "ssh -W %h:%p proxy-server" 这个命令来建立连接，其中 %h 会被替换为目标主机名，%p 会被替换为目标端口号。
    # 通过 proxy-server 作为代理来连接到 internal-server。
    Host internal-server
        HostName internal-server.local
        User your_username
        ProxyCommand ssh -W %h:%p proxy-server
    ```

- **LocalCommand**：表示当成功连接目标主机时，本地立即执行的命令；

- PermitLocalCommand：是否允许执行本机命令，默认no，和 LocalCommand 配合使用；
  -  PermitLocalCommand yes

- **LocalForward**：端口转发，表示**客户端会创建一个本地端口，从远程主机（SSH Server）转发到指定的主机和端口**；

  - 第一个参数必须是本地Ip：本地Port，第二个参数必须为目标host：目标port。

  - ```shell
    # 本地 3310 端口的数据通过远程主机连接到目标数据库
    LocalForward 127.0.0.1:3336 数据库连接:数据库端口
    ```

##### 05：SSH 隧道

- -w host:port
  - 将客户端过来的标准输入和输出**forward到host和port指定的地方**。
  - -W %h:%p ：%p表示转发到目标机的端口，使用%h和%p可以保证在Hostname和Port变化的情况下不用跟着变化。

##### 06：端口转发示例

- ```shell
  # 堡垒机域名 
  Host login
   Hostname 堡垒机域名
   User likang
   Port 1000
   ForwardAgent yes
   AddKeysToAgent yes
   IdentityFile ~/.ssh/rsa_pub
  
  # 跳板机【远程主机代理】
  Host cm-jump
   Hostname cm-jump的ip
   User likang
   Port 1000
   ForwardAgent yes
   ProxyCommand ssh login -W %h:%p
  
  # 数据库代理【端口转发】
  Host mysql-atlantis
   Hostname 代理服务器IP(ssh server)
   User likang
   Port 1000
   ForwardAgent yes
   ProxyCommand ssh cm-jump nc %h %p
   LocalForward 127.0.0.1:3336 目标数据库IP:3306
  ```


###### known_hosts 文件

- known_hosts 文件是 SSH 客户端用来存储已连接过的远程主机的公钥信息的文件。当客户端尝试连接到一个远程主机时，它会检查这个文件，以确保远程主机的公钥没有被篡改，从而提供一种安全机制，以防止中间人攻击。





