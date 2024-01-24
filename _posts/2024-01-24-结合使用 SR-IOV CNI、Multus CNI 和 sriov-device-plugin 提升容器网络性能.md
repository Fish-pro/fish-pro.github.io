---
layout: post
title: 结合使用 SR-IOV CNI、Multus CNI 和 sriov-device-plugin 提升容器网络性能
author: chatgpt 3.5
tags:
- 容器网络
date: 2024-01-24 01:56 +0800
---
容器网络性能对于高性能应用和大规模容器化部署至关重要。Single Root I/O Virtualization（SR-IOV）和 Multus 是两个 Kubernetes 网络扩展项目，通过结合使用 SR-IOV CNI 和 Multus CNI，并搭配 sriov-device-plugin，可以实现更灵活和高性能的容器网络。本文将介绍如何在 Kubernetes 中实践测试这种组合，以验证其在容器网络中的性能提升效果。

## 准备工作

在开始测试之前，请确保已完成以下准备工作：

1. **硬件支持：** 服务器上的网络适配器必须支持 SR-IOV，并通过 BIOS 或 UEFI 启用了 SR-IOV 功能。

2. **Kubernetes 集群：** 你需要一个运行中的 Kubernetes 集群，确保已正确安装并配置了 SR-IOV CNI、Multus CNI 和 sriov-device-plugin 插件。

3. **Multus 插件：** 确保 Multus CNI 插件已经成功安装在 Kubernetes 集群中。

## 安装 SR-IOV CNI、Multus CNI 和 sriov-device-plugin

### 安装 SR-IOV CNI 插件

```bash
# 克隆 SR-IOV CNI 仓库
git clone https://github.com/intel/sriov-cni.git

# 进入 sriov-cni 目录
cd sriov-cni

# 构建并安装插件
make
```

### 安装 Multus CNI 插件

```bash
# 使用官方提供的 YAML 文件部署 Multus CNI
kubectl apply -f https://raw.githubusercontent.com/intel/multus-cni/master/images/multus-daemonset.yml
```

### 安装 sriov-device-plugin

```bash
# 克隆 sriov-device-plugin 仓库
git clone https://github.com/intel/sriov-network-device-plugin.git

# 进入 sriov-network-device-plugin 目录
cd sriov-network-device-plugin

# 构建并安装插件
make
```

### 部署 sriov-device-plugin DaemonSet

```bash
# 部署 sriov-device-plugin DaemonSet
kubectl create -f sriov-network-device-plugin/deployments/sriovdp-daemonset.yaml
```

## 配置 SR-IOV 和 Multus

### 配置 SR-IOV CNI

创建 SR-IOV CNI 的配置文件，例如 `sriov-cni-config.json`，以指定 SR-IOV 网卡的相关信息。确保配置文件中的参数符合你的硬件和网络设置。

```json
// sriov-cni-config.json
{
  "cniVersion": "0.4.0",
  "name": "sriov-cni",
  "type": "sriov",
  "vlan": 100
}
```

### 配置 Multus CNI

创建 Multus CNI 的配置文件，例如 `multus-cni-config.yaml`，以指定使用 SR-IOV CNI。

```yaml
# multus-cni-config.yaml
apiVersion: "k8s.cni.cncf.io/v1"
kind: NetworkAttachmentDefinition
metadata:
  name: sriov-network
spec:
  config: '{
    "cniVersion": "0.4.0",
    "type": "sriov",
    "vlan": 100
  }'
```

## 创建容器 Pod

创建一个使用 SR-IOV 和 Multus 的 Pod 配置文件，例如 `sriov-multus-pod.yaml`。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sriov-multus-pod
spec:
  containers:
  - name: sriov-multus-container
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
    volumeMounts:
    - name: sriov-net-cfg
      mountPath: /etc/cni/net.d
  volumes:
  - name: sriov-net-cfg
    hostPath:
      path: /etc/cni/net.d
  - name: sriov-fs
    hostPath:
      path: /var/lib/sriov
```

## 部署并验证

部署创建的 Pod：

```bash
kubectl apply -f sriov-multus-pod.yaml
```

等待 Pod 运行，并确保它成功启动。

验证 SR-IOV 和 Multus 是否正确工作：

```bash
kubectl exec -it sriov-multus-pod -- ip a
```

确保 Pod 中存在与 SR-IOV 相关的网络接口。

## 运行网络性能测试

在测试 Pod 内运行网络性能测试，例如使用 iperf 工具。首先，在测试 Pod 中运行 iperf 服务器：

```bash
kubectl exec -it sriov-multus-pod -- iperf3 -s
```

然后，在另一个节点上运行 iperf 客户端测试：

```bash
kubectl run -it --rm --restart=Never iperf-client --image=networkstatic/iperf3 -- iperf3 -c <Pod_IP_Address>
```

替换 `<Pod_IP_Address>` 为测试 Pod 的 IP 地址。观察测试结果，确保性能测试正常运行。

## 结论

通过结合使用 SR-IOV CNI、Multus CNI 和 sriov-device-plugin，我们实现了更灵活和高性能的容器网络。这种组合允许我们充分利用 SR-IOV 的硬件加速特性，并通过 Multus 实现多网络接口的管理。同时，sriov-device-plugin 确保了正确的设备资源分配。

在实际应用中，建议根据具体需求调整配置，并进行详细的性能测试以确保系统性能达到最佳状态。通过这种组合，我们能够在容器化环境中更好地满足对高吞吐量和

灵活网络配置的需求。在结合使用这些工具时，请密切关注官方文档和社区更新，以获取最新的功能和性能优化。