---
title: 软件仓库和软件包模块流管理
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

## 管理 DNF 软件仓库

​将系统注册到订阅管理服务可根据所附加的订阅自动配置软件存储库的访问。

​使用 `dnf repolist all` 查看可用的存储库：

```bash
[skinyi@skinyi ~]\$ dnf repolist all	# 不加选项 all 仅列出已启用的仓库
仓库 id                           仓库名称                                  状态
fedora                            Fedora 33 - x86_64                        启用
fedora-cisco-openh264             Fedora 33 openh264 (From Cisco) - x86_64  启用
fedora-cisco-openh264-debuginfo   Fedora 33 openh264 (From Cisco) - x86_64  禁用
fedora-debuginfo                  Fedora 33 - x86_64 - Debug                禁用
fedora-modular                    Fedora Modular 33 - x86_64                启用
fedora-modular-debuginfo          Fedora Modular 33 - x86_64 - Debug        禁用
fedora-modular-source             Fedora Modular 33 - Source                禁用
fedora-source                     Fedora 33 - Source                        禁用
updates                           Fedora 33 - x86_64 - Updates              启用
updates-debuginfo                 Fedora 33 - x86_64 - Updates - Debug      禁用
updates-modular                   Fedora Modular 33 - x86_64 - Updates      启用
updates-modular-debuginfo         Fedora Modular 33 - x86_64 - Updates - De 禁用
updates-modular-source            Fedora Modular 33 - Updates Source        禁用
updates-source                    Fedora 33 - Updates Source                禁用
updates-testing                   Fedora 33 - x86_64 - Test Updates         禁用
updates-testing-debuginfo         Fedora 33 - x86_64 - Test Updates Debug   禁用
updates-testing-modular           Fedora Modular 33 - x86_64 - Test Updates 禁用
updates-testing-modular-debuginfo Fedora Modular 33 - x86_64 - Test Updates 禁用
updates-testing-modular-source    Fedora Modular 33 - Test Updates Source   禁用
updates-testing-source            Fedora 33 - Test Updates Source     
```

​使用 `dnf config-manager` 来启用或禁用存储库。

```bash
[root@skinyi ~]\# dnf config-manager --enable fedora-cisco-openh264-debuginfo
[root@skinyi ~]\# dnf repolist all | grep fedora-cisco
fedora-cisco-openh264             Fedora 33 openh264 (From Cisco) - x86_64  启用
fedora-cisco-openh264-debuginfo   Fedora 33 openh264 (From Cisco) - x86_64  启用
[root@skinyi ~]\# dnf config-manager --disable fedora-cisco-openh264-debuginfo
[root@skinyi ~]\# dnf repolist all | grep fedora-cisco
fedora-cisco-openh264             Fedora 33 openh264 (From Cisco) - x86_64  启用
fedora-cisco-openh264-debuginfo   Fedora 33 openh264 (From Cisco) - x86_64  禁用
```

### 添加自定义存储库配置

​要启用对新的第三方存储库的支持，可在 */etc/yum.repos.d/* 目录中创建一个文件。存储库配置文件必须以 *.repo* 扩展名结尾。存储库定义包含存储库的 URL 和名称，也定义是否使用 GPG 检查软件包签名，如果是，则还检查 URL 是否指向受信任的 GPG 密钥。

> DNF 的软件仓库配置目录仍是 */etc/yum.repos.d*。

​使用 `dnf config-manager` 来添加一个 dnf 存储库。

```bash
[root@skinyi dnf]\# dnf config-manager \
--add-repo="http://dl.fedoraproject.org/pub/epel/8/x86_64/"
添加仓库自：http://dl.fedoraproject.org/pub/epel/8/x86_64/
[root@skinyi dnf]\# ls /etc/yum.repos.d/
dl.fedoraproject.org_pub_epel_8_x86_64_.repo
fedora-cisco-openh264.repo
......
[root@skinyi dnf]\# dnf repolist all	# 添加后的仓库默认为启用状态
仓库 id                                 仓库名称                            状态
dl.fedoraproject.org_pub_epel_8_x86_64_ created by dnf config-manager from  启用
......
```

​按需修改仓库文件，并配置 GPG 密钥以增强安全性。密钥存储在远程存储库站点上的不同位置，如 *http://dl.fedoraproject.org/pub/epel/RPM_GPG_KEY_EPEL-8*。系统管理员应该将此密钥下载到本地并配置 *dnf* 从本地检索此密钥。例如：

```bash
[root@skinyi dnf]\# vi /etc/yum.repos.d/dl.fedoraproject.org_pub_epel_8_x86_64_.repo 
[EPEL-8]									# id 不能存在空格
name=EPEL8 repositories
baseurl=http://dl.fedoraproject.org/pub/epel/8/x86_64/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-8
:wq
```

​要删除某个软件仓库，将其配置文件删除即可：

```bash
[root@skinyi ~]\# rm -f /etc/yum.repos.d/dl.fedoraproject.org_pub_epel_8_x86_64_.repo 
[root@skinyi ~]\# dnf repolist all | grep EPEL
[root@skinyi ~]\# 
```

### 安装软件仓库及其配置文件

​一些存储库将一个配置文件和 GPG 公钥作为 RPM 软件包的一部分提供，该软件包也可以使用 `dnf install` 命令来下载和安装。

​以下命令将安装 RHEL8 EPEL 软件存储库软件包。

```bash
[root@skinyi ~]\# rpm --import http://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-8
[root@skinyi ~]\# dnf install http://dl.fedoraproject.org/pub/epel/8/x86_64/e/epel-release-8-2.noarch.rpm
```

> 先安装 RPM GPG 密钥再安装签名的软件包，安装软件包时可以通过 `--nogpgcheck` 选项忽略检查 GPG key，但此操作并不安全。

​一个软件仓库配置文件可以配置多个存储库的引用，每一存储库的开头为包含在方括号里的存储库的 id。

```bash
[root@skinyi ~]\# cat /etc/yum.repos.d/fedora.repo 
[fedora]
name=Fedora $releasever - $basearch
#baseurl=http://download.example/pub/fedora/linux/releases/$releasever/Everything/$basearch/os/
metalink=https://mirrors.fedoraproject.org/metalink?repo=fedora-$releasever&arch=$basearch
enabled=1
countme=1
metadata_expire=7d
repo_gpgcheck=0
type=rpm
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$releasever-$basearch
skip_if_unavailable=False

[fedora-debuginfo]
name=Fedora $releasever - $basearch - Debug
#baseurl=http://download.example/pub/fedora/linux/releases/$releasever/Everything/$basearch/debug/tree/
metalink=https://mirrors.fedoraproject.org/metalink?repo=fedora-debug-$releasever&arch=$basearch
enabled=0
metadata_expire=7d
repo_gpgcheck=0
type=rpm
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$releasever-$basearch
skip_if_unavailable=False

[fedora-source]
name=Fedora $releasever - Source
#baseurl=http://download.example/pub/fedora/linux/releases/$releasever/Everything/source/tree/
metalink=https://mirrors.fedoraproject.org/metalink?repo=fedora-source-$releasever&arch=$basearch
enabled=0
metadata_expire=7d
repo_gpgcheck=0
type=rpm
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$releasever-$basearch
skip_if_unavailable=False

```

## 管理软件包模块流

​*模块化* 是构建、组织、分发软件包的一种新型方式，具有透明构建和交付、积极维护和易于安装的特点。

### 模块化的基本概念

- 软件包：普通的 rpm 软件包；
- 模块：将软件包进行分组整合成模块 —— 特定的应用程序、语言运行时或任何逻辑组的表示。这使内容更加精细、更容易找到自己所需；
- 模块流：模块可以在多个流中可用，通常代表它们包含的软件的主要版本，自动选择所需软件的正确版本。
- 配置文件：为了简化安装而引入了安装配置文件 —— 模块特定用例的软件包子集。

### 管理软件包模块流

#### 列出模块

​使用 `dnf module list` 命令列出可用的模块的列表：

```bash
[root@skinyi ~]\# dnf module list	# 列出所有可用的模块流
Fedora 33 openh264 (From Cisco) - x86_64        576  B/s | 989  B     00:01    
Fedora Modular 33 - x86_64
Name                Stream           Profiles Summary                           
ant                 1.10             default  Java build tool                   
                                     [d]                                        
avocado             latest           default  Framework with tools and libraries
                                     [d], min  for Automated Testing            
                                     imal                                       
avocado-vt          latest           common   Avocado Virt Test Plugin          
cobbler             3                default  Versatile Linux deployment server 
cri-o               nightly          default  Kubernetes Container Runtime Inter
                                              face for OCI-based containers    
......
[root@skinyi ~]\# dnf module list perl	# 搜索特定的软件模块
上次元数据过期检查：0:34:07 前，执行于 2021年09月07日 星期二 10时56分52秒。
Fedora Modular 33 - x86_64
Name Stream Profiles                    Summary                                 
perl 5.30   common, default [d], minima Practical Extraction and Report Language
            l                                                                   
perl 5.32   common [d], minimal         Practical Extraction and Report Language

Fedora Modular 33 - x86_64 - Updates
Name Stream Profiles                    Summary                                 
perl 5.30   common, default [d], minima Practical Extraction and Report Language
            l                                                                   
perl 5.32   common [d], minimal         Practical Extraction and Report Language

提示：[d]默认，[e]已启用，[x]已禁用，[i]已安装
```

​使用 `dnf module info [MOD_NAME]` 查看软件模块的详细信息：

```bash
[root@skinyi ~]\# dnf module info perl | head -n 10
上次元数据过期检查：0:36:27 前，执行于 2021年09月07日 星期二 10时56分52秒。
Name             : perl
Stream           : 5.30
Version          : 3320200819225620
Context          : 394aad6b
Architecture     : x86_64
Profiles         : common, default [d], minimal
Default profiles : default
Repo             : fedora-modular
Summary          : Practical Extraction and Report Language
```

> 若不指定模块流，`dnf module info` 将显示使用默认流的模块的配置文件所安装的软件包列表。使用 *module-name:stream* 的格式来查看特定的模块流。添加 *--profile* 选项可显示有关各个模块的配置文件所安装的软件包的信息。如：
>
> ```bash
> [root@skinyi ~]\# dnf module info --profile perl:5.30
> 上次元数据过期检查：0:43:18 前，执行于 2021年09月07日 星期二 10时56分52秒。
> Name    : perl:5.30:3320200819225620:394aad6b:x86_64
> common  : perl
> default : perl
> minimal : perl-interpreter
> 
> Name    : perl:5.30:3320210211084215:394aad6b:x86_64
> common  : perl
> default : perl
> minimal : perl-interpreter
> ```

#### 启用模块流和安装模块

> 必须启用模块流才能安装其模块。可以使用 `yum module enable MOD_STRM` 来手动启用模块流。
>
> ```bash
> [root@skinyi ~]\# dnf module install perl
> 上次元数据过期检查：0:50:52 前，执行于 2021年09月07日 星期二 10时56分52秒。
> 参数 'perl' 可以匹配模块 'perl' 的 2 个流（'5.30', '5.32'），但是这些流都未被启用或非默认
> 无法解析参数 perl
> 错误：请求中出现的问题 :
> 损坏的组或模块： perl
> ```
>
> 对于给定的模块，仅可启用一个模块流，启用其他模块流将禁用原始的模块流。

​使用模块流安装模块：

```bash
# 1. 首先启用要安装的模块
[root@skinyi ~]\# dnf module enable perl:5.32
上次元数据过期检查：3:46:24 前，执行于 2021年09月07日 星期二 10时56分52秒。
依赖关系解决。
================================================================================
 软件包            架构             版本                仓库               大小
================================================================================
启用模块流:
 perl                               5.32                                       

事务概要
================================================================================

确定吗？[y/N]： y
完毕！
# 2. 安装需要的模块
[root@skinyi ~]\# dnf module install perl
上次元数据过期检查：3:48:26 前，执行于 2021年09月07日 星期二 10时56分52秒。
依赖关系解决。
================================================================================
 软件包         架构   版本                               仓库             大小
================================================================================
安装组/模块包:
 perl           x86_64 4:5.32.1-471.module_f33+12592+71aff0e2
                                                          updates-modular  27 k
安装依赖关系:
 annobin        x86_64 9.68-2.fc33                        updates          96 k
 binutils       x86_64 2.35-18.fc33                       updates         5.4 M
 binutils-gold  x86_64 2.35-18.fc33                       updates         774 k
 cpp            x86_64 10.3.1-1.fc33                      updates         9.4 M
 ......
安装模块配置档案:
 perl/common                                                                   

事务概要
================================================================================
安装  258 软件包

总下载：92 M
安装大小：301 M
确定吗？[y/N]： y
......
```

> 安装模块时可以省略选项 *module* 而用前缀 *@* 符号代表模块名，如：`dnf install @perl`。

​验证安装结果（我没有安装）：

```bash
[root@skinyi ~]\# dnf module list perl
上次元数据过期检查：3:52:55 前，执行于 2021年09月07日 星期二 10时56分52秒。
Fedora Modular 33 - x86_64
Name Stream   Profiles                   Summary                                
perl 5.30 [x] common, default [d], minim Practical Extraction and Report Languag
              al                         e                                      
perl 5.32 [x] common [d], minimal        Practical Extraction and Report Languag
                                         e                                      

Fedora Modular 33 - x86_64 - Updates
Name Stream   Profiles                   Summary                                
perl 5.30 [x] common, default [d], minim Practical Extraction and Report Languag
              al                         e                                      
perl 5.32 [x] common [d], minimal        Practical Extraction and Report Languag
                                         e                                      

提示：[d]默认，[e]已启用，[x]已禁用，[i]已安装
```

#### 删除模块和禁用模块流

​删除模块会删除当前启用的模块流的配置集所安装的所有软件包，以及依赖于这些软件包的任何其他软件包和模块。

> 删除模块和切换模块流有些棘手。切换流会重置当前流并启用新流，但本地已安装的软件包并不会被更改。切记删除旧有模块流后再安装新流以保证数据不会丢失或配置错误。

​使用 `dnf module remove MOD_NAME` 或 `dnf remove @MOD_NAME` 删除模块流：

```bash
[root@skinyi ~]\# dnf remove @perl:5.30
上次元数据过期检查：4:04:46 前，执行于 2021年09月07日 星期二 10时56分52秒。
无法匹配参数 perl:5.30 中的配置档案
依赖关系解决。
无需任何处理。
完毕！
```

​使用 `dnf module disable MOD_NAME` 禁用模块流：

```bash
[root@skinyi ~]\# dnf module disable perl:5.30
上次元数据过期检查：4:06:34 前，执行于 2021年09月07日 星期二 10时56分52秒。
依赖关系解决。
无需任何处理。
完毕！
```

#### 切换模块流

​切换模块流需要先删除就有安装的模块，删除模块内容及配置文件后，使用命令 `dnf module reset` 重置模块流：

```bash
[root@skinyi ~]\# dnf module reset perl
上次元数据过期检查：4:12:14 前，执行于 2021年09月07日 星期二 10时56分52秒。
依赖关系解决。
================================================================================
 软件包            架构             版本                仓库               大小
================================================================================
重置模块:
 perl                                                                          

事务概要
================================================================================

确定吗？[y/N]： y
完毕！
```

​然后启用并安装要使用的模块流。
