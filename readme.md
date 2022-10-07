# 创建 Jenkins + GitLab + Harbor + K8S 环境并构建 Spring Boot 项目

#### 环境介绍：

* 虚拟实例1：运行jenkins + gitlab + harbor环境，节点地址192.168.14.244（可以把当前机器的地址进行查找替换）
* 虚拟实例2：运行单节点k8s群集，版本为1.23.00
* 上述两个实例的配置均为4vCPU 8GB内存 127GB 硬盘空间，操纵系统为ubuntu 20.04

#### 第一部分，环境初始化安装

**安装docker**

```
apt -y install apt-transport-https ca-certificates curl software-properties-common
```

```
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
```

```
sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
```

\
修改docker配置文件

```
cat > /etc/docker/daemon.json << EOF
{
    "exec-opts": ["native.cgroupdriver=systemd"],
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "100m",
        "max-file": "10"
    },
    "registry-mirrors": ["https://pqbap4ya.mirror.aliyuncs.com"]
}
EOF
```

\
安装docker docker-compose

```
apt install docker-compose
```

\
下载项目文件（可选）

```
git clone https://github.com/cloudzun/devopslab
```

**安装Jenkins**

创建文件夹

进入文件夹

创建docker-compose文件

使用以下范例填充docker-compose文件

```
version: '2'
 
services:
 jenkins:
 image: docker.io/bitnami/jenkins:2.319.1-debian-10-r11
 restart: always
 ports:
 - '8080:8080'
 - '50000:50000'
 environment:
 - JENKINS_PASSWORD=password
 - JENKINS_USERNAME=admin
 volumes:
 - 'jenkins_data:/bitnami/jenkins'
 
volumes:
 jenkins_data:
 driver: local
```

备注： 如果已经已经下载了项目文件可以直接 cd devopslab/jenkins/

安装Jenkins

<figure><img src="https://pic2.zhimg.com/v2-dd56fc62c310d7309de30d86dfc0d185_b.jpg" alt=""><figcaption></figcaption></figure>

查看安装进度

```
docker logs -f jenkins_jenkins_1
```

<figure><img src="https://pic1.zhimg.com/v2-b05d5b3dfd1ca930cfb1346b0657d81c_b.jpg" alt=""><figcaption></figcaption></figure>

等待看到Jenkins is fully up and running字样说明安装成功

<figure><img src="https://pic4.zhimg.com/v2-3f2eff71a951933d3cd0c4376405ef57_b.jpg" alt=""><figcaption></figcaption></figure>

使用admin/password 登录到[http://192.168.14.244:8080](https://link.zhihu.com/?target=http%3A//192.168.14.244%3A8080),建议修改密码

<figure><img src="https://pic2.zhimg.com/v2-a25dba70630fb7b35f82aa349e949479_b.jpg" alt=""><figcaption></figcaption></figure>

加载jenkins插件:[http://192.168.14.244:8080/pluginManager/available](https://link.zhihu.com/?target=http%3A//192.168.14.244%3A8080/pluginManager/available)

* Git
* Git Parameter
* Git Pipeline for Blue Ocean
* GitLab
* Credentials
* Credentials Binding
* Blue Ocean
* Blue Ocean Pipeline Editor
* Blue Ocean Core JS
* Pipeline SCM API for Blue Ocean
* Dashboard for Blue Ocean
* Build With Parameters
* Dynamic Extended Choice Parameter
* Extended Choice Parameter
* List Git Branches Parameter
* Pipeline
* Pipeline: Declarative
* Kubernetes
* Kubernetes CLI
* Kubernetes Credentials
* Image Tag Parameter
* Active Choices

<figure><img src="https://pic4.zhimg.com/v2-dab9ad31e27c1a207c8b3998ed330fe7_b.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://pic1.zhimg.com/v2-98d786dcc2d885e494fab48a8bfecf1c_b.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://pic2.zhimg.com/v2-b1e47cf1545b0ff78ff67ea2fca3303d_b.jpg" alt=""><figcaption></figcaption></figure>

**安装GitLab**\
创建目录

进入目录

创建docker-compose文件

使用以下代码填充docker-compose文件

```
version: '3.6'
services:
  Gitlab:
    image: 'gitlab/gitlab-ce:latest'
    restart: always
    hostname: 'gitlab.admin.com'
    environment:
      TZ: 'Asia/Shanghai'
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://192.168.14.244'
        gitlab_rails['gitlab_ssh_host'] = '192.168.14.244'
        gitlab_rails['gitlab_shell_ssh_port'] = 10022
        prometheus_monitoring['enable'] = false
    ports:
      - '80:80'
      - '10443:443'
      - '10022:22'
    volumes:
      - '/gitlab-data/config:/etc/gitlab'
      - '/gitlab-data/logs:/var/log/gitlab'
      - '/gitlab-data/data:/var/opt/gitlab'
```

安装gitlab

<figure><img src="https://pic2.zhimg.com/v2-f0507a0e90a6de3e1a04dc8ff453155d_b.jpg" alt=""><figcaption></figcaption></figure>

\
查看安装进度，确认gitlab容器的状态为：healthy\


<figure><img src="https://pic2.zhimg.com/v2-ccb8b696d5c5f528a52a6525d202be45_b.png" alt=""><figcaption></figcaption></figure>

查看初始化密码

```
nano /gitlab-data/config/initial_root_password
```

<figure><img src="https://pic2.zhimg.com/v2-d89dbc630820d3957435d61c398d52a5_b.png" alt=""><figcaption></figcaption></figure>

\
使用root和初始化密码登录到服务器，并修改密码\


<figure><img src="https://pic2.zhimg.com/v2-1d5b51241c5b9ca9147d86186fe1bd99_b.jpg" alt=""><figcaption></figcaption></figure>

\
创建一个公开的组，比如kubernetes\


<figure><img src="https://pic4.zhimg.com/v2-bcbfcd2aff2898965c1eec79d927c257_b.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://pic4.zhimg.com/v2-6299798b9aa2dd09efffe2c5404bf207_b.jpg" alt=""><figcaption></figcaption></figure>

\
从githuba上导入一个项目（比如：[https://github.com/cloudzun/cloudzun](https://link.zhihu.com/?target=https%3A//github.com/cloudzun/cloudzun)）进行测试\


<figure><img src="https://pic2.zhimg.com/v2-5184d83c8aa07d65ff61136296d03759_b.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://pic2.zhimg.com/v2-29c0b8fc1350a8a317659f2548fa5501_b.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://pic2.zhimg.com/v2-62314d6e280e5454db5ac6e341ba8f59_b.jpg" alt=""><figcaption></figcaption></figure>

\
测试gitlab项目

```
git clone http://192.168.14.244/kubernetes/cloudzun.git
```

```
git config --global user.email info@cloudzun.com
```

```
git config --global user.name "Abraham Cheng"
```

\
针对文件做些修改，或者增加一些文件

```
git commit -am "first commit"
```

**安装 Harbor**\
下载安装文件

```
wget https://github.com/goharbor/harbor/releases/download/v2.4.3/harbor-offline-installer-v2.4.3.tgz
```

解压

```
tar xf harbor-offline-installer-v2.4.3.tgz
```

进入目录

加载映像文件和配置文件

```
docker load -i harbor.v2.4.3.tar.gz
```

```
cp harbor.yml.tmpl harbor.yml
```

\
修改配置文件

<figure><img src="https://pic2.zhimg.com/v2-9d8e35d0f06ad4c82ce6a2150aaf0a85_b.jpg" alt=""><figcaption></figcaption></figure>

\
hostname:使用IP地址\
port:使用5000端口\
https:这一段全部注释\


<figure><img src="https://pic4.zhimg.com/v2-3bbb390a98c8849a726a0b698bf59177_b.jpg" alt=""><figcaption></figcaption></figure>

\
harbor\_admin\_password:默认为admin:Harbor12345\
data\_volume:改为/data/harbor

\
预配存储

```
mkdir /data/harbor /var/log/harbor -p
```

\
安装

<figure><img src="https://pic2.zhimg.com/v2-9ebb5da918f7f46cd646fb3208a0b4a1_b.jpg" alt=""><figcaption></figcaption></figure>

\
验证安装过程

<figure><img src="https://pic1.zhimg.com/v2-d0942ddf178ced3d2e698512fdb0a264_b.jpg" alt=""><figcaption></figcaption></figure>

\
登录yml文件定义的密码登录到节点\


<figure><img src="https://pic2.zhimg.com/v2-1d7ee2a36822854d20b84ba8d7c7ce95_b.jpg" alt=""><figcaption></figcaption></figure>

\
创建一个公共项目chengzh\


<figure><img src="https://pic2.zhimg.com/v2-920003d5faa713799516d023ebb15f25_b.jpg" alt=""><figcaption></figcaption></figure>

\
在另一台机器上测试

\
修改daemon.json

```
cat > /etc/docker/daemon.json << EOF
{
 "exec-opts": ["native.cgroupdriver=systemd"],
 "log-driver": "json-file",
 "log-opts": {
 "max-size": "100m",
 "max-file": "10"
 },
 "registry-mirrors": ["https://pqbap4ya.mirror.aliyuncs.com"],
 "insecure-registries": ["192.168.14.244:5000"]
}
EOF
```

\
重启docker

\
登录到harbor

```
docker login 192.168.14.244:5000
```

\
上传映像

```
docker pull alexwhen/docker-2048
```

```
docker tag alexwhen/docker-2048 192.168.14.244:5000/chengzh/docker-2048
```

```
docker push 192.168.14.244:5000/chengzh/docker-2048
```

\
可以在另一台机器上测试

```
docker run -d -p 2048:80  192.168.14.244:5000/chengzh/docker-2048
```

**创建测试K8S群集（单节点群集）**

按照常规步骤安装K8S之后，执行以下操作：

删除master污点，使其能承载工作负载

```
kubectl taint node node node-role.kubernetes.io/master-
```

注意：node为充当master这台机器的计算机名

打印admin.conf文件内容，并复制保存到本地

创建ingress

```
kubectl apply -f https://raw.githubusercontent.com/cloudzun/devopslab/main/ingress.yaml
```

修改daemon.json

```
cat > /etc/docker/daemon.json << EOF
{
 "exec-opts": ["native.cgroupdriver=systemd"],
 "log-driver": "json-file",
 "log-opts": {
 "max-size": "100m",
 "max-file": "10"
 },
 "registry-mirrors": ["https://pqbap4ya.mirror.aliyuncs.com"],
 "insecure-registries": ["192.168.14.244:5000"]
}
EOF
```

重启docker

#### 第二部分，环境整合配置

**配置Jenkins**\
配置jenkins location\
在 [http://192.168.14.244:8080/configure](https://link.zhihu.com/?target=http%3A//192.168.14.244%3A8080/configure) 确认Jenkins URL\


<figure><img src="https://pic3.zhimg.com/v2-003db8754532b7e97e432bb5d06966be_b.jpg" alt=""><figcaption></figcaption></figure>

\
配置agent\
在[http://192.168.14.244:8080/configureSecurity/](https://link.zhihu.com/?target=http%3A//192.168.14.244%3A8080/configureSecurity/) 上启用50000端口\


<figure><img src="https://pic1.zhimg.com/v2-ff3a27cef393212ee426241028586348_b.jpg" alt=""><figcaption></figcaption></figure>

\
配置凭据在 [http://192.168.14.244:8080/credentials/store/system/domain/\_/](https://link.zhihu.com/?target=http%3A//192.168.14.244%3A8080/credentials/store/system/domain/\_/)\
配置kubernetes证书（ kind: secret file id: study-kubernetes）\


<figure><img src="https://pic2.zhimg.com/v2-d795c3200183230a741fbe8e42da40b5_b.jpg" alt=""><figcaption></figcaption></figure>

\
配置Harbor账号密码 id: HARBOR\_ACCOUNT\


<figure><img src="https://pic2.zhimg.com/v2-3c448c58a775ab1cb842b5cb89820ef5_b.jpg" alt=""><figcaption></figcaption></figure>

\
配置gitlab key(可选)\
创建cloud\
[http://192.168.14.244:8080/configureClouds/](https://link.zhihu.com/?target=http%3A//192.168.14.244%3A8080/configureClouds/)\
（id：study-kubernetes）\


<figure><img src="https://pic3.zhimg.com/v2-7450da18decb69992724385b3df665ea_b.jpg" alt=""><figcaption></figcaption></figure>

**配置Harbor**\
创建公共项目：kubernetes\


<figure><img src="https://pic3.zhimg.com/v2-74338adc792b16341637bf7c3fec8daa_b.jpg" alt=""><figcaption></figcaption></figure>

**配置GitLab**\
创建公共组：kubernetes\
在上述公共项目组内导入项目：[https://github.com/cloudzun/spring-boot-project](https://link.zhihu.com/?target=https%3A//github.com/cloudzun/spring-boot-project)\


<figure><img src="https://pic2.zhimg.com/v2-0be80d9efc898c523741a755cb83b065_b.jpg" alt=""><figcaption></figcaption></figure>

\
注意：新导入的公共项目组的地址：[http://192.168.14.244/kubernetes/spring-boot-project.git](https://link.zhihu.com/?target=http%3A//192.168.14.244/kubernetes/spring-boot-project.git)\
对项目中的Jenkins文件做必要调整\
104行 123行的项目url: 地址更新为当前项目地址\


<figure><img src="https://pic2.zhimg.com/v2-3c07eeb1fe4c9ef264e3c6be48d6d2a5_b.jpg" alt=""><figcaption></figcaption></figure>

\
180行的HARBOR\_ADDRESS替换位当前harbor地址（如果未使用80端口，则需要替换端口）\


<figure><img src="https://pic2.zhimg.com/v2-006693df9e17cab8c355eb010e996df5_b.jpg" alt=""><figcaption></figcaption></figure>

\
注意：可参照[https://github.com/cloudzun/devopslab/blob/main/Jenkinsfile](https://link.zhihu.com/?target=https%3A//github.com/cloudzun/devopslab/blob/main/Jenkinsfile)

**配置K8S**

给节点打标签

```
kubectl label node node build=true
```

创建命名空间

```
kubectl create ns kubernetes
```

创建包含映像库用户名密码的secret

```
kubectl create secret docker-registry harborkey --docker-server=192.168.14.244:5000 --docker-username=admin --docker-password=2wsx#EDC --docker-email=info@cloudzun.com -n kubernetes
```

<figure><img src="https://pic4.zhimg.com/v2-a3d2da7c804b8a46b1dc79ec737b26e7_b.png" alt=""><figcaption></figcaption></figure>

#### 第三部分：构建 spring-boot-project 项目

在K8S群集上创建项目所需资源

```
kubectl apply -f https://raw.githubusercontent.com/cloudzun/devopslab/main/spring-boot-project.yaml
```

在Jenkins上创建项目，名称为spring-boot-project，类型为pipeline

<figure><img src="https://pic3.zhimg.com/v2-475e721574ed8f31fa236867def055d2_b.jpg" alt=""><figcaption></figcaption></figure>

使用[http://192.168.14.244/kubernetes/spring-boot-project.git](https://link.zhihu.com/?target=http%3A//192.168.14.244/kubernetes/spring-boot-project.git) 作为代码库

<figure><img src="https://pic1.zhimg.com/v2-2a48e0296504d83775f0cd06ead8b410_b.jpg" alt=""><figcaption></figcaption></figure>

注意：因为项目属于公共项目可以不用输入credentials

点击Build Now手动触发build过程

<figure><img src="https://pic3.zhimg.com/v2-012c383f333939d60814043d820b6c66_b.jpg" alt=""><figcaption></figcaption></figure>

第一次触发会可能失败，报错信息为：

```
The specified working directory should be fully accessible to the remoting executable (RWX): /home/jenkins/agent
```

<figure><img src="https://pic1.zhimg.com/v2-2b12a8596c6bf9a0deddee4662683f8c_b.jpg" alt=""><figcaption></figcaption></figure>

解决方法在K8S节点上设置权限

```
chmod -R 777 /opt/workspace
```

再次使用build with Parameter创建

<figure><img src="https://pic4.zhimg.com/v2-bafb1cf09dd926d6ba2f3994e96f6687_b.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://pic4.zhimg.com/v2-296a85a9cfd019e3340d98e599f52cfb_b.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://pic3.zhimg.com/v2-0cd79bbda59daf0d5d8af54f7daa107e_b.jpg" alt=""><figcaption></figcaption></figure>

使用Blue Ocean查看pipeline运行过程

<figure><img src="https://pic4.zhimg.com/v2-d86f6373cc3c283cd73343055c3125af_b.jpg" alt=""><figcaption></figcaption></figure>

查看Harbor映像库

<figure><img src="https://pic2.zhimg.com/v2-96565aa6d62dce7f2d4cd23d8be0d619_b.jpg" alt=""><figcaption></figcaption></figure>

Pipeline运行结束之后进行验证 [http://k8s:30080](https://link.zhihu.com/?target=http%3A//k8s%3A30080)

```
kubectl patch svc -n kubernetes spring-boot-project  -p '{"spec":{"type": "NodePort"}}'
```

```
kubectl patch service spring-boot-project --namespace=kubernetes spring-boot-project --type='json' --patch='[{"op": "replace", "path": "/spec/ports/0/nodePort", "value":30080}]'
```

<figure><img src="https://pic2.zhimg.com/v2-e83cc39be3091ed92a363b38646a9fbd_b.jpg" alt=""><figcaption></figcaption></figure>

<figure><img src="https://pic1.zhimg.com/v2-047552b9363f3f27b886f254636fa0b4_b.jpg" alt=""><figcaption></figcaption></figure>

在k8s节点上检查项目对应的工作负载和服务

```
kubectl get pod -n kubernetes -o wide
```

```
kubectl get svc -n kubernetes -o wide
```

```
kubectl get deployment -n kubernetes -o wide
```

<figure><img src="https://pic3.zhimg.com/v2-3d1e49414b0c4dc95561e2aec7f6ad16_b.png" alt=""><figcaption></figcaption></figure>

实验圆满结束，如果运行有问题，或者需要更多的解释或说明，请留言或者私信，谢谢！
