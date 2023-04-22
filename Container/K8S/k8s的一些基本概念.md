---
title: k8s的一些基本概念
tags:
- operation
categories:
- k8s
---

# 1.Pod

[<font color=red>Pod | Kubernetes</font>](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/)

1. Pause容器

   又称infra容器。由于不同容器资源是隔离的，那么pod中多个容器是怎么实现通信和共享资源的呢，这就需要pause容器来实现了。

   查看kubelet环境变量配置文件：

   ~~~shell
   $ sudo cat /var/lib/kubelet/kubeadm-flags.env
   KUBELET_KUBEADM_ARGS="--network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.4.1"
   $ sudo docker ps |grep -i pause
   596000cabcbd        registry.aliyuncs.com/google_containers/pause:3.4.1   "/pause"                 3 weeks ago         Up 3 weeks                              k8s_POD_kube-proxy-dhnrz_kube-system_c4e04d58-312d-420b-94e7-303002db7e07_0
   7ed52f69a624        registry.aliyuncs.com/google_containers/pause:3.4.1   "/pause"                 3 weeks ago         Up 3 weeks                              k8s_POD_kube-scheduler-vm-12-3-centos_kube-system_ce43a169b79c11c8561b3ee3a8e3498b_0
   d4100d23547b        registry.aliyuncs.com/google_containers/pause:3.4.1   "/pause"                 3 weeks ago         Up 3 weeks                              k8s_POD_kube-controller-manager-vm-12-3-centos_kube-system_eb813cfbdc724e02bf143d2329ae66c2_0
   。。。。。。
   ~~~

   kubelet的参数定义了pause容器的名称。关于实现原理可参考： [The Almighty Pause Container - Ian Lewis](https://www.ianlewis.org/en/almighty-pause-container)

   简单来讲，就是pause容器第一个启动，它创建了一个network namespace，随后所有的容器就会加入到这个namespace中来共享网络资源。所以实际上pod的生命周期是pause容器的生命周期，和应用的容器无关。

2. 初始化pod

   由于pod中可以运行多个容器，有时候在运行某个容器前，我们想要先运行其他一些命令做一些准备工作以保证容器能够正常运行，比如在运行某个java应用前需要先确定是不是数据库已经正常启动，或者需要执行一些在容器中没有的命令，比如下列例子：

   ~~~yaml
   # 修改物理机内核参数，注意这里的参数只对该pod生效，不会对其他容器生效，因为部分内核参数已经namespace化了
   # 参考：https://kubernetes.io/docs/tasks/administer-cluster/sysctl-cluster/
   spec:
     containers:
     - image：nginx
     。。。。。。
     initContainers:
     - image: alpine:lastest
       command: ["/bin/bash", "-c", "/sbin/sysctl -w net.ipv4.tcp_syncookies=1"]
   ~~~

3. 静态pod

   正常情况下pod由master节点来统一调度，但是在使用kubeadm安装k8s的时候，在master还没有启动之前，那些pod又是怎么启动的呢？这里用到的就是静态pod的概念，它是由kubelet控制的：

   ~~~shell
   $ sudo systemctl status kubelet.service
   ● kubelet.service - kubelet: The Kubernetes Node Agent
      Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
     Drop-In: /usr/lib/systemd/system/kubelet.service.d
              └─10-kubeadm.conf
      Active: active (running) since Sat 2022-09-03 21:41:58 CST; 3 weeks 1 days ago
   。 。。。。。
   $ cat /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf 
   # Note: This dropin only works with kubeadm and kubelet v1.11+
   [Service]
   Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
   Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
   。。。。。。
   $ sudo cat /var/lib/kubelet/config.yaml | grep -i static
   staticPodPath: /etc/kubernetes/manifests
   $ sudo ls -l /etc/kubernetes/manifests
   total 16
   -rw------- 1 root root 2236 Sep  3 21:36 etcd.yaml
   -rw------- 1 root root 3352 Sep  3 21:36 kube-apiserver.yaml
   -rw------- 1 root root 2893 Sep  3 21:36 kube-controller-manager.yaml
   -rw------- 1 root root 1479 Sep  3 21:36 kube-scheduler.yaml
   ~~~

   可以看到以上目录中存在四个yaml，kubelet服务启动时会自动加载这个目录下面的yaml文件。如果这时在这个目录下创建yaml文件的话也会被自动加载。

4. pod的调度

   - 节点标签（label）

     ~~~shell
     $ kubectl label nodes vm-12-13-centos node-role.kubernetes.io/worker=""
     $ kubectl label nodes vm-12-13-centos disk=ssd
     $ cat pod.yaml
     ......
     spec:
       nodeSelector:
         disk: ssd
       containers:
       - image: nginx
     ~~~

     有种特殊的标签就是上面关于Role的node-role.kubernetes.io/worker或者node-role.kubernetes.io/master的标签，其中node-role.kubernetes.io/后面的部分会自动配置为节点的Role值。

   - 注解（annotations）

     注解是node，pod以及其他某些对象都有的属性：

     ~~~shell
     $ kubectl annotate nodes vm-12-13-centos app=nginx
     $ kubectl describe nodes vm-12-13-centos 
     ......
     Annotations:        app: nginx
     ......
     ~~~

     它是对象的元数据，适用于给对象添加一些方便用户管理和分析排错的信息，它不能用于标识和选择对象，但是可以被其他客户端用来检索到这些数据以执行相应的操作。

   - 污点（taint）与tolerations（容忍）

     如果给某台节点设置taint的话，只有设置了tolerations的pod才能运行在该节点上：

     ~~~shell
     $ kubectl taint nodes vm-12-13-centos app=web:NoSchedule
     $ kubectl describe nodes vm-12-13-centos  |grep -i taint
     Taints:             app=web:NoSchedule
     ~~~

     上面的命令中NoSchedule的含义是没有容忍的pod不执行调度，但是原来的pod还在上面，如果是NoExecute则会把上面的pod也驱逐。由于现在节点同时具有了label disk=ssd和taint app=web:NoSchedule，所以当前需要同时满足才能调度到该节点：

     ~~~shell
     $ cat pod.yaml
     ......
     spec:
       nodeSelector:
         disk: ssd
       tolerations:
       - key: "app"
         operator: "Equal"
         value: "web"
         effect: "NoSchedule"
       containers:
       - name: nginx
     ~~~

     只有同时满足以上要求的pod才能运行在该节点上，当只有tolerations，没有label时，pod仍然可以运行在任意节点上。当有多个taint时，pod必须要设置成容忍所有的污点才能运行。

     这里的operator还有一个值是Exists，只有在taint的value为空时才可以使用。

5. 维护模式

   当有时候升级系统或者是进行其他一些维护操作时，为了不影响业务，需要不能再分配pod或者将pod调度到其他节点去运行。

   原有的pod仍然运行，不能分配新pod：

   ~~~shell
   $ kubectl cordon vm-12-13-centos
   $ kubectl get nodes vm-12-13-centos --no-headers 
   vm-12-13-centos   Ready,SchedulingDisabled   worker1   42d   v1.22.1
   ~~~

   所有的pod被驱逐，并且不能分配新pod：

   ~~~shell
   # 节点上有一些deamonset或者其他挂有本地目录的pod
   $ kubectl drain vm-12-13-centos --ignore-daemonsets --delete-emptydir-data 
   $ kubectl get nodes vm-12-13-centos --no-headers 
   vm-12-13-centos   Ready,SchedulingDisabled   worker1   42d   v1.22.1
   ~~~

   以上两种方式的取消是一样的，都是使用uncordon：

   ~~~shell
   $ kubectl uncordon vm-12-13-centos
   node/vm-12-13-centos uncordoned
   $ kubectl get nodes vm-12-13-centos --no-headers 
   vm-12-13-centos   Ready   worker1   42d   v1.22.1
   ~~~

6. 关闭pod的方式：

- 优雅的关闭：默认模式，容器里的进程接受到TERM信号，这种信号会等待当前的进程任务完成后再退出，默认30s宽限期（由terminationGracePeriodSeconds参数控制），超过30s强制删除（SIGKILL信号）；

  在这种情况下由于某些容器中的进程定义了在接受到term信号时就是立即关闭，这会导致正在处理中的session立即中断，比如nginx，从而影响业务，这种情况可以使用pod的lifecycle中的pod hook的功能：

  - postStart：pod启动时和主进程一起启动，不分先后；
  - preStop：删除pod的时候先执行preStop中的命令，再关闭pod。

  从而可以得到下列解决方式：

  ~~~yaml
  spec：
    terminationGracePeriodSeconds： 600
    containers：
    - image: nginx
      lifecycle:
        preStop:
          exec:
            commad: ["/bin/bash", "-c", "/usr/sbin/nginx -s quit"]
  ~~~

  以上preStop命令可以让nginx的pod在关闭前先执行nginx -s quit的正常退出命令，然后再接受TERM信号直接退出，其中宽限期设置为了600s，以便让nginx有足够的时间处理任务。

- 强制关闭：--force --grace-period=0

# 2.密码

1. sercet

   [<font color=red>Secret | Kubernetes</font>](https://kubernetes.io/zh-cn/docs/concepts/configuration/secret/)

   创建

   - 命令行：一般用来存储密码，密钥等；

     ~~~shell
     $ kubectl create secret generic mysecret1 --from-literal=xx=aa --from-literal=yy=bb
     ~~~

   - 文件：文件名为key，文件内容为value

     ~~~shell
     $ kubectl create secret generic mysecret2 --from-file=/etc/hosts
     ~~~

   - 变量文件：文件中的值为key=value对

     ~~~shell
     $ kubectl create secret generic mysecret3 --from-env-file=env.txt 
     ~~~

   - yaml文件

     直接编辑yaml文件来创建，过程略。

   使用

   - 卷：在挂载的目录中创建一个文件，文件名为key，内容为value

     ~~~yaml
     spec:
       volumes:
       - name: xx
         secret:
           secretName: mysecret2
       containers:
       - image: nginx
         volumeMounts:
         - name: xx
           mountPath: /etc/test
     ~~~

     以上结果就是会在/etc/test目录下生成一个名为mysecret2中定义的key的文件，内容就是value的值

     subPath：在有多个key=value对的情况下，只写入指定的文件到挂载点

   - 变量：引用key对应的value值

     ~~~shell
     spec：
       containers：
       - image: mysql
       env:
       - name: MYSQL_ROOT_PASSWORD
         valueFrom:
           secretKeyRef:
             name: mysecret1
             key: xx
     ~~~

2. configmap

   [<font color=red>ConfigMap | Kubernetes</font>](https://kubernetes.io/zh-cn/docs/concepts/configuration/configmap/)

   创建

   - 命令行

     ~~~shell
     kubectl create cm test-cm --from-literal=xx=aa --from-literal=yy=bb
     ~~~

   - 文件：文件名为key，文件内容为value

     ~~~shell
     kubectl create cm test-cm --from-file=/etc/hosts
     ~~~

   - yaml文件

     直接编辑yaml文件来创建，过程略。

   使用

   - 卷：用法与secret一致

     ~~~yaml
     spec:
       volumes:
       - name: xx
         configMap:
           name: test-cm
       containers:
       - image: nginx
         volumeMounts:
         - name: xx
           mountPath: /etc/test
     ~~~

     /etc/test目录下有以key为文件名，value为内容的文件

   - 变量：用法也与secret一致

     ~~~yaml
     spec：
       containers：
       - image: mysql
       env:
       - name: MYSQL_ROOT_PASSWORD
         valueFrom:
           configMapKeyRef:
             name: test-cm
             key: xx
     ~~~

# 3.存储

[<font color=red>存储 | Kubernetes</font>](https://kubernetes.io/zh-cn/docs/concepts/storage/)

1. mountPath不存在时会自动创建；
2. pv全局可见，pvc基于namespace，不同namespace相互隔离，所以可实现pvc的挂载（用户）和pv的创建（admin）权限分开；
3. pvc里accessMode的值如果与pv一样，并且storage小于pv的值，则可以自动进行绑定，最终一个pv只能和一个pvc绑定；
4. 由于pvc相互隔离，当用户看到一个pv时，可能并不知道其他namespace的pvc已经绑定了这个pv，当他试图绑定时，发现处于pending，无法绑定，这种情况需要通过storageClass解决；
5. 删除pod后数据是否仍然保留：
   - emptyDir：删除，它是临时性的，实际内容是保存在内存里，所以删除pod就没有了；
   - hostPath：保留，指定的宿主机路径；
   - 直接使用nfs：保留，数据不受影响；
   - pv：由persistentVolumeReclaimPolicy参数决定：
     - Recycle：数据会被删除，然后pv状态变成Available，也就是说pv重新可以被其他pvc绑定；
     - Retain：数据需要手动删除，pv状态变为Released，也就是不可用，需要手动删除pv然后重建，但是删除pv也不会删除数据（这种方式下需要有人专门管理存储中的pv）。
   - sc：由各分配器中相应的参数决定，在定义sc时指定。

# 4.deployment

[<font color=red>Deployments | Kubernetes</font>](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/)

pod控制器，保证pod运行个数。

1. deployment的labels可以和后面定义的pod不一样；

2. HPA(Horizontal Pod Autoscalers):

   - 配置：

     ~~~shell
     $ kubectl autoscale deployment nginx --max=M --min=N --cpu-percent=x
     $ kubectl get hpa
     ~~~

     cpu使用率最大不超过request的值的x%，默认是80%，最少运行N个pod，最多运行M个pod，不受deployment里面replicas的限制。

   - pod负载降低之后，pod数默认5分钟之后自动减少

3. 镜像的升级、回滚以及滚动升级

   - 回滚

     ~~~shell
     $ kubectl rollout history deployment nginx
     $ kubectl rollout undo deployment nginx --to-revision=2
     ~~~

     可选择版本回滚

   - 滚动升级

     ~~~yaml
     apiVersion: apps/v1
     kind: Deployment
     。。。。。。
     spec：
       strategy：
         rollingUpdate:
           maxSurge: 25%
           maxUnavailable: 25%
         type: RollingUpdate
     ~~~

     maxSurge：最多一次性可以创建的pod数，也可以是具体数目，比如1；

     maxUnavailable：最多删除多少pod，也可以是数字，比如1.

# 5.其他控制器

1. daemonset

   [<font color=red>DaemonSet | Kubernetes</font>](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/daemonset/)

   一个节点只运行一个pod

2. ReplicationController（RC）

   [<font color=red>ReplicationController | Kubernetes</font>](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/replicationcontroller/)

   用法与deployment一致

3. ReplicaSet（RS）

   [<font color=red>ReplicaSet | Kubernetes</font>](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/replicaset/)

   Deployment管理 ReplicaSet，并向 Pod 提供声明式的更新以及许多其他有用的功能。

区别：

| 类型                  | api     | select                           |
| --------------------- | ------- | -------------------------------- |
| deployment            | apps/vi | selector: <br />    matchLabels: |
| daemonset             | apps/vi | selector: <br />    matchLabels: |
| ReplicationController | v1      | selector:                        |
| ReplicaSet            | apps/vi | selector: <br />    matchLabels: |

# 6.探针

[<font color=red>Pod 的生命周期 | Kubernetes</font>](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)

1. liveness probe

   有问题则重建pod

   - command

     ~~~yaml
     spec:
       containers:
       - name: liveness
         image: busybox
         livenessProbe:
           exec:
             command:
             - cat
             - /tmp/healthy
           initailDelaySeconds: 5       # 容器启动后5秒内不探测
           periodSeconds: 5		# 每5s探测一次
     ~~~

   - httpGet

     ~~~yaml
     spec:
       containers:
       - name: liveness
         image: nginx
         livenessProbe:
           failureThreshold: 3
           httpGet:
             path: /index.html
             port: 80
             scheme: HTTP
           initailDelaySeconds: 5       # 容器启动后5秒内不探测
           periodSeconds: 5		# 每5s探测一次
           succesThreshold: 1
     ~~~

     - failureThreshold：失败后的重试次数，默认是3；
     - successThreshold：失败后，最少连续次数才被认定为成功，默认是1。

   - tcpSocket

     是否能和端口建立tcp连接

     ~~~yaml
     spec:
       containers:
       - name: liveness
         image: nginx
         livenessProbe:
           failureThreshold: 3
           tcpSocket:
           	port: 808
           initailDelaySeconds: 5       # 容器启动后5秒内不探测
           periodSeconds: 5		# 每5s探测一次
           succesThreshold: 1
     ~~~

2. readiness probe

   探测到pod有问题之后，svc不会再转发到此pod

   ~~~yaml
   spec:
     containers:
     - name: liveness
       image: nginx
       lifecycle:
         postStart:
           exec:
             command: ["/bin/sh", "-c", "touch /tmp/healthy"]
       readinessProbe:
         exec:
           command:
           - cat
           - /tmp/healthy
   ~~~

   当某个pod中的文件/tmp/healthy被删除时，这个pod就不会再接受svc发过来的请求。

# 7.job

1. job

   [<font color=red>Job</font>](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/job/)

   用于执行一次性任务

   - restart 策略：
     - Never：任务没有完成， 则重建pod运行，最终会有多个pod产生；
     - OnFailure：任务没有完成则重新启动pod运行，只会有一个pod存在。
   - 参数：
     - parallelism：m，一次性并行运行m个pod；
     - completions：m，任务需要成功执行m次才算成功。如果parallelism也设置的话，则必须不能小于parallelism的值；
     - backoffLimit：m，如果job失败，则重试m次；
     - activedeadlineseconds：m，job运行的最长时间。

2. cronjob

   [<font color=red>CronJob | Kubernetes</font>](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/cron-jobs/)

   定时周期性的Job任务。

# 8.service

[<font color=red>服务（Service） | Kubernetes</font>](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/)

1. 基本管理

   - Service通过label来定位pod；
   - 负载均衡（轮询）。

2. 服务发现

   简单来讲，就是pod怎么连接其他svc。

   - Service ClusterIP；
   - 变量：
     - 相同命名空间里一个服务中的pod会自动加载其他svc相关的一些变量，以svcname_PORT或者svcname_SERVICE_HOST等方式命名；
     - 使用此方式，在yaml文件中引用方法为$()，不是常见的{}。
   - DNS：
     - 所有的服务会自动注册到coredns这个dns组件；
     - 访问其他namespace使用服务名.命名空间

3. 服务发布

   - Nodeport：会映射到集群中所有节点的端口；
   - LoadBalancer：需要使用第三方工具；
   - Ingress：
     - Ingress控制器：[<font color=red>Ingress 控制器 | Kubernetes</font>](https://kubernetes.io/zh-cn/docs/concepts/services-networking/ingress-controllers/)，至于多个控制器的配置；
     - Ingress规则：根据Ingress的类型，配置对应的Ingress对象。

# 9.网络管理

[<font color=red>网络策略 | Kubernetes</font>](https://kubernetes.io/zh-cn/docs/concepts/services-networking/network-policies/)

1. ingress

   存在以下多个策略时，是或的关系

   - 允许特定标签的pod访问：spec.podSelector指定应用生效的pod，spec.inrgess.from.podSelector.matchLabels指定客户端pod；
     - 当指定允许访问的端口时，可会导致ICMP协议失效，也就是不能ping；
     - 注意这里的matchLabels为空时，代表匹配所有。
   - 允许特定网段的客户端访问：spec.podSelector指定应用生效的pod，spec.inrgess.from.ipBlock.cidr指定ip段，注意这里的ip不是pod的ip；
   - 允许特定命名空间的pod访问：spec.podSelector指定应用生效的pod，spec.ingress.from.namespaceSelector.matchLebels指定客户端namespace。

2. egress

   - spec.podSelector指定应用生效的pod，spec.egress.to.podSelector.matchLabels指定目标pod；
     - 注意对dns的访问端口的开通

# 10.安全管理

[<font color=red>安全 | Kubernetes</font>](https://kubernetes.io/zh-cn/docs/concepts/security/)

1. kubeconfig文件

   - 申请证书得到crt文件，注意证书申请的yaml写法，使用CertificateSigningRequest对象；
   - 创建kubeconfig文件(同时需要集群ca.crt文件)；
     - 设置集群字段：kubectl config set-cluster
     - 设置用户字段：kubectl config set-credential
     - 设置上下文字段：kubectl config set-context
   - 验证：kubectl auth can-i list pods --as user

2. 授权（RBAC）

   - 基本概念
     - apigroup：对应资源的apiversion的类型的父级，比如deployment的apiversion为apps/v1，则如果定义了一个对deployment资源权限的话，这个就是apps。没有父级的，比如pod或者service，就为空；
     - resources：指定资源类型，比如上面的apigroup如果为空，那到底对pod还是service有效，由这个参数指定；
     - verbs：权限类型，比如get，watch，list等
   - clusterrole可以通过rolebinding或者clusterrolebinding绑定到用户，如果是rolebinding的话，这个binding在哪个命名空间创建的就拥有哪个的权限。
   - 用户
     - user account：用来登录的用户；
     - service account：pod里的进程以什么身份运行
       - 每创建一个sa，会默认创建对应的secret；
       - role或者clusterrole同样可以赋给sa用户。

3. 资源限制

   [<font color=red>策略| Kubernetes</font>](https://kubernetes.io/zh-cn/docs/concepts/policy/[)

   - resource：容器中的spec.containers.resources.limits和requests；
   - LimitRange：限制pod中每个容器最多能够使用的资源，cpu、内存或者pvc等；
     - 设置了limitrange之后，都会创建default的值。如果没有显式定义default的值，则默认为max的值。
   - ResourceQuota：限制命名空间使用的资源，pod或者svc个数等。
     - 启用了资源配额之后，用户必须对这些资源设置limit和request；
     - 可以配置作用域。