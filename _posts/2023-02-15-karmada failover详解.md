---
layout: post
title: karmada failover实现原理详解
author: Fish-pro
tags:
- karmada
- failover
date: 2023-02-15 19:26 +0800
---
Karmada (Kubernetes Armada)使用户能够跨多个云运行云原生应用，而不需更改已有的应用。通过使用Kubernetes原生API提供高级调度功能，实现了真正开放的多云。Karmada旨在为多云和混合云场景下的多集群应用管理提供便捷的自动化，具有集中式多云管理、高可用性和故障恢复等关键特性。而其中，目前被客户广泛关注的，便是Karmada的跨集群的故障恢复能力。本文基于Karmada最新的release版本v1.4.2，和大家一起来探究一下karmada跨集群的故障恢复是如何实现，它是通过哪些控制器和调度器来实现的，以及每个控制器在这个过程中承担了什么样的能力？以及如何保证用户业务的高可用性和连续性？

如果您在阅读本文之前，还未了解或者使用过karmada，推荐阅读：

[Karmada官方文档](https://karmada.io/docs/)

[云原生多云应用利器--Karmada 总览篇](https://mp.weixin.qq.com/s?__biz=MzA5NTUxNzE4MQ==&mid=2659273869&idx=1&sn=f6e03df6f34aa6106972193dba1604d8&chksm=8bcbcc1fbcbc4509060f92b3d636c28c6ccaad62fa3aeb4da9f17971b06e655d1d1385ab2f2c&scene=21#wechat_redirect)

[云原生多云应用利器--Karmada 控制器](https://mp.weixin.qq.com/s/OaX1NE6I2ox1b6aT_NMwHg)

## 一、为什么需要故障转移？

+ 管理员在Karmada控制平面上部署离线应用，并将Pod实例分配到多个集群。当集群发生故障时，管理员希望Karmada将故障集群中的Pod实例迁移到满足条件的其他集群中。
+ 普通用户通过Karmada控制平面在集群中部署在线应用。应用包括数据库实例、服务器实例和配置文件。此时，集群故障。客户希望将整个应用程序迁移到另一个合适的集群。在应用迁移过程中，请确保业务不中断。
+ 管理员升级集群后，集群中作为基础设施的容器网络和存储设备将发生变化。管理员希望在升级集群前将集群中的应用迁移到其他合适的集群中。在迁移过程中，必须持续提供服务。

## 二、Karmada故障恢复示例演示

Karmada故障恢复支持两种方式，Duplicated（全量调度策略）和Divided（副本拆分调度策略），本文示例以Divided为例

+ Duplicated，全量调度策略，当满足“传播策略”限制的候选集群数量不少于失败的集群数量时，将根据失败的集群数量将其重新调度到候选集群。否则，不重新调度。
+ Divided，副本拆分调度策略，业力调度程序将尝试将副本迁移到其他运行状况集群。

![image-20230315195034856](/Users/york/Library/Application Support/typora-user-images/image-20230315195034856.png)

1.下载karmada官方v1.4.2 [sourece code](https://github.com/karmada-io/karmada/archive/refs/tags/v1.4.2.tar.gz)后，使用`hack/local-up-karmada.sh`，启动本地的Karmada。启动后，自动纳管了三个工作集群，其中member1和member2使用push模式，member3使用pull模式，push模式和pull模式的区别这里就不展开了。

```bash
[root@10-29-14-46 ~]# export KUBECONFIG=$HOME/.kube/karmada.config
[root@10-29-14-46 ~]# kubectl --kubeconfig $HOME/.kube/karmada.config config use-context karmada-apiserver
Switched to context "karmada-apiserver".
[root@10-29-14-46 ~]# kubectl get cluster
NAME      VERSION   MODE   READY   AGE
member1   v1.23.4   Push   True    32m
member2   v1.23.4   Push   True    32m
member3   v1.23.4   Pull   True    31m
```

2.在karmada控制平面部署如下的应用配置，可以发现我们定义了一个副本数为3的nginx的应用，同时定义了一个传播策略。传播策略中，使用`clusternames`的方式指定了集群亲和性，需要调度到集群member1和member2上。同时在副本调度中，采用副本拆分的方式进行调度，遵循member1权重为1，member2权重为2的静态权重方式进行调度。

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
---
apiVersion: policy.karmada.io/v1alpha1
kind: PropagationPolicy
metadata:
  name: nginx-propagation
spec:
  resourceSelectors:
    - apiVersion: apps/v1
      kind: Deployment
      name: nginx
  placement:
    clusterAffinity:
      clusterNames:
        - member1
        - member2
    replicaScheduling:
      replicaDivisionPreference: Weighted
      replicaSchedulingType: Divided
      weightPreference:
        staticWeightList:
          - targetCluster:
              clusterNames:
                - member1
            weight: 1
          - targetCluster:
              clusterNames:
                - member2
            weight: 2
```

应用下发后，我们将会看到如下结果，member1集群上传播了一个副本数为1的deployment, member2集群上传播了一个副本数为2的deployment。合计三个副本，符合我们调度预期。
查看控制平面发现在控制平面的成员集群对应命名空间下，创建了对应的两个work，这里的work即是通过传播策略和覆盖策略作用后实际需要在成员集群上传播的workload，资源deployment的resourcebinding调度到了集群member1和集群member2。

```bash
[root@10-29-14-46 ~]# kubectl create -f failover.yaml
deployment.apps/nginx created
propagationpolicy.policy.karmada.io/nginx-propagation created
[root@10-29-14-46 ~]# kubectl get deploy,pp
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   3/3     3            3           2m

NAME                                                    AGE
propagationpolicy.policy.karmada.io/nginx-propagation   119s
[root@10-29-14-46 ~]# kubectl get work -A | grep nginx
karmada-es-member1   nginx-687f7fb96f                  True      20m
karmada-es-member2   nginx-687f7fb96f                  True      20m
[root@10-29-14-46 ~]# kubectl get rb nginx-deployment -o yaml
...
spec:
  clusters:
  - name: member2
    replicas: 2
  - name: member2
    replicas: 1    
  replicas: 3
  resource:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
    namespace: default
    resourceVersion: "5776"
    uid: 530aa301-760a-48a7-ada0-fc3a2112564b
...
[root@10-29-14-46 ~]# karmadactl get po
NAME                     CLUSTER   READY   STATUS    RESTARTS   AGE
nginx-85b98978db-d7q92   member2   1/1     Running   0          110s
nginx-85b98978db-xmbp9   member2   1/1     Running   0          110s
nginx-85b98978db-97xbx   member1   1/1     Running   0          110s
[root@10-29-14-46 ~]# karmadactl get deploy
NAME    CLUSTER   READY   UP-TO-DATE   AVAILABLE   AGE     ADOPTION
nginx   member2   2/2     2            2           3m15s   Y
nginx   member1   1/1     1            1           3m15s   Y
```

3.模拟集群发生故障，由于安装集群是使用kind启动的，那么我们直接暂停member1集群的容器，模拟实际情况中，由于网络或者集群本身问题，从而在联邦中失联。

```bash
[root@10-29-14-46 ~]# docker ps -a
CONTAINER ID   IMAGE                  COMMAND                  CREATED          STATUS          PORTS                       NAMES
8794507af450   kindest/node:v1.23.4   "/usr/local/bin/entr…"   52 minutes ago   Up 51 minutes   127.0.0.1:40000->6443/tcp   member2-control-plane
cc57b0eb54fe   kindest/node:v1.23.4   "/usr/local/bin/entr…"   52 minutes ago   Up 51 minutes   127.0.0.1:35728->6443/tcp   karmada-host-control-plane
5ac1815cd40e   kindest/node:v1.23.4   "/usr/local/bin/entr…"   52 minutes ago   Up 51 minutes   127.0.0.1:39837->6443/tcp   member1-control-plane
f5e5f753dcb8   kindest/node:v1.23.4   "/usr/local/bin/entr…"   52 minutes ago   Up 51 minutes   127.0.0.1:33529->6443/tcp   member3-control-plane
[root@10-29-14-46 ~]# docker stop member1-control-plane
member1-control-plane
```

暂停成功，我们再来看一下实际情况，可以发现，集群启动成功后，控制平面的模板状态中依然是`3/3`,符合整体的预期。获取成员集群上的deployment，集群member1网络不可达。集群membe2上获取到了副本数为3的deployment，故障发生时新增了新的副本。
通过查看karmada资源rb(rersourcebinding)和pp(propagationPolicy)后，可以发现，故障转移后，deployment的资源绑定只调度到集群member2。但是有个问题，从rb(rersourcebinding)的配置中，我们可以看到，此时资源未调度到和集群member1，但是此时集群member1对应的集群命名空间下仍然存在work呢？不急，我们来继续进一步探究。

```bash
[root@10-29-14-46 ~]# kubectl get cluster
NAME      VERSION   MODE   READY   AGE
member1   v1.23.4   Push   False   43m
member2   v1.23.4   Push   True    43m
member3   v1.23.4   Pull   True    42m
[root@10-29-14-46 ~]# kubectl get deploy,pp
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   3/3     3            3           11m

NAME                                                    AGE
propagationpolicy.policy.karmada.io/nginx-propagation   11m
[root@10-29-14-46 ~]# karmadactl get deploy
NAME    CLUSTER   READY   UP-TO-DATE   AVAILABLE   AGE   ADOPTION
nginx   member2   3/3     3            3           12m   Y
error: cluster(member1) is inaccessible, please check authorization or network
[root@10-29-14-46 ~]# karmadactl get po
NAME                     CLUSTER   READY   STATUS    RESTARTS   AGE
nginx-85b98978db-8zj5k   member2   1/1     Running   0          3m18s
nginx-85b98978db-d7q92   member2   1/1     Running   0          12m
nginx-85b98978db-xmbp9   member2   1/1     Running   0          12m
error: cluster(member1) is inaccessible, please check authorization or network
[root@10-29-14-46 ~]# kubectl get rb nginx-deployment -o yaml
...
spec:
  clusters:
  - name: member2
    replicas: 3
  replicas: 3
  resource:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
    namespace: default
    resourceVersion: "5776"
    uid: 530aa301-760a-48a7-ada0-fc3a2112564b
...
[root@10-29-14-46 ~]# kubectl get work -A | grep nginx
karmada-es-member1   nginx-687f7fb96f                  True      30m
karmada-es-member2   nginx-687f7fb96f                  True      30m
```

将集群状态恢复，模拟集群被运维管理员修复的情况，此时副本的变化又将会如何呢？可以发现，当故障集群恢复后，集群member2上的副本数不变，同时，控制平面在集群member1对应命名空间的work已经被删除，从而集群member1上的deployment被删除。

```bash
[root@10-29-14-46 ~]# kubectl get cluster
NAME      VERSION   MODE   READY   AGE
member1   v1.23.4   Push   True    147m
member2   v1.23.4   Push   True    147m
member3   v1.23.4   Pull   True    146m
[root@10-29-14-46 ~]# karmadactl get deploy,po
NAME                         CLUSTER   READY   STATUS    RESTARTS   AGE
pod/nginx-85b98978db-2p8hn   member2   1/1     Running   0          73m
pod/nginx-85b98978db-7j8xs   member2   1/1     Running   0          73m
pod/nginx-85b98978db-m897g   member2   1/1     Running   0          70m

NAME                    CLUSTER   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   member2   3/3     3            3           73m
[root@10-29-14-46 ~]# kubectl get work | grep nginx
No resources found in default namespace.
```

从以上的示例，我们可以发现，当集群发生故障时，karmada会自动调节其余成员集群上的副本，从而来满足用户整体的副本要求，从而达到集群级别的故障恢复。当故障集群恢复后，转移的副本会保持不变，原有调度集群上资源会被删除，karmada v1.4.2中不会发生重调度。

## 三、Karmada故障恢复实现原理

在多集群场景下，用户应用可能部署在多个集群中，以提高业务的高可用性。在Karmada中，当集群发生故障或用户不想继续在集群上运行应用时，集群状态将被标记为不可用，并添加两个污点。

检测到集群故障后，控制器将从故障集群中移除应用。然后，被移除的应用将被调度到满足要求的其他集群。这样可以实现故障转移，保证用户业务的高可用性和连续性。

![](/Users/york/Library/Application Support/typora-user-images/image-20230221113957832.png)

用户在Karmada中加入了三个集群:member1、member2和member3。一个名为foo的部署，它有两个副本，部署在karmada控制平面上。通过使用PropagationPolicy将部署分布到集群member1和member2。

当集群member1发生故障时，集群上的pod实例将被清除并迁移到集群member2或新的集群member3。这种不同的迁移行为可以由PropagationPolicy/ClusterPropagationPolicy的副本调度策略ReplicaSchedulingStrategy控制。

karmada实现故障转移，主要由以下几个控制器和调度器参与完成。

1. clusterStatus controller
2. cluster controller
3. taint-manager controller
4. karmada scheduler
5. gracefulEviction controller
6. binding controller
7. execution controller

接下来我们就来介绍一下每个控制器和调度器，在集群故障的场景中，分别承担了什么样的能力

### 1.clusterStatus controller

![image-20230315200620885](/Users/york/Library/Application Support/typora-user-images/image-20230315200620885.png)

顾名思义，集群状态控制器，它主要用于同步集群实际状态。当集群无法访问时，clusterStatus controller就会感知到集群状态处于非在线状态，此时便会更新控制控制平面的cluster的状态，修改conditions中type为`Ready`的condition状态为`False`, 以下为主要实现代码。

通过访问成员集群的`/readyz`和`/healthz`，获取到集群的在线状态和健康状态。

```go
func getClusterHealthStatus(clusterClient *util.ClusterClient) (online, healthy bool) {
	healthStatus, err := healthEndpointCheck(clusterClient.KubeClient, "/readyz")
	if err != nil && healthStatus == http.StatusNotFound {
		// do health check with healthz endpoint if the readyz endpoint is not installed in member cluster
		healthStatus, err = healthEndpointCheck(clusterClient.KubeClient, "/healthz")
	}

	if err != nil {
		klog.Errorf("Failed to do cluster health check for cluster %v, err is : %v ", clusterClient.ClusterName, err)
		return false, false
	}

	if healthStatus != http.StatusOK {
		klog.Infof("Member cluster %v isn't healthy", clusterClient.ClusterName)
		return true, false
	}

	return true, true
}
```

根据在线状态和健康状态获取初始的`readyCondition`

```go
func generateReadyCondition(online, healthy bool) metav1.Condition {
	if !online {
		return util.NewCondition(clusterv1alpha1.ClusterConditionReady, clusterNotReachableReason, clusterNotReachableMsg, metav1.ConditionFalse)
	}
	if !healthy {
		return util.NewCondition(clusterv1alpha1.ClusterConditionReady, clusterNotReady, clusterUnhealthy, metav1.ConditionFalse)
	}

	return util.NewCondition(clusterv1alpha1.ClusterConditionReady, clusterReady, clusterHealthy, metav1.ConditionTrue)
}
```

如果集群已经下线，并且初始化得到的condition状态为false，则更新集群状态，然后返回

```go
	online, healthy := getClusterHealthStatus(clusterClient)
	observedReadyCondition := generateReadyCondition(online, healthy)
	readyCondition := c.clusterConditionCache.thresholdAdjustedReadyCondition(cluster, &observedReadyCondition)

	// cluster is offline after retry timeout, update cluster status immediately and return.
	if !online && readyCondition.Status != metav1.ConditionTrue {
		klog.V(2).Infof("Cluster(%s) still offline after %s, ensuring offline is set.",
			cluster.Name, c.ClusterFailureThreshold.Duration)
		setTransitionTime(cluster.Status.Conditions, readyCondition)
		meta.SetStatusCondition(&currentClusterStatus.Conditions, *readyCondition)
		return c.updateStatusIfNeeded(cluster, currentClusterStatus)
	}
```

### 2.cluster controller

![image-20230315200831296](/Users/york/Library/Application Support/typora-user-images/image-20230315200831296.png)

顾名思义，集群控制器。集群控制器会根据集群当前的状态下的conditions, 为集群打上不可调度和不可执行的污点。以下代码为核心实现逻辑，当conditions中type为`Ready`的condition状态为`False`时，执行`UpdateClusterControllerTaint`添加effect为`NoSchedule`和`NoExecute`的污点。

```go
func (c *Controller) taintClusterByCondition(ctx context.Context, cluster *clusterv1alpha1.Cluster) error {
	currentReadyCondition := meta.FindStatusCondition(cluster.Status.Conditions, clusterv1alpha1.ClusterConditionReady)
	var err error
	if currentReadyCondition != nil {
		switch currentReadyCondition.Status {
		case metav1.ConditionFalse:
			// Add NotReadyTaintTemplateForSched taint immediately.
			if err = utilhelper.UpdateClusterControllerTaint(ctx, c.Client, []*corev1.Taint{NotReadyTaintTemplateForSched}, []*corev1.Taint{UnreachableTaintTemplateForSched}, cluster); err != nil {
				klog.ErrorS(err, "Failed to instantly update UnreachableTaintForSched to NotReadyTaintForSched, will try again in the next cycle.", "cluster", cluster.Name)
			}
		case metav1.ConditionUnknown:
			// Add UnreachableTaintTemplateForSched taint immediately.
			if err = utilhelper.UpdateClusterControllerTaint(ctx, c.Client, []*corev1.Taint{UnreachableTaintTemplateForSched}, []*corev1.Taint{NotReadyTaintTemplateForSched}, cluster); err != nil {
				klog.ErrorS(err, "Failed to instantly swap NotReadyTaintForSched to UnreachableTaintForSched, will try again in the next cycle.", "cluster", cluster.Name)
			}
		case metav1.ConditionTrue:
			if err = utilhelper.UpdateClusterControllerTaint(ctx, c.Client, nil, []*corev1.Taint{NotReadyTaintTemplateForSched, UnreachableTaintTemplateForSched}, cluster); err != nil {
				klog.ErrorS(err, "Failed to remove schedule taints from cluster, will retry in next iteration.", "cluster", cluster.Name)
			}
		}
	} else {
		// Add NotReadyTaintTemplateForSched taint immediately.
		if err = utilhelper.UpdateClusterControllerTaint(ctx, c.Client, []*corev1.Taint{NotReadyTaintTemplateForSched}, nil, cluster); err != nil {
			klog.ErrorS(err, "Failed to add a NotReady taint to the newly added cluster, will try again in the next cycle.", "cluster", cluster.Name)
		}
	}
	return err
}
```

### 3. taint-manager controller

![image-20230315201328949](/Users/york/Library/Application Support/typora-user-images/image-20230315201328949.png)

污点控制器，该控制器随cluster controller同步启动，可参数配置是否启动，默认启动。它感知cluster的变更事件，当感知到集群拥有effect为`NoExecute`的污点时，会获取到该集群上的所有命名空间级别和集群级别的rb，然后放入对应的驱逐处理队列。驱逐处理消费者（worker）会判断获取到rb对应的pp，看pp中是否存在污点容忍，如果污点容忍匹配，那么直接跳过，否则认为需要驱逐。如果需要驱逐，那么此时会判断是否开启了优雅驱逐，优雅驱逐默认开启。同时，为rb对象写入任务，即`rb.spec.gracefulEvictionTasks`。

```go
	needEviction, tolerationTime, err := tc.needEviction(cluster, binding.Annotations)
	if err != nil {
		klog.ErrorS(err, "Failed to check if binding needs eviction", "binding", fedKey.ClusterWideKey.NamespaceKey())
		return err
	}

	// Case 1: Need eviction now.
	// Case 2: Need eviction after toleration time. If time is up, do eviction right now.
	// Case 3: Tolerate forever, we do nothing.
	if needEviction || tolerationTime == 0 {
		// update final result to evict the target cluster
		if features.FeatureGate.Enabled(features.GracefulEviction) {
			binding.Spec.GracefulEvictCluster(cluster, workv1alpha2.EvictionProducerTaintManager, workv1alpha2.EvictionReasonTaintUntolerated, "")
		} else {
			binding.Spec.RemoveCluster(cluster)
		}
		if err = tc.Update(context.TODO(), binding); err != nil {
			helper.EmitClusterEvictionEventForResourceBinding(binding, cluster, tc.EventRecorder, err)
			klog.ErrorS(err, "Failed to update binding", "binding", klog.KObj(binding))
			return err
		}
		if !features.FeatureGate.Enabled(features.GracefulEviction) {
			helper.EmitClusterEvictionEventForResourceBinding(binding, cluster, tc.EventRecorder, nil)
		}
	} else if tolerationTime > 0 {
		tc.bindingEvictionWorker.AddAfter(fedKey, tolerationTime)
	}
```

可以发现，写入驱逐任务时，会将优雅驱逐任务集群的集群从`rb.spec.clusters`中移除，也就是会从调度结果中移除

```go
// This function no-opts if the cluster does not exist.
func (s *ResourceBindingSpec) GracefulEvictCluster(name, producer, reason, message string) {
   // find the cluster index
   var i int
   for i = 0; i < len(s.Clusters); i++ {
      if s.Clusters[i].Name == name {
         break
      }
   }
   // not found, do nothing
   if i >= len(s.Clusters) {
      return
   }

   // build eviction task
   evictingCluster := s.Clusters[i].DeepCopy()
   evictionTask := GracefulEvictionTask{
      FromCluster: evictingCluster.Name,
      Reason:      reason,
      Message:     message,
      Producer:    producer,
   }
   if evictingCluster.Replicas > 0 {
      evictionTask.Replicas = &evictingCluster.Replicas
   }

   s.GracefulEvictionTasks = append(s.GracefulEvictionTasks, evictionTask)
   s.Clusters = append(s.Clusters[:i], s.Clusters[i+1:]...)
}
```

### 4. karmada scheduler

调度器中，如果当调度的目标副本和期望的目标副本，也就是当`rb.spec.clusters`下所有的副本之和不等于`rb.spec.replicas`时，调度器认为这是一次扩容或者所容操作，从而触发重调度操作。

```go
// IsBindingReplicasChanged will check if the sum of replicas is different from the replicas of object
func IsBindingReplicasChanged(bindingSpec *workv1alpha2.ResourceBindingSpec, strategy *policyv1alpha1.ReplicaSchedulingStrategy) bool {
	if strategy == nil {
		return false
	}
	if strategy.ReplicaSchedulingType == policyv1alpha1.ReplicaSchedulingTypeDuplicated {
		for _, targetCluster := range bindingSpec.Clusters {
			if targetCluster.Replicas != bindingSpec.Replicas {
				return true
			}
		}
		return false
	}
	if strategy.ReplicaSchedulingType == policyv1alpha1.ReplicaSchedulingTypeDivided {
		replicasSum := GetSumOfReplicas(bindingSpec.Clusters)
		return replicasSum != bindingSpec.Replicas
	}
	return false
}
```

在重调度逻辑中，由于集群member1由于存在污点，所以会被调度插件`TaintToleration` filter，结果，只有member2满足要求

```go
// Filter checks if the given tolerations in placement tolerate cluster's taints.
func (p *TaintToleration) Filter(
	_ context.Context,
	bindingSpec *workv1alpha2.ResourceBindingSpec,
	_ *workv1alpha2.ResourceBindingStatus,
	cluster *clusterv1alpha1.Cluster,
) *framework.Result {
	// skip the filter if the cluster is already in the list of scheduling results,
	// if the workload referencing by the binding can't tolerate the taint,
	// the taint-manager will evict it after a graceful period.
	if bindingSpec.TargetContains(cluster.Name) {
		return framework.NewResult(framework.Success)
	}

	filterPredicate := func(t *corev1.Taint) bool {
		return t.Effect == corev1.TaintEffectNoSchedule || t.Effect == corev1.TaintEffectNoExecute
	}

	taint, isUntolerated := v1helper.FindMatchingUntoleratedTaint(cluster.Spec.Taints, bindingSpec.Placement.ClusterTolerations, filterPredicate)
	if !isUntolerated {
		return framework.NewResult(framework.Success)
	}

	return framework.NewResult(framework.Unschedulable, fmt.Sprintf("cluster(s) had untolerated taint {%s}", taint.ToString()))
}
```

在计算可用副本时，由于设置根据静态权重进行副本拆分，但是，因为只有一个集群满足要求，所以在设置集群副本时，会将满足要求的member2集群副本数由2提升为3，从而满足调度期望。

```go

// TakeByWeight divide replicas by a weight list and merge the result into previous result.
func (a *Dispenser) TakeByWeight(w ClusterWeightInfoList) {
	if a.Done() {
		return
	}
	sum := w.GetWeightSum()
	if sum == 0 {
		return
	}

	sort.Sort(w)

	result := make([]workv1alpha2.TargetCluster, 0, w.Len())
	remain := a.NumReplicas
	for _, info := range w {
		replicas := int32(info.Weight * int64(a.NumReplicas) / sum)
		result = append(result, workv1alpha2.TargetCluster{
			Name:     info.ClusterName,
			Replicas: replicas,
		})
		remain -= replicas
	}
	// TODO(Garrybest): take rest replicas by fraction part
	for i := range result {
		if remain == 0 {
			break
		}
		result[i].Replicas++
		remain--
	}

	a.NumReplicas = remain
	a.Result = util.MergeTargetClusters(a.Result, result)
}
```

以下为截取部分调度器日志，可以清晰看到整个filter和副本计算的过程

```shell
I0217 06:46:53.057843       1 scheduler.go:390] Reschedule ResourceBinding(default/nginx-deployment) as replicas scaled down or scaled up
I0217 06:46:53.057888       1 scheduler.go:473] "Begin scheduling resource binding" resourceBinding="default/nginx-deployment"
I0217 06:46:53.083460       1 generic_scheduler.go:119] cluster "member3" is not fit, reason: cluster(s) didn't match the placement cluster affinity constraint
I0217 06:46:53.083524       1 generic_scheduler.go:119] cluster "member1" is not fit, reason: cluster(s) had untolerated taint {cluster.karmada.io/not-ready: }
I0217 06:46:53.083542       1 generic_scheduler.go:73] feasible clusters found: [member2]
I0217 06:46:53.083567       1 generic_scheduler.go:146] Plugin ClusterLocality scores on default/nginx => [{member2 100}]
I0217 06:46:53.083590       1 generic_scheduler.go:146] Plugin ClusterAffinity scores on default/nginx => [{member2 0}]
I0217 06:46:53.083601       1 generic_scheduler.go:79] feasible clusters scores: [{member2 100}]
I0217 06:46:53.085488       1 util.go:72] Target cluster: [{member2 99}]
I0217 06:46:53.085532       1 select_clusters.go:19] select all clusters
I0217 06:46:53.085548       1 generic_scheduler.go:85] selected clusters: [member2]
I0217 06:46:53.085571       1 scheduler.go:489] ResourceBinding default/nginx-deployment scheduled to clusters [{member2 3}]
I0217 06:46:53.151403       1 scheduler.go:491] "End scheduling resource binding" resourceBinding="default/nginx-deployment"
```

### 5. gracefulEviction controller

![image-20230315202056569](/Users/york/Library/Application Support/typora-user-images/image-20230315202056569.png)

优雅驱逐控制器，它感知带有`rb.spec.gracefulEvictionTasks`的rb资源的变更，即感知到需要优雅驱逐的rb。然后评估根据任务集群，获取到任务集群的资源健康状态，如果需求目标集群所有资源健康，则认为不需要保留任务。优雅驱逐控制器即是在保证新集群应用已经启动并且状态健康后，才移除驱逐集群任务。示例中已获取保存的资源状态为健康，所以认为不需要保留优雅驱逐任务。

```go
// assessEvictionTasks assesses each task according to graceful eviction rules and
// returns the tasks that should be kept.
func assessEvictionTasks(bindingSpec workv1alpha2.ResourceBindingSpec,
	observedStatus []workv1alpha2.AggregatedStatusItem,
	timeout time.Duration,
	now metav1.Time,
) ([]workv1alpha2.GracefulEvictionTask, []string) {
	var keptTasks []workv1alpha2.GracefulEvictionTask
	var evictedClusters []string

	for _, task := range bindingSpec.GracefulEvictionTasks {
		// set creation timestamp for new task
		if task.CreationTimestamp.IsZero() {
			task.CreationTimestamp = now
			keptTasks = append(keptTasks, task)
			continue
		}

		// assess task according to observed status
		kt := assessSingleTask(task, assessmentOption{
			scheduleResult: bindingSpec.Clusters,
			timeout:        timeout,
			observedStatus: observedStatus,
		})
		if kt != nil {
			keptTasks = append(keptTasks, *kt)
		} else {
			evictedClusters = append(evictedClusters, task.FromCluster)
		}
	}
	return keptTasks, evictedClusters
}
```

### 6. binding controller

![image-20230315202536822](/Users/york/Library/Application Support/typora-user-images/image-20230315202536822.png)

资源绑定（rb）控制器，当rb资源发生变更时，会被binding controller所感知。首先会移除孤儿work（孤儿work是指，带有rb资源对应标签的，但是随着rb的更新，当前work已不是rb所期望的work）。除了rb目标集群，`rb.spec.requiredBy`和优雅驱逐`rb.spec.gracefulEvictionTasks`所期望集群下的work,其他都会被移除。在实例中，可以发现当故障转移发生后，可以发现故障命名空间下仍然存在work, 理论上，故障集群下work已经被判定为孤儿，为什么没有删除成功呢。进一步探究会发现，测试work的`deletionTimestamp`不为空，没删除的原因是存在`finalizer`。该`finalizer`属于execution controller, finalizer的实现这里就不展开了。

```bash
[root@10-29-14-46 ~]# kubectl -n karmada-es-member1 get work nginx-687f7fb96f -oyaml
...
apiVersion: work.karmada.io/v1alpha1
kind: Work
metadata:
  annotations:
    resourcebinding.karmada.io/name: nginx-deployment
    resourcebinding.karmada.io/namespace: default
  creationTimestamp: "2023-02-17T08:53:08Z"
  deletionGracePeriodSeconds: 0
  deletionTimestamp: "2023-02-17T08:55:39Z"
  finalizers:
  - karmada.io/execution-controller
  generation: 2
  labels:
    resourcebinding.karmada.io/key: 687f7fb96f
  name: nginx-687f7fb96f
  namespace: karmada-es-member1
  resourceVersion: "28134"
  uid: d215a03c-522d-4a89-9c5d-e87e27338a03
...
```

以下为binding controller的核心代码，核心逻辑是

+ 移除孤儿work
+ 确保rb期望work存在
+ 聚合rb状态
+ 更新模版状态

```go
// syncBinding will sync resourceBinding to Works.
func (c *ResourceBindingController) syncBinding(binding *workv1alpha2.ResourceBinding) (controllerruntime.Result, error) {
	if err := c.removeOrphanWorks(binding); err != nil {
		return controllerruntime.Result{Requeue: true}, err
	}

	workload, err := helper.FetchWorkload(c.DynamicClient, c.InformerManager, c.RESTMapper, binding.Spec.Resource)
	if err != nil {
		if apierrors.IsNotFound(err) {
			// It might happen when the resource template has been removed but the garbage collector hasn't removed
			// the ResourceBinding which dependent on resource template.
			// So, just return without retry(requeue) would save unnecessary loop.
			return controllerruntime.Result{}, nil
		}
		klog.Errorf("Failed to fetch workload for resourceBinding(%s/%s). Error: %v.",
			binding.GetNamespace(), binding.GetName(), err)
		return controllerruntime.Result{Requeue: true}, err
	}
	var errs []error
	start := time.Now()
	err = ensureWork(c.Client, c.ResourceInterpreter, workload, c.OverrideManager, binding, apiextensionsv1.NamespaceScoped)
	metrics.ObserveSyncWorkLatency(binding.ObjectMeta, err, start)
	if err != nil {
		klog.Errorf("Failed to transform resourceBinding(%s/%s) to works. Error: %v.",
			binding.GetNamespace(), binding.GetName(), err)
		c.EventRecorder.Event(binding, corev1.EventTypeWarning, events.EventReasonSyncWorkFailed, err.Error())
		c.EventRecorder.Event(workload, corev1.EventTypeWarning, events.EventReasonSyncWorkFailed, err.Error())
		errs = append(errs, err)
	} else {
		msg := fmt.Sprintf("Sync work of resourceBinding(%s/%s) successful.", binding.Namespace, binding.Name)
		klog.V(4).Infof(msg)
		c.EventRecorder.Event(binding, corev1.EventTypeNormal, events.EventReasonSyncWorkSucceed, msg)
		c.EventRecorder.Event(workload, corev1.EventTypeNormal, events.EventReasonSyncWorkSucceed, msg)
	}
	if err = helper.AggregateResourceBindingWorkStatus(c.Client, binding, workload, c.EventRecorder); err != nil {
		klog.Errorf("Failed to aggregate workStatuses to resourceBinding(%s/%s). Error: %v.",
			binding.GetNamespace(), binding.GetName(), err)
		errs = append(errs, err)
	}
	if len(errs) > 0 {
		return controllerruntime.Result{Requeue: true}, errors.NewAggregate(errs)
	}

	err = c.updateResourceStatus(binding)
	if err != nil {
		return controllerruntime.Result{Requeue: true}, err
	}

	return controllerruntime.Result{}, nil
}
```

### 7. execution controller

![image-20230315203205019](/Users/york/Library/Application Support/typora-user-images/image-20230315203205019.png)

执行控制器，它的能力是将集群对应命名空间的workload在对应集群创建。所以，他的执行是和work的状态和集群的状态是息息相关的。如果work的`deletionTimestamp`不为空，则会继续判断集群状态，如果集群状态为`Not Ready`，则会尝试删除work，但是`finalizer`仍然存在，所以无法清理。如果集群状态为删除，则此时会重新入队列。如果以上两个条件都没有进入，那么则会执行removeFinalizer。也就是示例中当集群恢复，`Ready`后，work为什么被删除了。就是此时execution controllerj将work `finalizer` 移除，从而work被删除，紧随着成员集群对应的deployment被移除。

```go
	if !work.DeletionTimestamp.IsZero() {
		// Abort deleting workload if cluster is unready when unjoining cluster, otherwise the unjoin process will be failed.
		if util.IsClusterReady(&cluster.Status) {
			err := c.tryDeleteWorkload(clusterName, work)
			if err != nil {
				klog.Errorf("Failed to delete work %v, namespace is %v, err is %v", work.Name, work.Namespace, err)
				return controllerruntime.Result{Requeue: true}, err
			}
		} else if cluster.DeletionTimestamp.IsZero() { // cluster is unready, but not terminating
			return controllerruntime.Result{Requeue: true}, fmt.Errorf("cluster(%s) not ready", cluster.Name)
		}

		return c.removeFinalizer(work)
	}
```

以上就是Karmada实现故障恢复的原理，从中可以看出，它是由多个控制器协同完成的。如果其中一个控制器出问题，那么都会影响故障恢复的功能。

## 四、总结

当前版本的karmada可以实现副本拆分和全量策略，但是在真正的业务场景中，如有状态应用如何实现故障转移。转移后如何保证应用的网络和存储，这个有待进一步实际操作验证，需要根据实际情况具体问题具体分析。同时，笔者在同客户交流过程中，有客户提到，集群故障恢复后，是否可以进行重调度，将副本调度回到恢复的集群。目前，在社区会议中有提到这一点，目前社区的方向是，期望将来能提供给客户选择是否重调度的能力。也就是说，将来的karmada版本可能会支持用户自定义当故障恢复时，是否要进行重调度，我们一起期待。

随时多云的发展，Karmada越来越被更多客户接受，这意味着Karmada将会被越来越多的人所使用。在笔者参与贡献的CNCF项目贡献中，Karmada是issue回复最快，代码合并请求（Pull Request）处理最快的项目。所以，基于增长的使用用户，社区快速的响应，我们一起期待karmada会提供更加强大的故障恢复能力。



## 参考

[Karmada官方文档](https://karmada.io/docs/)
