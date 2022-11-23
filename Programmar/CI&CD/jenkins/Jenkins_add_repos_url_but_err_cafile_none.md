# Jenkins 配置pipeline使用私域的gitlab出现 CAfile:none

## 遭遇问题
想在公司内网配置一台jenkins以落地流水线demo，安装后编写了一个带helloWorld的Jenkinsfile的工程，放置于内网的gitlab上。

jenkins在配置pipeline时
1. 定义选中Pipeline script from SCM
2. SCM选择git
3. repository URL填入了内网项目所在url

报出了异常：无法连接仓库 Command "git ls-remote -h https://gitlab. * .com/ * -jenkins-repos.git HEAD" returned status code 128:
stdout:
stderr: fatal: unable to access 'https://gitlab. * .com/ * -jenkins-repos.git/': server certificate verification failed. CAfile: none CRLfile:none

## 问题原因
原因一看就很简单，内网的gitlab虽然使用了https，但是TLS证书是自己颁发的，公认的根证书不认这证书。

这个问题类似于访问某些网站，浏览器会提示这个网站不安全，原因是一样的，网站的证书并非来自于公认的CA机构。

## 解决问题
就像平时我们可以通过在自己的浏览器添加CA证书来避免浏览器提示不安全一样，在jenkins上也能这么操作。

我使用的jenkins是docker启动的，所以这里的解决方法可以完全适用docker上的jenkins，但对其他方式部署的jenkins也有参考意义。

1. 进入jenkins后台
2. 下载公司内部gitlab的ca证书到jenkins后台
3. 执行命令`docker cp ca.crt jenkins:/etc/ssl/certs/`把证书复制到docker内存放证书的位置。
4. 再次尝试，连接成功。