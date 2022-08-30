---
title: RHCE8 考前练习题及答案（下）
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

## （接上常规题）

### 修改文件内容

按照下方所述，创建一个名为 `/home/greg/ansible/issue.yml` 的 playbook ：

1. 该 `playbook` 将在所有清单主机上运行；

2. 该 `playbook` 会将 `/etc/issue` 的内容替换为下方所示的一行文本：
   
   * 在 `dev` 主机组中的主机上，这行文本显示 为：`Development`;
   * 在 `test` 主机组中的主机上，这行文本显示 为：`Test`;
   * 在 `prod` 主机组中的主机上，这行文本显示 为：`Production`.

💯 **答案**：

1. 创建并编辑剧本：

```bash
[greg@control ~]$ vim /home/greg/ansible/issue.yml
```

```yml
---
- name: Modify File
  hosts: all
  tasks:
    - name: On dev
      copy:
        content: "Development"
        dest: /etc/issue
      when: "inventory_hostname in groups.dev"
    - name: On test
      copy:
        content: "Test"
        dest: /etc/issue
      when: "inventory_hostname in groups.test"
    - name: On prod
      copy:
        content: "Production"
        dest: /etc/issue
      when: "inventory_hostname in groups.prod"
```

2. 执行剧本并验证：

```bash
[greg@control ansible]$ ansible-playbook /home/greg/ansible/issue.yml
[greg@control ansible]$ ansible dev,test,prod -a "cat /etc/issue"    # 验证
```

### 创建 web 内容目录

按照下方所述，创建一个名为 `/home/greg/ansible/webcontent.yml` 的 playbook ：

1. 该 `playbook` 在 `dev` 主机组中的受管节点上运行；

2. 创建符合下列要求的目录 `/webdev`:
   
   * 所有者为 `webdev` 组；
   * 具有常规权限：`owner=read+write+execute`, `group=read+write+execute`, `other=read+execute`;
   * 具有特殊权限：设置组 `ID`;
   * 用符号链接将 `/var/www/html/webdev` 链接到 `/webdev`;
   * 创建文件 `/webdev/index.html`, 其中包含如下所示的单行文件： `Development`;
   * 在 `dev` 主机组中主机上浏览此目录（例如 `http://172.25.250.9/webdev/` ）将生成以下输出：

```
Development
```

💯 **答案**：

本题中由于涉及到 web 服务就需要我们进行安装 apache 并配置以及配置防火墙和 selinux 等一系列工作，但是还好我们之前创建过一个 apache 角色，其中的部分操作满足我们此题的需要因此我们也可以直接把 apache 任务部分直接拿过来用。

1. 创建并编写剧本：

```bash
[greg@control ~]$ vim /home/greg/ansible/webcontent.yml
```

```yml
---
- name: 
  hosts: dev
  tasks:
    - name: Install Apache Packages
      yum: 
        name: httpd
        state: present
    - name: Configure Firewall
      filewalld: 
        service: http
        state: enabled
        permanent: yes    # 防火墙规则永久生效
        immediate: yes    # 防火墙规则立即生效（可能会失败，若防火墙事先未开启）
    - name: Create Webdav Directory
      file:
        src: /webdav
        group: webdav
        mode: '2775'
        state: directory
        setype: httpd_sys_content_t
    - name: Create Link to Webdav
      file:
        src: /webdav
        dest: /var/www/html/webdav
        state: link
    - name: Generate index.html
      copy:
        content: "Development"
        src: /webdav/index.html    # 文件不存在会自动创建
        setype: httpd_sys_content_t
    - name: Enable Auto-start Services
      service: 
        name: "{{ item }}"
        state: restarted    # 配置重启让改动生效
        enabled: yes
      loop:
        - firewalld
        - httpd
```

2. 执行剧本并验证：

```bash
[greg@control ansible]$ ansible-playbook /home/greg/ansible/webcontent.yml
[greg@control ansible]$ curl http://node1/    # 验证
```

### 生成硬件报告

创建一个名为 `/home/greg/ansible/hwreport.yml` 的 playbook, 它将在所有受管节点上生成含有以下信息的输出文件 `/root/hwreport.txt`:

* 清单主机名称；
* 以 `MB` 表示的总内存大小；
* `BIOS` 版本；
* 磁盘设备 `vda` 的大小；
* 磁盘设备 `vdb` 的大小；
* 输出文件中的每一行含有一个 `key=value` 对。

您的 playbook 应当：

* 从 `http://materials/hwreport.empty` 下载文件，并将它保存为 `/root/hwreport.txt`;
* 使用正确的值改为 `/root/hwreport.txt`;
* 如果硬件项不存在，相关的值应设为 `NONE`.

💯 **答案**：

此题要求在 playbook 任务中实现模板文件下载并替换内容，跟原来自己事先下下来再套用模板的思路不一样。我们可以使用 `get_url`和`replace`模块来实现文件下载和文件内容替换，并使用 when 关键字判断硬件是否存在。

1. 下载 hwreport.empty 了解其内容：

```bash
[greg@control ~]$ wget http://materails/hwreport.empty | cat
```

```ini
# Hardware report
HOST=inventoryhostname
MEMORY=memory_in_MB
BIOS=BIOS_version
DISK_SIZE_VDA=disk_vda_size
DISK_SIZE_VDB=disk_vdb_size
```

2. 创建并编辑剧本：

```bash
[greg@control ~]$ vim /home/greg/ansible/hwreport.yml
```

```yml
---
- name: Generate hwreport on all
  hosts: all
  tasks:
    - name: Download Template File
      get_url:
        url: http://materials/hwreport.empty
        dest: /root/hwreport.txt
    - name: Replace Content Of Host Field
      replace:
        path: /root/hwreport.txt
        regexp: "inventoryhostname"
        replace: "{{ ansible_hostname }}"
    - name: Replace Content Of Memory Field
      replace:
        path: /root/hwreport.txt
        regexp: "memory_in_MB"
        replace: "{{ ansible_memory_mb }}"
    - name: Replace Content Of BIOS Field
      replace:
        path: /root/hwreport.txt
        regexp: "BIOS_version"
        replace: "{{ ansible_bios_version }}"
    - name: Replace Content Of DISK_SIZE_VDA Field When VDA Exists
      replace:
        path: /root/hwreport.txt
        regexp: "disk_vda_size"
        replace: "{{ ansible_devices.vda.size }}"
      when: "'vda' in ansible_devices"
    - name: Output NONE When VDA not Exists
      replace:
        path: /root/hwreport.txt
        regexp: "disk_vda_size"
        replace: "NONE"
      when: "'vda' not in ansible_devices"
    - name: Replace Content Of DISK_SIZE_VDB Field When VDB Exists
      replace:
        path: /root/hwreport.txt
        regexp: "disk_vdb_size"
        replace: "{{ ansible_devices.vdb.size }}"
      when: "'vdb' in ansible_devices"
    - name: Output NONE When VDB not Exists
      replace:
        path: /root/hwreport.txt
        regexp: "disk_vdb_size"
        replace: "NONE"
      when: "'vdb' not in ansible_devices"
```

3. 执行 playbook 并验证：

```bash
[greg@control ansible]$ ansible-playbook /home/greg/ansible/hwreport.yml
[greg@control ansible]$ ansible all -a "cat /root/hwreport.txt"
```

### 创建密码库

按照下方所述，创建一个 Ansible 库来存储用户密码：

1. 库名称为 `/home/greg/ansible/locker.yml`；

2. 库中含有两个变量，名称如下：
   
   * `pw_developer`, 值为 `Imadev`;
   * `pw_manager`, 值为 `Imamgr`.

3. 用于加密和解密该库的密码为 `whenyouwishuponastar`；

4. 密码存储在文件 `/home/greg/ansible/secret.txt` 中。

💯 **答案**：

本题的意思是可以通过加密 playbook 来规避泄密等安全风险，在本题中被加密的 playbook 是 `/home/greg/ansible/locker.yml`。

1. 创建密码文件并将密文写入文件

```bash
[greg@control ~]$ echo "whenyouwishuponastar" > /home/greg/ansible/secret.txt
```

2. 创建 Ansible 库存储用户名和密码信息

```bash
[greg@control ~]$ echo "pw_developer: Imadev\npwmanager: Imamgr\n" > /home/greg/ansible/locker.yml
```

3. 加密该 Ansible 库文件并验证

```bash
[greg@control ansible]$ ansible-vault encrypt --vault-id=/home/greg/ansible/secret.txt
[greg@control ansible]$ ansible-vault view /home/greg/ansible/locker.yml    # 解密验证
Vault Password: whenyouwishuponastar
```

### 创建用户账户

1. 从 `http://materials/user_list.yml` 下载要创建的用户的列表，并将它保存到 `/home/greg/ansible`；

2. 使用在本次考试中使用在其他位置创建的密码库 `/home/greg/ansible/locker.yml`，创建名为 `/home/greg/ansible/users.yml` 的 playbook ，从而按以下所述创建用户帐户：
   
   1. 职位描述为 `developer` 的用户应当：
      
      * 在 `dev` 和 `test` 主机组中的受管节点上创建；
      * 从 `pw_developer` 变量分配密码；
      * 是补充组 `devops` 的成员。
   
   2. 职位描述为 `manager` 的用户应当：
      
      * 在 `prod` 主机组中的受管节点上创建；
      * 从 `pw_manager` 变量分配密码；
      * 是补充组 `opsmgr` 的成员。

3. 密码采用 `SHA512` 哈希格式；

4. 您的 playbook 应能够在本次考试中使用在其他位置创建的库密码文件 `/home/greg/ansible/secret.txt` 正常运行。

💯 **答案**：

1. 下载用户列表文件并分析

```bash
[greg@control ~]$ wget http://materials/user_list.yml -p /home/greg/ansible && cat /home/greg/ansible/user_list.yml
```

```yml
users:
  - name: bob
    job: developer
  - name: sally
    job: manager
  - name: fred
    job: developer
```

2. 编写 playbook

```bash
[greg@control ~]$ vim /home/greg/ansible/users.yml
```

```yml
---
- name: Create User on dev,test
  hosts: dev,test
  vars_files: 
    - /home/greg/ansible/users.yml
    - /home/greg/ansible/locker.yml
  tasks:
    - name: Create Group devops
      group:
        name: devops
        state: present
    - name: Create User on dev,test
      user:
        name: "{{ item.name }}"
        password: "{{ pw_developer | password_hash('sha512') }}"
        groups: devops
        state: present
      loop: "{{ users }}"
      when: item.job == 'developer'

- name: Create User on prod
  hosts: prod
    vars_files: 
    - /home/greg/ansible/users.yml
    - /home/greg/ansible/locker.yml
  tasks:
    - name: Create Group opsmgr
      group: 
        name: opsmgr
        state: present
    - name: Create User on prod
      user:
        name: "{{ item.name }}"
        password: "{{ pw_manager | password_hash('sha512') }}"
        groups: opsmgr
        state: present
      loop: "{{ users }}"
      when: item.job == 'manager'
```

3. 运行剧本并验证，注意：上面引用了加密的文件因此必须在参数中指定密码文件进行解密才能正常运行

```bash
[greg@control ansible]$ ansible-playbook --vault-id=/home/greg/ansible/secret.txt /home/greg/ansible/users.yml
[greg@control ansible]$ ansible dev,test,prod -a "tail -2 /etc/passwd"    # 验证
```

### 更新 Ansible 库的密钥

按照下方所述，更新现有 Ansible 库的密钥：

* 从 `http://materials/salaries.yml` 下载 Ansible 库到 `/home/greg/ansible`;
* 当前的库密码为 `insecure8sure`;
* 新的库密码为 `bbs2you9527`;
* 库使用新密码保持加密状态。

💯 **答案**：

1. 下载所需文件并使用当前库密码进行解密查看

```bash
[greg@control ~]$ wget http://materials/salaries.yml -p /home/greg/ansible
[greg@control ansible]$ ansible-vault view /home/greg/ansible/salaries.yml
Vault Password: insecure8sure
haha    # 能看到解密后的内容
```

2. 重新加密文件并使用新密码进行解密查看

```bash
[greg@control ansible]$ ansible-vault rekey /home/greg/ansible/salaries.yml
Vault Password: insecure8sure
New Vault Password: bbs2you9527
Confirm New Vault Password: bbs2you9527
[greg@control ansible]$ ansible-vault view /home/greg/ansible/salaries.yml
Vault Password: bbs2you9527
haha    # 用新密码能看到解密后的内容
```

## 附加题

### 安装 RHEL 角色

安装 RHEL ⻆⾊，并使⽤ rhel-system-roles.selinux ⻆⾊，创建剧本`/home/greg/ansible/selinux.yml`，要求在所有节点运⾏，将 SELINUX 设置为强制模式。

💯 **答案**：

1. 安装 RHEL 系统角色包并使用

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

2. 编写剧本

```bash
[greg@control ~]$ cp /usr/share/doc/rhel-system-roles/selinux/example-selinux-playbook.yml /home/greg/ansible/selinux.yml
[greg@control ~]$ vim /home/greg/ansible/selinux.yml
```

```yml
---
- name: Set Enforcing
  hosts: all
  vars:
    selinux_policy: targeted
    selinux_state: enforcing
  roles:
    - role: rhel-system-roles.selinux
  tasks:
    - name: apply SElinux role
      block:
        - include_role:
            name: rhel-system-roles.selinux
      rescue:
        - name: check
          fail:
          when: not selinux_reboot_required
        - name: reboot
          shell: reboot
        - name: changed
          include_role:
            name: rhel-system-roles.selinux
```

3. 执行剧本并验证

```bash
[greg@control ansible]$ ansible-playbook /home/greg/ansible/selinux.yml
[greg@control ansible]$ ansible all -a "getenforce"    # 验证
```

### 创建新的磁盘分区

编写剧本`/home/greg/ansible/partition.yml`，在 balancers 主机上，划分新的 partition，/dev/vdd，编号1，⼤⼩1500m，格式化成 ext4，挂载到 /newpart ⽬录，如果空间不够，分800m，如果没有vdd，报错。

💯 **答案**：

```bash
[greg@control ~]$ vim /home/greg/ansible/partition.yml
```

```yml
---
- name: 
  hosts: balancers
  tasks:
    - name: Create Mount Point Directory
      file:
        name: /newpart
        state: present
    - name: When /dev/vdd Exists
      block:
        - name: Try To Make A 1500m Partition
          parted:
            device: /dev/vdd
            number: 1
            part_end: 1500Mib
            state: present
      rescue:
        - name: Show Error Messages
          debug:
            msg: Could not create partation of that size
        - name: Try To Make A 800m Partition
          parted:
            device: /dev/vdd
            number: 1
            part_end: 800Mib
            state: present
      always:
        - name: Make File System
          filesystem:
            device: /dev/vdd1
            fstype: ext4
        - name: Mount
          mount:
            src: /dev/vdd1
            path: /newpart
            fstype: ext4
            state: mounted 
      when: "'/dev/vdd' in ansible_facts.devices"
    - name: When /dev/vdd Does not Exist
      debug:
        msg: Disk does not exist
      when: "'/dev/vdd' not in ansible_facts.devices"
```

### 使用 crontab 模块

用户 jack 每三个⽉的每周⽇晚上 22 点 39 分查看⼀次⾃⾝用户登录情况。

💯 **答案**：

```bash
[greg@control ~]$ vim /home/greg/ansible/crontab.yml
```

```yml
---
- name: 
  hosts: all
  tasks:
    - name:
      cron:
        name: Login Time
        minute: "39"    #分
        hour: "22"    #时
        # day: ""     #⽇
        month: "*/3"    #⽉
        weekday: "0"    #周
        user: jack    #指定⽤⼾
        job: "(last && lastb)|grep jack"    #执⾏内容
```

### 创建到期用户账户

创建用户账户，默认密码是 redhat，使用 sha512 方式加密，账⼾ jack，新增设置密码有效期为 30 天。账⼾ jony，新增设置相应的 ID 1111，⽤⼾有效期到2022-01-20。

```bash
[greg@control ~]$ vim /home/greg/ansible/create_user.yml
```

```yml
---
- name: 
  hosts: all
  vars:
    - users: 
      - name: jack
      - name: jony
  tasks:
    - name: Create User
      user:
        name: "{{ item.name }}"
        password: "{{ 'redhat' | password_hash('sha512') }}"
        state: present
      loop: "{{ users }}"
    - name: Set Password Expired Day of jack
      shell: chage -M 30 jack
    - name: Set User Validity Deadline
      user:
        name: jony
        uid: '1111'
        expires: '1642636800'    # 事先执行：date -d 2022-01-20 +%s 获得
```
