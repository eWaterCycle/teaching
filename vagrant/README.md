# k8s

Try to run
- JupyterHub for Kubernetes
- nbgrader
- own conda/pip deps
- run containerized ewatercycle models
- inside vagrant with hyper-v
- micro8ks as kubernetes deployment

## Boot

```shell
vagrant up
```

https://jet.dev/blog/spin-up-local-kubernetes-cluster-agrant/

```
vagrant ssh microk8s_a
ip route | grep default | grep eth0 | cut -d' ' -f9
172.19.226.152
vagrant ssh microk8s_b
ip route | grep default | grep eth0 | cut -d' ' -f9
172.19.234.158

#a
sudo -i
echo "172.19.234.158 microk8s-b" >> /etc/hosts
exit

microk8s add-node

#b
microk8s join 172.19.226.152:25000/13a97d6ef692d3eadb078866e78f2acd/6da73c5b9623

#a
microk8s kubectl get nodes

microk8s enable metallb
# Use range inside hyperv default switch
172.19.231.83-172.19.231.93
microk8s enable hostpath-storage
```
# NFS

https://microk8s.io/docs/how-to-nfs

```
sudo apt-get install nfs-kernel-server
sudo mkdir -p /srv/nfs
sudo chown nobody:nogroup /srv/nfs
sudo chmod 0777 /srv/nfs
sudo mv /etc/exports /etc/exports.bak
echo '/srv/nfs 172.19.0.0/16(rw,sync,no_subtree_check)' | sudo tee /etc/exports
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
  server: 172.19.226.152
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

# Update config.yaml


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

# TODO

- ewatercycle image
- nbgrader
- run model in container
  - singleuser container can run apptainer/podman inside
  - https://www.redhat.com/sysadmin/podman-inside-kubernetes