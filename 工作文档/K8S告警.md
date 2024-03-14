
## 简介

>   K8S 告警使用统一的一套helm进行部署， 简化实现快捷方便
>   
>   ⚠️： 告警依赖于 kubesphere， 需要开启其 altert组件，同时针对不同版本需要对配置作修改

项目地址：[brtc-monitor](https://git.baijiashilian.com/cloud/BRTC/brtc-monitor)


## 部署指导


### 一. 启用kubesphere自带的alert服务

1. 修改自定义资源中的 clusterconfigurations.ks-installer 文件
2. 修改为 alerting.enabled = true：
```
spec:
  alerting:
    enabled: true # 启用
```
3. 删除 kubesphere-monitoring-system 下面的 alermanager-main 的secret

```
kubectl delete secret alermanager-main -n kubesphere-monitoring-system
```

4. 等待crd自动部署， 会影响一定功能，需要等没有人

### 二. 新增helm

1. 创建namespace
```
kubectl create ns brtc-monitoring
```

2. 安装监控

```
helm install brtc-monitoring ./ --values=values.yaml  -n brtc-monitoring
```

3. 更新监控

```
helm upgrade brtc-monitoring ./ --values=values.yaml  -n brtc-monitoring
```

### 三. kubesphere 安装

1. 启用 kubesphere 的 alert 插件

![image.png](https://img.baijiayun.com/0ewiki/attachments/a05a4d68872644d3ce934f89e6eab9c1.png)

![image.png](https://img.baijiayun.com/0ewiki/attachments/16373ed8dd630ddf3ca0a37cb04f08e2.png)


修改 ks-installer 之后等待自动同步


2.  待第一步完成后， 在自定义资源中查找 ThanosRuler， 并且查看 kubesphere 

![image.png](https://img.baijiayun.com/0ewiki/attachments/1e6b55210d1de0f28d175c9b99f4edd2.png)

![image.png](https://img.baijiayun.com/0ewiki/attachments/180cd0aed53859c1995f983d4d2f56a6.png)


3. 在选中的区域的 values 中修改  customRuleLabels 和 2 对齐

![image.png](https://img.baijiayun.com/0ewiki/attachments/8f15c24bac20c56072c1ae004cb2b382.png)

4. 删除 旧版的 alertmager-main 的 secret

```
kubectl delete secret alertmanager-main -n kubesphere-monitoring-system
```


5.  执行 helm 安装告警