# Purpose

需要意识到都任何task都有可能存在潜在的问题，因此一个良好的工作习惯和实施标准未来会减少我们承担的技术债务。



# ansible playbook

1. playbook命名方式：从名称就可以知道playbook的目的 + 内容：简单易懂，复杂逻辑要加注释
2. 大的playbook尽量拆分成小的模块，方便做快速测试和调整；
3. tasks多时使用tag，以便在部署时方便指定特定task；
4. 任何task需要标明清楚它的目的；
5. 尽量使用在不同操作系统中都通用的语法，windows也是一样，需要考虑不用版本通用性；
6. 单个role实现单个目的；
7. 不使用shell，command模块，尽量使用其他模块实现，确保幂等性；
8. 变量需要有默认值，多个role使用同一个变量时创建公共变量文件;
9. 操作日志可追溯。

可参考：

​	[10 habits of great Ansible users](https://www.redhat.com/sysadmin/10-great-ansible-practices)

​	[Ansible Best Practices Roles and Modules](https://www.youtube.com/watch?v=XF2EcT-cIh4&ab_channel=RedHatAnsibleAutomation)

​	[Make your Ansible playbooks flexible, maintainable, and scalable](https://www.youtube.com/watch?v=kNDL13MJG6Y&ab_channel=JeffGeerling)



# Dockerfile

1. 除官方提供的镜像外，尽量使用UBI镜像作为base镜像；
2. 创建LABEL定义镜像的创建时间，目的以及来自于哪个dockerfile等；
3. 将不常更新的部分放在前面步骤，充分利用build cache；
4. 减少指令行数，当然也要考虑指令的变更频繁次数；
5. build需要编译的代码镜像，使用多层构建；
6. 镜像中不能包含敏感数据，通过环境变量或者挂载的方式赋给容器；
7. 确保镜像创建记录，以便提供定时更新的依据；
8. 使用podman，buildah以及skopeo。

可参考：

[Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

[Dockerfile 的最佳实践 ](https://developer.aliyun.com/article/923555)



# OCP Pipeline

1. 需要包含ACS漏洞扫描和image sign步骤；
2. prod环境需要有reviewer，也就是需要提MR来合并；
3. 尽量创建通用的Cluster Task，只有和项目有关的才需要创建专属于本项目的task；
4. 运行task镜像也需要升级；
5. Pipeline文件的处理也需要作为IaC的一部分。

可参考：

[OpenShift Pipelines Tutorial](https://github.com/openshift/pipelines-tutorial)

[OpenShift Pipelines](https://nubenetes.com/openshift-pipelines/)



# Gitlab

1. 权限
2. 提交的流程
3. 清理不必要的仓库