# k8s

Try to run
- JupyterHub for Kubernetes
- nbgrader
- own conda/pip deps
- run containerized ewatercycle models
- inside vagrant with hyper-v
- micro8ks as kubernetes deployment
- mount dcache

## Boot

```shell
vagrant up
```

https://jet.dev/blog/spin-up-local-kubernetes-cluster-agrant/

```
vagrant ssh microk8s_a
ip route | grep default | grep eth0 | cut -d' ' -f9
172.26.149.178
vagrant ssh microk8s_b
ip route | grep default | grep eth0 | cut -d' ' -f9
172.26.146.78

#a
sudo -i
echo "172.26.146.78 microk8s-b" >> /etc/hosts
exit

microk8s add-node

#b
microk8s join 172.26.149.178:25000/ca5b88a8fce3cd45bff6aa8eb435d140/699b0df3a535

#a
microk8s kubectl get nodes

# Use range inside hyperv default switch
microk8s enable metallb:172.26.145.83-172.26.145.93
microk8s enable hostpath-storage
 microk8s enable rbac # As advised at https://z2jh.jupyter.org/en/stable/administrator/security.html#use-role-based-access-control-rbac
```
# NFS

https://microk8s.io/docs/how-to-nfs

```
# on a
sudo apt update
sudo apt-get install -y nfs-kernel-server
sudo mkdir -p /srv/nfs
sudo chown nobody:nogroup /srv/nfs
sudo chmod 0777 /srv/nfs
sudo mv /etc/exports /etc/exports.bak
echo '/srv/nfs 172.0.0.0/8(rw,sync,no_subtree_check)' | sudo tee /etc/exports
sudo systemctl restart nfs-kernel-server
microk8s enable helm3
microk8s helm3 repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
microk8s helm3 repo update
microk8s helm3 install csi-driver-nfs csi-driver-nfs/csi-driver-nfs \
    --namespace kube-system \
    --set kubeletDir=/var/snap/microk8s/common/var/lib/kubelet
microk8s kubectl wait pod --selector app.kubernetes.io/name=csi-driver-nfs --for condition=ready --namespace kube-system
microk8s kubectl get csidrivers
```

```
 cat sc-nfs.yaml
# sc-nfs.yaml
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-csi
provisioner: nfs.csi.k8s.io
parameters:
  server: 172.26.149.178
  share: /srv/nfs
reclaimPolicy: Delete
volumeBindingMode: Immediate
mountOptions:
  - hard
  - nfsvers=4.1
```

```
microk8s kubectl apply -f - < sc-nfs.yaml
```

```
# pvc-nfs.yaml
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  storageClassName: nfs-csi
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 5Gi
```

```
microk8s kubectl apply -f - < pvc-nfs.yaml
microk8s kubectl describe pvc my-pvc
```

After login to JH /home/shared is mounted nfs volume

# Setup JH

https://z2jh.jupyter.org/en/stable/jupyterhub/installation.html

Use namespace `teach` and helm release name `teach1`.

```
touch config.yaml
microk8s helm repo add jupyterhub https://hub.jupyter.org/helm-chart/
microk8s helm repo update
microk8s helm upgrade --cleanup-on-fail \
  --install teach1 jupyterhub/jupyterhub \
  --namespace teach \
  --create-namespace \
  --version=3.2.1 \
  --values config.yaml

microk8s kubectl config set-context $(microk8s kubectl config current-context) --namespace teach
microk8s kubectl --namespace teach get service proxy-public
 NAME           TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
proxy-public   LoadBalancer   10.152.183.208   172.19.231.83   80:30233/TCP   2m51s
```

Login on http://172.19.231.83 with any username:password

# Upgrade config.yaml


```
microk8s helm upgrade --cleanup-on-fail \
  teach1 jupyterhub/jupyterhub \
  --namespace teach \
  --version=3.2.1 \
  --values config.yaml
```

# Dashboard

```
microk8s dashboard-proxy
```

Goto public ip of node A with port+token in console output.

# ngshare

https://ngshare.readthedocs.io/en/latest/user_guide/install_z2jh.html

```
microk8s helm repo add ngshare https://libretexts.github.io/ngshare-helm-repo/
microk8s helm repo update
```

```
 cat config.ngshare.yaml
deployment:
  # Resource limitations for the pod
  resources:
    limits:
      cpu: 100m
      memory: 128Mi
    requests:
      cpu: 100m
      memory: 128Mi

ngshare:
  hub_api_token: demo_token_9wRp0h4BLzAnC88jjBfpH0fa4QV9tZNI
  # Please change the line below with the namespace your Z2JH helm chart is installed under
  # You can omit this value if you're installing ngshare in the same namespace
  hub_api_url: http://hub.teach.svc.cluster.local:8081/hub/api
  admins:
    - sverhoeven

pvc:
  # Amount of storage to allocate
  storage: 1Gi
  ```

```
microk8s helm install ngshare ngshare/ngshare  --namespace teach -f  config.ngshare.yaml
```

## Container in container

```
# fuse-device-plugin.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
 name: fuse-device-plugin-daemonset
 namespace: kube-system
spec:
 selector:
   matchLabels:
     name: fuse-device-plugin-ds
 template:
   metadata:
     labels:
       name: fuse-device-plugin-ds
   spec:
     hostNetwork: true
     containers:
     - image: soolaugust/fuse-device-plugin:v1.0
       name: fuse-device-plugin-ctr
       securityContext:
         allowPrivilegeEscalation: false
         capabilities:
           drop: ["ALL"]
       volumeMounts:
         - name: device-plugin
           mountPath: /var/lib/kubelet/device-plugins
     volumes:
       - name: device-plugin
         hostPath:
           path: /var/lib/kubelet/device-plugins
     imagePullSecrets:
       - name: registry-secret
```

```
microk8s kubectl apply -f - < fuse-device-plugin.yaml
```

```
apptainer run docker://alpine:latest cat /etc/os-release
apptainer run docker://ghcr.io/ewatercycle/leakybucket-grpc4bmi:v0.0.1
```

```python
from grpc4bmi.bmi_client_apptainer import BmiClientApptainer
model = BmiClientApptainer('docker://ghcr.io/ewatercycle/leakybucket-grpc4bmi:v0.0.1', work_dir='/tmp')
model.get_component_name()
del model
```

# TODO

- ewatercycle image
- nbgrader with ngshare
- for ngshare-course-management cli create web gui, like a jupyterlab extension, as cli might be too complex for teachers
- instead of ngshare use nfs volume thats read/write for all users. Is less secure but easier to use.
- run model in container
  - singleuser container can run apptainer/podman inside
  - https://www.redhat.com/sysadmin/podman-inside-kubernetes
- mount dcache
  https://github.com/wunderio/csi-rclone
  https://github.com/simplyzee/kube-rclone
