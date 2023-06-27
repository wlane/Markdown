# Packer 使用说明

https://developer.hashicorp.com/packer/docs

## 1. 下载安装

![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230626201957.png)

请注意有些操作系统已经存在同为packer名称的组件包，所以需要注意将这次安装的packer使用软链接换个名称：

~~~shell
[root@packer vmware]# ll /usr/sbin/packer
lrwxrwxrwx. 1 root root 15 May 11  2019 /usr/sbin/packer -> cracklib-packer
[root@packer vmware]# ll /usr/bin/packer
-rwxr-xr-x. 1 root root 82911232 Jun  2 03:22 /usr/bin/packer
[root@packer vmware]# ln -sf /usr/bin/packer /usr/sbin/packer.io
[root@packer vmware]# packer.io
Usage: packer [--version] [--help] <command> [<args>]

Available commands are:
    build           build image
    。。。。。。
~~~

## 2. 语法

https://developer.hashicorp.com/packer/docs/templates/hcl_templates

优先使用HCL语言。主要需要了解各种Block的语法规则：
https://developer.hashicorp.com/packer/docs/templates/hcl_templates/blocks

以及插件的使用：

https://developer.hashicorp.com/packer/docs/plugins

接下来就以创建一个vmware虚拟机为例说明整个操作流程。

## 3.前提条件

参考：

https://developer.hashicorp.com/packer/plugins/builders/vmware/iso

https://developer.hashicorp.com/packer/plugins/builders/vmware#vmware-workstation-player-on-linux

![](https://images-pigo.oss-cn-beijing.aliyuncs.com/20230627204836.png)

这里采用基于[VMware Workstation](https://www.vmware.com/products/workstation/overview.html) 安装的方式。

1. 安装GUI

   ~~~shell
   [root@packer figo]# dnf -y group install "Server with GUI"
   。。。。。。
   ~~~

2. 安装vmware workstation pro

   参考：https://docs.vmware.com/cn/VMware-Workstation-Pro/index.html

   ~~~shell
   [root@packer figo]# sh VMware-Workstation-Full-17.0.2-21581411.x86_64.bundle --console --required --eulas-agreed
   Extracting VMware Installer...done.
   Installing VMware Workstation 17.0.2
       Configuring...
   [######################################################################] 100%
   Installation was successful.
   ~~~



## 4.配置

https://developer.hashicorp.com/packer/plugins/provisioners/ansible/ansible 

https://developer.hashicorp.com/packer/plugins/builders/vmware/iso

首先创建HCL文件：

~~~shell
[root@packer ~]# mkdir vmware/
[root@packer ~]# cd vmware/
[root@packer vmware]# cat plugins.pkr.hcl
packer {
  required_plugins {
    vmware = {
      version = ">= 1.0.0"
      source = "github.com/hashicorp/vmware"
    }
  }
}
[root@packer vmware]# cat template.pkr.hcl
source "vmware-iso" "basic-example" {
  iso_url = "/data/os/CentOS-7-x86_64-DVD-2207-02.iso"
  iso_checksum = "md5:e116035b231c5f756e926372ab34d5c7"
  ssh_username = "packer"
  ssh_password = "packer"
  shutdown_command = "shutdown -P now"
  disk_size = "20000"
  headless = true
}

build {
  name = "GoldenImage"

  sources = ["sources.vmware-iso.basic-example"]

  provisioner "shell" {
    inline = ["this is a test"]
  }

}
~~~

然后安装插件，时间较长：

~~~shell
[root@packer vmware]# packer.io init plugins.pkr.hcl
~~~

再build：

~~~shell
[root@packer vmware]# PACKER_LOG=1 packer.io build -on-error=abort template.pkr.hcl
~~~

执行完成后会在本目录生成一些文件，假如build过程中有报错：

~~~shell
[root@packer vmware]# ll -a output-basic-example/
total 120
drwxr-xr-x. 2 root root   4096 Jun 28 00:34 .
drwxr-xr-x. 4 root root   4096 Jun 28 00:34 ..
-rw-------. 1 root root 524288 Jun 28 00:34 disk-s001.vmdk
-rw-------. 1 root root 524288 Jun 28 00:34 disk-s002.vmdk
-rw-------. 1 root root 524288 Jun 28 00:34 disk-s003.vmdk
-rw-------. 1 root root 524288 Jun 28 00:34 disk-s004.vmdk
-rw-------. 1 root root 524288 Jun 28 00:34 disk-s005.vmdk
-rw-------. 1 root root    601 Jun 28 00:34 disk.vmdk
-rw-r--r--. 1 root root      0 Jun 28 00:34 packer-basic-example.vmsd
-rw-r--r--. 1 root root   2874 Jun 28 00:34 packer-basic-example.vmx
-rw-r--r--. 1 root root    275 Jun 28 00:34 packer-basic-example.vmxf
~~~

可以在这个目录中找相应的信息。