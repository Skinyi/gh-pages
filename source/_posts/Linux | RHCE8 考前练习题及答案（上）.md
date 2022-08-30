---
title: RHCE8 考前练习题及答案（上）
lang: zh-CN
tags: 
  - Linux
  - RHEL
  - RHCE
  - 认证考试
categories: 
  - [学习, Linux]
  - [日常记录]
  - [迁移文章]
---
> 🔴 此文章由之前的 Typora 笔记迁移过来，内容可能已经过时。

## 常规题目

### 安装和配置 Ansible

按照下方所述，在控制节点 control 上安装和配置 Ansible：

1. 安装所需的软件包。
2. 创建名为 `/home/greg/ansible/inventory` 的静态清单文件，以满足以下要求： 
   - `node1` 是 `dev` 主机组的成员；
   - `node2` 是 `test` 主机组的成员；
   - `node3` 和 `node4` 是 `prod` 主机组的成员；
   - `node5` 是 `balancers` 主机组的成员；
   - `prod` 组是 `webservers` 主机组的成员；
3. 创建名为 `/home/greg/ansible/ansible.cfg` 的配置文件，以满足以下要求：
4. 主机清单文件为 `/home/greg/ansible/inventory`;
5. `playbook` 中使用的角色的位置包括 `/home/greg/ansible/roles`。

💯 **答案**：

```bash
# 1. 安装所需软件包，即安装 Ansible
[greg@control ~]$ sudo yum install ansible -y

# 2. 创建并编辑清单文件
[greg@control ~]$ vim /home/greg/ansible/inventory
```

编辑清单文件的内容如下：

```ini
[dev]
node1

[test]
node2

[prod]
node3
node4

[balancers]
node5

[webservers:children]
prod
```

将 Ansible 系统配置文件拷贝到用户 Ansible 工作目录一份，并修改以下内容，用户在该工作目录下执行 Ansible 相关命令时会优先使用该配置文件：

```bash
[greg@control ~]$ cp /etc/ansible/ansible.cfg /home/greg/ansible/
[greg@control ~]$ mkdir /home/greg/ansible/roles
[greg@control ~]$ vim /home/greg/ansible/ansible.cfg
```

```ini
#inventory = /etc/ansible/hosts
inventory = /home/greg/ansible/inventory

#roles_path = /etc/ansible/roles
roles_path = /home/greg/ansible/roles

#reomote_user = root
remote_user = greg

#host_key_checking = False
host_key_checking = False

[privilege_escalation]
become=True
become_method=sudo
become_user=root
become_ask_pass=False
```

测试结果：

```bash
[greg@control ansible]$ ansible all -m ping    # 不指定模块时默认使用 command 模块
```

### 创建和运行 Ansible 临时命令

作为系统管理员，您需要在受管节点上安装软件。

请按照正文所述，创建一个名为 `/home/greg/ansible/adhoc.sh` 的 shell 脚本，该脚本将使用 Ansible 临时命令在各个受管节点上安装 yum 存储库：

存储库 1：

* 存储库的名称为 `EX294_BASE`;
* 描述为 `EX294 base software`;
* 基础 URL 为 `http://content/rhel8.0/x86_64/dvd/BaseOS`;
* GPG 签名检查为启用状态;
* GPG 密钥 URL 为 `http://content/rhel8.0/x86_64/dvd/RPM-GPG-KEY-redhat-release`;
* 存储库为启用状态。

存储库 2：

* 存储库的名称为 `EX294_STREAM`;
* 描述为 `EX294 stream software`;
* 基础 URL 为 `http://content/rhel8.0/x86_64/dvd/AppStream`;
* GPG 签名检查为启用状态;
* GPG 密钥 URL 为 `http://content/rhel8.0/x86_64/dvd/RPM-GPG-KEY-redhat-release`;
* 存储库为启用状态。

💯 **答案**：

```bash
[greg@control ~]$ vim /home/greg/ansible/adhoc.sh
ansible all -m yum_repository -a "name=EX294_BASE description='EX294 base software' baseurl=http://content/rhel8.0/x86_64/dvd/BaseOS gpgcheck=yes gpgkey=http://content/rhel8.0/x86_64/dvd/RPM-GPG-KEY-redhat-release enabled=yes"
ansible all -m yum_repository -a "name=EX294_STREAM description='EX294 stream software' baseurl=http://content/rhel8.0/x86_64/dvd/AppStream gpgcheck=yes gpgkey=http://content/rhel8.0/x86_64/dvd/RPM-GPG-KEY-redhat-release enabled=yes"
[greg@control ~]$ chmod u+x /home/greg/ansible/adhoc.sh
[greg@control ~]$ /home/greg/ansible/adhoc.sh

# 验证
[greg@control ~]$ ansible all -a "yum repolist"
```

### 安装软件包

创建一个名为 `/home/greg/ansible/packages.yml` 的 `playbook`:

* 将 `php` 和 `mariadb` 软件包安装到 `dev`、`test` 和 `prod` 主机组中的主机上；
* 将 `RPM Development Tools` 软件包组安装到 `dev` 主机组中的主机上；
* 将 `dev` 主机组中主机上的所有软件包更新为最新版本。

💯 **答案**：

1. 首先，创建 playbook 文件：

```bash
[greg@control ~]$ vim /home/greg/ansible/packages.yml
```

2. 编辑该 playbook 文件的内容：

```yml
---
# 要求一
- name: Installation on dev,test,prod
  hosts: dev,test,prod
  tasks:
    - name: Install php and mariadb
      yum:
        name: "{{ item }}"
        state: present
      loop:
        - php
        - mariadb

# 要求二
- name: Installation on dev
  hosts: dev
  tasks:
    - name: Install @RPM Development Tools
      yum:
        name: "@RPM Development Tools"
        state: present

# 要求三
- name: Updates-check on dev
  hosts: dev
  tasks:
    - name: Check Updates
      yum:
        name: "*"    # 注意引号，不然会报错
        state: latest
```

3. 执行 playbook，一般只要结果不是红色就没问题：

```bash
[greg@control ansible]$ ansible-playbook /home/greg/ansible/packages.yml
```

4. 验证，若能查到已安装 php 则说明剧本执行的没问题：

```bash
[greg@control ansible]$ ansible dev -a "rpm -q php"
```

### 使用 RHEL 系统角色

安装 RHEL 系统角色软件包，并创建符合以下条件的 playbook `/home/greg/ansible/timesync.yml`：

* 在所有受管节点上运行
* 使用 `timesync` 角色
* 配置该角色，以使用当前有效的 `NTP` 提供商
* 配置该角色，以使用时间服务器 `172.25.254.254`
* 配置该角色，以启用 `iburst` 参数

💯 **答案**：

1. 安装 RHEL 系统角色包并使用：

```bash
[greg@control ~]$ sudo yum install rhel-system-roles -y
```

> 安装的 Ansible 角色一般会放在 `/usr/share/ansible/roles/` 目录下。在本题，我们需要用到的是 `rhel-system-roles.timesync` 这个角色。
> 要使用该角色，还需要在配置文件中引入，编辑配置文件`/home/greg/ansible/ansible.cfg`，修改以下内容：

```ini
#roles_path = /home/greg/ansible/roles
roles_path = /home/greg/ansible/roles:/usr/share/ansible/roles
```

> 查看本地角色列表：

```bash
[greg@control ansible]$ ansible-galaxy list
```

2. 创建 playbook，并参考文档`/usr/share/ansible/roles/rhel-system-roles.timesync/README.md`中的例子，编辑该 playbook：

```bash
[greg@control ~]$ vim /home/greg/ansible/timesync.yml
```

```yml
- name: Configre NTP server on all
  hosts: all
  vars: 
    timesync_ntp_servers:
      - hostname: 172.25.254.254
        iburst: yes
  roles:
    - timesync
```

3. 执行 playbook，此过程可能会有一些红颜色的报错，但是只要后面的验证没问题就可以：

```bash
[greg@control ansible]$ ansible-playbook /home/greg/ansible/timesync.yml
```

4. 验证：

```bash
[greg@control ansible]$ ansible all -a "chronyc sources"
[greg@control ansible]$ ansible all -a "timedatectl"
```

### 使用 Ansible Galaxy 安装角色

使用 `Ansible Galaxy` 和要求文件 `/home/greg/ansible/roles/requirements.yml`。从以下 URL 下载角色并安装到 `/home/greg/ansible/roles`：

* `http://materials/haproxy.tar` 此角色的名称应当为 `balancer`;
* `http://materials/phpinfo.tar` 此角色的名称应当为 `phpinfo`.

💯 **答案**：

1. 根据题目要求创建 requirements.yml 文件并编辑：

```bash
[greg@control ~]$ vim /home/greg/ansible/roles/requirements.yml
```

```yml
---
- name: balancer
  src: http://materails/haproxy.tar

- name: phpinfo
  src: http://materials/phpinfo.tar
```

2. 使用角色文件安装角色：

```bash
[greg@control ansible]$ ansible-galaxy install -r /home/greg/ansible/roles/requirements.yml -p /home/greg/ansible/roles
```

> 参数意义：
> 
> -r ROLE_FILE, --role-file=ROLE_FILE    指定从文件中导入角色
> -p ROLES_PATH, --roles-path=ROLES_PATH    指定安装角色的位置，默认为 ansible.cfg 中配置的

3. 验证：

```bash
[greg@control ansible]$ ansible-galaxy list
```

### 创建和使用角色

根据下列要求，在 `/home/greg/ansible/roles` 中创建名为 `apache` 的角色：

* `httpd` 软件包已安装，设为在系统启动时启用并启动
* 防火墙已启用并正在运行，并使用允许访问 `Web` 服务器的规则
* 模板文件 `index.html.j2` 已存在，用于创建具有以下输出的文件 `/var/www/html/index.html`:

```
Welcome to HOSTNAME on IPADDRESS
```

其中，`HOSTNAME` 是受管节点的完全限定域名，`IPADDRESS` 则是受管节点的 `IP` 地址。

创建一个使用此角色的 playbook `/home/greg/ansible/roles.yml`，该 playbook 在 `webservers` 主机组中的主机上运行。

💯 **答案**：

1. 创建角色

切换到角色目录并创建 apache 角色的骨架：

```bash
[greg@control ~]$ cd /home/greg/ansible/roles
[greg@control roles]$ ansible-galaxy init apache
[greg@control roles]$ tree /home/greg/ansible/apache/    # 查看 apache 角色的目录结构
```

角色目录中不同的文件夹的作用描述如下：

* defaults: 角色的默认变量；
* files: 包含可以经由此角色部署的文件；
* xxxxxxxxxx # 见 ServerA 配置 yum/dnf 仓库部分 或：[root@node2 ~]\# scp root@server1:/etc/yum.repos.d/Default.repo /etc/yum.repos.d/bash
* meta: 角色定义的一些元数据；
* tasks: 包含角色要执行的主要任务的列表；
* templates: 包含可以经由此角色部署的模板；
* tests: 角色的测试文件；
* vars: 角色的其他变量。

本题我们主要用到 tasks 和 templates 两个目录中的内容，其中 tasks 用来存放该角色所要执行的一些任务，templates 用来存放题目中涉及的模板文件。

2. 编辑角色的 playbook 及创建模板文件

编辑 tasks/main.yml 文件：

```bash
[greg@control roles]$ vim /home/greg/ansible/roles/apache/tasks/main.yml
```

```yml
---
# 0. 开启防火墙 - 注意看开始的说明：防火墙默认关闭，否则第二步会失败
- name: Start Firewalld Service
  service:
    name: firewalld
    state: started

# 1. 安装 httpd 包
- name: Install Apache Packages
  yum: 
    name: httpd
    state: present

# 2. 配置防火墙
- name: Configure Firewall
  firewalld: 
    service: http
    state: enabled
    permanent: yes    # 防火墙规则永久生效
    immediate: yes    # 防火墙规则立即生效（可能会失败，若防火墙事先未开启）

# 3. 根据模板生成网页文件
- name: Generate Web Page from template file
  template:
    src: index.html.j2
    dest: /var/www/html/index.html
    owner: apache
    group: apache
    mode: '0644'
    setype: httpd_sys_content_t

# 4. 配置服务自启
- name: Enable Auto-start Services
  service: 
    name: "{{ item }}"
    state: restarted    # 配置重启让改动生效
    enabled: yes
  loop:
    - firewalld
    - httpd
```

编辑 templates/index.html.j2 模板文件：

```bash
[greg@control roles]$ vim /home/greg/ansbible/roles/apache/templates/index.html.j2
```

```jinja2
Welcome to {{ ansible_hostname }} on {{ ansible_default_ipv4.address }}
```

3. 创建 playbook 使用 apache 角色：

```bash
[greg@control roles]$ vim /home/greg/ansible/roles.yml
```

```yml
---
- name: Deploy Apache Service
  hosts: webservers
  roles: 
    - apache
```

4. 执行 playbook 并验证：

```bash
[greg@control ansible]$ ansible-playbook /home/greg/ansible/newrole.yml
[greg@control ansible]$ curl http://node3    # 验证 node3
[greg@control ansible]$ curl http://node4    # 验证 node4
```

### 从 Ansible Galaxy 使用角色

根据下列要求，创建一个名为 `/home/greg/ansible/roles.yml` 的 playbook ：

1. playbook 中包含一个 play， 该 play 在 `balancers` 主机组中的主机上运行并将使用 `balancer` 角色。
   
   * 此角色配置一项服务，以在 `webservers` 主机组中的主机之间平衡 `Web` 服务器请求的负载。
   
   * 浏览到 `balancers` 主机组中的主机（例如 `http://172.25.250.13` ）将生成以下输出：
     
     ```
     Welcome to serverb.lab.example.com on 172.25.250.11
     ```
   
   * 重新加载浏览器将从另一 Web 服务器生成输出：
     
     ```
     Welcome to serverc.lab.example.com on 172.25.250.12
     ```

2. playbook 中包含一个 play， 该 play 在 `webservers` 主机组中的主机上运行并将使用 `phpinfo` 角色。
   
   * 请通过 URL /hello.php 浏览到 `webservers` 主机组中的主机将生成以下输出：
     
     ```
     Hello PHP World from FQDN
     ```
   
   * 其中，`FQDN` 是主机的完全限定名称。
     
     ```
     Hello PHP World from serverb.lab.example.com
     ```
     
     另外还有 PHP 配置的各种详细信息，如安装的 PHP 版本等。
   
   * 同样，浏览到 `http://172.25.250.12/hello.php` 会生成以下输出：
     
     ```
     Hello PHP World from serverc.lab.example.com
     ```
     
     另外还有 PHP 配置的各种详细信息，如安装的 PHP 版本等。

💯 **答案**：

这个题目中两条任务下的要求看着挺多挺复杂的，然而他们在所给的角色包中都已进行了配置，我们要做的只是编写 playbook 去使用两个角色包而已。

1. 根据题目要求创建并编辑 playbook

```bash
[greg@control ~]$ vim /home/greg/ansible/roles.yml
```

```yml
---
- name: Deploy and Setup Balancers
  hosts: balancers
  roles:
    - balancer

- name: Deploy and Setup Phpinfo
  hosts: webservers
  roles:
    - phpinfo
```

2. 执行剧本并验证：

```bash
[greg@control ansible]$ ansible-playbook /home/greg/ansible/roles.yml
[greg@control ansible]$ curl http://node5    # 会看到负责均衡轮番从 node3 和 node4 获得 web 页面
[greg@control ansible]$ curl http://node3/hello.php
[greg@control ansible]$ curl http://node4/hello.php
```

### 创建和使用逻辑卷

创建一个名为 `/home/greg/ansible/lv.yml` 的 playbook ，它将在所有受管节点上运行以执行下列任务：

创建符合以下要求的逻辑卷：

* 逻辑卷创建在 `research` 卷组中；
* 逻辑卷名称为 `data`;
* 逻辑卷大小为 `1500 MiB`;
* 使用 `ext4` 文件系统格式化逻辑卷；
* 如果无法创建请求的逻辑卷大小，应显示错误信息，并且应改为使用大小 `800 MiB`:

```bash
Could not create logical volume of that size
```

* 如果卷组 `research` 不存在，应显示如下错误信息：

```bash
Volume group done not exist
```

* 不要以任何方式挂载逻辑卷。

💯 **答案**：

1. 创建剧本并编辑：

```bash
[greg@control ~]$ vim /home/greg/ansible/lv.yml
```

```yml
---
- name: Create and Use LV on ALL
  hosts: all
  tasks:
    - name: Create and Use LV on ALL
      block:    # 任务块，这里面的任务执行不成功就会执行 rescue 块里的
        - name: Execute General Steps        
          lvol: 
            vg: research
            lv: data
            size: 1500m

      rescue:    # 救援块，block 块里步骤失败的补救措施
        - name: Execute Rescue Steps - Show Error Message
          debug:
            msg: Could not create logical volume of that size

        - name: Execute Rescue Steps - Plan B
          lvol:
            vg: research
            lv: data
            size: 800m

      always:    # 无论 block 失败与否都会执行的
        - name: Formating LV to Ext4
          filesystem:
            dev: /dev/research/data
            fstype: ext4
      when: " 'research' in ansible_lvm.vgs"    # 判断：当有 research vg 才执行上述任务

    - name: Show Error Message
      debug:
        msg: Volume group done not exist
      when: " 'research' not in ansible_lvm.vgs "
```

2. 执行剧本并验证：

```bash
[greg@control ansible]$ ansible-playbook /home/greg/ansible/lv.yml
[greg@control ansible]$ ansible all -a "lvs"    # 验证
```

### 生成 hosts 文件

* 将一个初始模板文件从 `http://materials/hosts.j2` 下载到 `/home/greg/ansible`;
* 完成该模板，以便用它生成以下文件：针对每个清单主机包含一行内容，其格式与 `/etc/hosts` 相同；
* 创建名为 `/home/greg/ansible/hosts.yml` 的 playbook ，它将使用此模板在 `dev` 主机组中的主机上生成文件 `/etc/myhosts`;
* 该 playbook 运行后， `dev` 主机组中主机上的文件 `/etc/myhosts` 应针对每个受管主机包含一行内容：

```
IP_ADDRESS FQDN HOSTNAME
```

其中，IP_ADDRESS 代表本机 ip，FQDN 代表本机完全限定域名，HOSTNAME 代表本机主机名。

注：清单主机名称的显示顺序不重要。

💯 **答案**：

1. 下载并查看模板文件：

```bash
[greg@control ~]$ wget http://materials/host.j2 -p /home/greg/ansible/ 
[greg@control ~]$ cat /home/greg/ansible/hosts.j2
127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4
::1 localhost localhost.localdomain localhost6 localhost6.localdomain6
```

2. 编辑模板文件，向其添加以下内容：

```jinja2
{% for host in groups.all %}
{{ hostvars[host].ansible_default_ipv4.address }} {{ hostvars[host].ansible_fqdn }} {{ hostvars[host].ansible_hostname }}
{% endfor %}
```

3. 编写剧本，完成任务：

```bash
[greg@control ~]$ vim /home/greg/ansible/hosts.yml
```

```yml
---
- name: Gather Facts of All    # 这一步很重要，有这条才会获取所有主机的 facts
  hosts: all

- name: Main Task
  hosts: dev
  tasks:
    - name: Generate myhosts from template file
      template:
        src: /home/greg/ansible/hosts.j2
        dest: /etc/myhosts
```

4. 执行剧本并验证：

```bash
[greg@control ansible]$ ansible-playbook /home/greg/ansible/hosts.yml
[greg@control ansible]$ ansible dev -a "cat /etc/myhosts"    # 验证
```
