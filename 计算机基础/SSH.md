登录

    ssh user@host
    ssh host
    ssh -p port user@host

---

为什么SSH是安全的?

    1.远程主机收到用户的登录请求,把自己的公钥发给用户
    2.用户使用这个公钥,将登录密码加密后,发送回来
    3.远程主机用自己的私钥,解密登录密码,如果密码正确,就同意用户登录
    
    远程主机公钥地址
    /etc/ssh/ssh_known_hosts
    ~/.ssh/known_hosts

---

公钥登录

    1.用户的公钥存储在远程主机上
    2.登录的时候,远程主机向用户发送一串随机码
    3.用户使用私钥加密随机码,再发送回远程主机
    4.远程主机用公钥解密
    
    ssh-keygen  //生成公钥和私钥
    ~/.ssh/id_rsa.pub  //生成的公钥
    ~/.ssh/id_rsa  //生成的私钥
    ssh-copy-id user@host  //将公钥存储到远程主机
    ~/.ssh/authorized_keys  //远程主机上用户的公钥存储地址

---

