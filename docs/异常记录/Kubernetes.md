## K8s HTTP GET 探针失败

问题

`kubectl create -f test.yaml` 命令创建资源失败。

`kubectl describe`描述如下：

```sql
Events:
Type Reason Age From Message

Normal Scheduled 10m default-scheduler Successfully assigned dev/test-app-12345 to node-06
Normal Pulled 10m kubelet Container image “harbor:5000/XXX” already present on machine
Normal Created 10m kubelet Created container copyfile
Normal Started 10m kubelet Started container copyfile
Normal Pulling 10m kubelet Pulling image “harbor:5000/XXX”
Normal Pulled 10m kubelet Successfully pulled image “harbor:5000/XXX” in 40.066135ms
Normal Created 10m kubelet Created container test-app
Normal Started 10m kubelet Started container test-app
Warning Unhealthy 9m44s kubelet Liveness probe failed: Get “<http://127.0.0.1:8080/actuator/health/liveness”:> dial tcp 127.0.0.1:8080: connect: connection refused
Warning Unhealthy 9m44s kubelet Readiness probe failed: Get “<http://127.0.0.1:8080/actuator/health/readiness”:> dial tcp 127.0.0.1:8080: connect: connection refused
```

原因：服务初始化未完成，容器启动后延迟 `10` 秒开始探测，探测失败

解决：`initialDelaySeconds`参数 配置长一点，例如`30`秒

- `initialDelaySeconds: 30` ，容器启动后延迟 `30` 秒开始探测
------
## nodes are available: 1 node(s) were unschedulable......

### 问题

`kubectl create -f test.yaml` 命令创建pod后，pod处于`pending`。

`kubectl describe`描述如下：

```bash
Events:
Type Reason Age From Message

Warning FailedScheduling 60s (x8 over 8m16s) default-scheduler 0/7 nodes are available: 1 node(s) were unschedulable, 6 node(s) didn’t match Pod’s node affinity/selector.
```

### 原因

根据提供的 `kubectl describe` 输出信息，Pod 处于 `Pending` 原因是调度器**无法找到合适的节点**来部署Pod。具体的原因有两个：

1. **1 个节点不可调度**：意味着一个节点被标记为不可调度（`unschedulable`），通常是因为节点已经被标记为 `cordoned` 或 `drained`，也可能是主节点（默认是不可调度）。
2. **6 个节点不符合 Pod 的节点亲和性/选择器要求**：意味着 Pod 的节点亲和性或节点选择器规则**不匹配**这 6 个节点的标签或其他属性。

### 解决：检查节点状态

**检查节点是否被标记为不可调度**：

- 查看节点列表：`kubectl get nodes`，看看是否有节点被标记为 `SchedulingDisabled`。
- 如果有，可以使用命令使节点可调度：`kubectl uncordon <node-name>`
- 注意，**主节点（master）默认是不可调度**的，例如：

```bash
[root@k8s-master01]# kubectl get nodes
NAME        STATUS                     ROLES    AGE    VERSION
master-01   Ready,SchedulingDisabled   master   112d   v1.23.8
node-01     Ready                      node     112d   v1.23.8
node-02     Ready                      node     112d   v1.23.8
node-03     Ready                      node     112d   v1.23.8
```

### 解决：检查节点亲和性和选择器

1. **检查 Pod 的节点亲和性和选择器**：

   - 查看 `test.yaml` 文件中是否定义了 `nodeSelector` 或 `nodeAffinity`。

   - 确保这些规则与集群中节点的标签匹配。

   - 例如，检查 `test.yaml` 中的以下部分：

     ```yaml
     spec:
       affinity:
         nodeAffinity:
           requiredDuringSchedulingIgnoredDuringExecution:
             nodeSelectorTerms:
             - matchExpressions:
               - key: <label-key>
                 operator: In
                 values:
                 - <label-value>
       nodeSelector:
         <label-key>: <label-value>
     ```

2. **检查节点标签**：`kubectl get nodes --show-labels`，例如：

   ```bash
   NAME        STATUS                     ROLES   LABELS
   master-01   Ready,SchedulingDisabled   master  ...,kubernetes.io/role=master
   node-01     Ready                      node    ...,feature2=webrtcs,feature3=etcd,feature=app,...,kubernetes.io/role=node
   node-02     Ready                      node    ...,feature2=influxdb,feature3=etcd,feature=app,...,kubernetes.io/role=node
   node-03     Ready                      node    ...feature2=edge_phone,feature3=etcd,feature=app,...,kubernetes.io/role=node
   ```

3. 确保**节点标签**与 Pod 的 `nodeSelector` 或 `nodeAffinity` 匹配。

4. **修改或删除不必要的亲和性规则**

   1. 如果 `nodeSelector` 或`nodeAffinity` 不必要，可以暂时移除这些规则以便进行测试。

### 其他检查

1. **资源限制**：检查是否有资源限制导致无法调度，比如 CPU 或内存需求超过节点可用资源。
2. **调度器策略**：确认调度器策略没有限制 Pod 的调度。
3. **调度器日志**：如果问题依然存在，可以查看 Kubernetes 调度器的日志以获得更多信息。