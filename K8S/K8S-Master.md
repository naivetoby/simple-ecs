#### 初始化 k8s-master

~~~bash
kubeadm config print init-defaults > kubeadm-config.yaml

# 修改(以及新增)kubeadm-config.yaml以下内容
localAPIEndpoint:
  advertiseAddress: 192.168.5.153
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  imagePullPolicy: IfNotPresent
  name: k8s-master
  taints: null
kubernetesVersion: v1.23.0
networking:
  dnsDomain: cluster.local
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12

# 新增
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: ipvs

# 初始化, 并且将标准输出同时写入至 kubeadm-init.log 文件
kubeadm init --config=kubeadm-config.yaml | tee kubeadm-init.log

# 修改 kube-proxy mode 为 ipvs
kubectl edit configmap kube-proxy -n kube-system
mode: ipvs

kubectl get po -n kube-system
kubectl delete po -n kube-system <pod-name>

# 列出使用的镜像以及版本号(如果代理可以解决，则忽略)
kubeadm config images list --config kubeadm-config.yaml

# 编写脚本 docker-download.sh(如果代理可以解决，则忽略)
---
#!/bin/bash

images=(
    kube-apiserver:v1.23.1
    kube-controller-manager:v1.23.1
    kube-scheduler:v1.23.1
    kube-proxy:v1.23.1
    pause:3.6
    etcd:etcd:3.5.1-0
    coredns:1.8.6
)

for imageName in ${images[@]} ; do
    docker pull k8simage/$imageName
    docker tag k8simage/$imageName k8s.gcr.io/$imageName
    docker rmi k8simage/$imageName
done
---

# 执行(如果代理可以解决，则忽略)
chmod +x docker-download.sh
./docker-download.sh

# 执行日志中的指令 (root用户直接改环境变量就可以了)
# mkdir -p $HOME/.kube
# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
# sudo chown $(id -u):$(id -g) $HOME/.kube/config
# export KUBECONFIG=/etc/kubernetes/admin.conf

# 主节点：
# 复制 admin.conf, 请在主节点服务器上执行此命令
# scp /etc/kubernetes/admin.conf node1:/etc/kubernetes/admin.conf
# scp /etc/kubernetes/admin.conf node2:/etc/kubernetes/admin.conf

# 从节点：
# 设置 kubeconfig 文件
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> /etc/profile
source /etc/profile

# 下载 flannel 配置文件
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
# 安装flannel
kubectl create -f kube-flannel.yml

# 常用命令

# 加入子节点
kubeadm join 192.168.5.153:6443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:673e58a2a93d03c0ea7520780f84aa0b1d02c2352c64c429304bc83a17eca6c5

# 如果无法加入, 可能是 token 过期了, 重新生成不会过期的 token
kubeadm token create --ttl 0

# 如果出现镜像拉取失败的错误, 可能需要重新添加 docker 代理
mkdir /etc/systemd/system/docker.service.d
vim /etc/systemd/system/docker.service.d/http_proxy.conf
---
[Service]
Environment="HTTP_PROXY=http://192.168.5.200:7890"
Environment="HTTPS_PROXY=http://192.168.5.200:7890"
---
systemctl daemon-reload && systemctl restart docker
# 查看 docker 代理是否生效
systemctl show --property=Environment docker

# 删除子节点
kubectl delete nodes k8s-node01
kubectl delete nodes k8s-node01

# 节点重置
kubeadm reset

# 任意节点查看集群状态
kubectl get pod

# 任意节点查看 pod 信息
kubectl get pod --all-namespaces

# 查看命名空间为 kube-system 的 pod 情况
kubectl get pod -n kube-system

# 查看更详细的信息
kubectl get pod -n kube-system -o wide

# 查看 Pod 详情
kubectl describe pod {pod-name} -n {namespace}

# 安装 Dashboard

# 生产自定义证书
mkdir key
cd key
openssl genrsa -out dashboard.key 2048
openssl req -new -out dashboard.csr -key dashboard.key -subj '/CN=192.168.5.153'
openssl x509 -req -in dashboard.csr -signkey dashboard.key -out dashboard.crt

# 获取资源清单
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml -O kubernetes-dashboard.yaml

# 修改 kubernetes-dashboard.yaml 里的端口, 外网开放
---
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort      #新增type类型为NodePort
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30002   #设置nodeport 端口
  selector:
    k8s-app: kubernetes-dashboard

args:
  - --auto-generate-certificates
  - --namespace=kubernetes-dashboard
  - --tls-key-file=dashboard.key # 设置自定义证书
  - --tls-cert-file=dashboard.crt # 设置自定义证书
---

# 根据资源清单创建 dashboard
kubectl create secret generic kubernetes-dashboard-certs --from-file=/root/key/dashboard.crt --from-file=/root/key/dashboard.key -n kubernetes-dashboard
kubectl apply -f kubernetes-dashboard.yaml

# 如果处于 running 状态，要在看一下 log 日志
kubectl logs  -f  kubernetes-dashboard-6bd77794f-5l67x -n kubernetes-dashboard
kubectl logs  -f  dashboard-metrics-scraper-577dc49767-9h22w  -n kubernetes-dashboard

# 创建管理员账户登录 dashboard
kubectl create serviceaccount admin

# 绑定组
kubectl create clusterrolebinding dash-admin --clusterrole=cluster-admin --serviceaccount=default:admin

# 创建密钥
secret=$(kubectl get sa admin -o jsonpath='{.secrets[0].name}')

# 获得token
kubectl get secret $secret -o go-template='{{ .data.token | base64decode }}'

# 删除原有证书(如果没有遇到问题, 可以忽略)
kubectl delete secret kubernetes-dashboard-certs -n kubernetes-dashboard
# 删除原有容器(如果没有遇到问题, 可以忽略)
kubectl get pod -n kubernetes-dashboard | grep -v NAME | awk '{print "kubectl delete po " $1 " -n kubernetes-dashboard"}' | sh

# 安装 Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# 创建默认 Storage
vim kube-local-storage.yaml
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-sc
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
---
kubectl apply -f kube-local-storage.yaml

# 安装 Portainer
mkdir portainer
# 创建默认 Storage
vim portainer-storage.yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: portainer-pv
  labels:
    type: local
spec:
  storageClassName: local-sc
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/portainer-storage"
---
kubectl apply -f portainer-storage.yaml
# 如果发生错误, 可以删除
kubectl delete -f portainer-storage.yaml

helm repo add portainer https://portainer.github.io/k8s/
helm repo update
helm install --create-namespace -n portainer portainer portainer/portainer

# 如果发生错误, 可以删除
helm delete portainer -n portainer

# 安装 Kubesphere
mkdir kubesphere
# 创建默认 Storage
vim kubesphere-storage.yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: kubesphere-pv-01
  labels:
    type: local
spec:
  storageClassName: local-sc
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/kubesphere-storage-01"
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: kubesphere-pv-02
  labels:
    type: local
spec:
  storageClassName: local-sc
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/kubesphere-storage-02"
---
kubectl apply -f kubesphere-storage.yaml
# 如果发生错误, 可以删除
kubectl delete -f kubesphere-storage.yaml

kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.2.1/kubesphere-installer.yaml
kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.2.1/cluster-configuration.yaml
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
kubectl get svc/ks-console -n kubesphere-system

# 如果发生错误, 可以删除
wget https://raw.githubusercontent.com/kubesphere/ks-installer/release-3.1/scripts/kubesphere-delete.sh
chmod +x kubesphere-delete.sh
./kubesphere-delete.sh
~~~