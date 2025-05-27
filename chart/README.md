# K8GB on kubernetes

参考：

https://mp.weixin.qq.com/s/BM51ysd6F6XBQQiMnKn_GA


镜像列表

```
docker pull absaoss/k8gb:v0.14.0
docker pull ghcr.io/k8gb-io/external-dns:v0.13.4-azure-ns-multiarch
docker pull absaoss/k8s_crd:v0.1.2

registry-dev.21vianet.com/system_containers/k8gb:v0.14.0 镜像ID：110e6c7bb4f9
registry-dev.21vianet.com/system_containers/external-dns:v0.13.4 镜像ID：96af04f8b32a
registry-dev.21vianet.com/system_containers/k8s_crd:v0.1.2 镜像ID：7d495260bb4e
```

## 安装部署

### 创建 rfc2136 secret

获取 bind9 dns 服务端的 tsig key，生成一个 secret

```bash
kubectl -n k8gb create secret generic rfc2136 --from-literal=secret='96Ah/a2g0/nLeFGK+d/0tzQcccf9hCEIy34PoXX2Qg8='
```

### 通过 helm 包进行安装

* helm 目录内安装
```bash
helm -n k8gb upgrade -i k8gb . --wait --timeout=10m0s
```

* helm tar 包安装
```bash
helm -n k8gb install k8gb ./k8gb-v0.14.0.tgz -f "" \
        --set k8gb.clusterGeoTag='BeiJing' --set k8gb.extGslbClustersGeoTags='ShangHai' \
        --set k8gb.edgeDNSZone=example.com \
        --set k8gb.dnsZone=example.com \
        --set k8gb.imageRepo=registry-dev.21vianet.com/system_containers/k8gb \
        --set k8gb.imageTag=v0.14.0 \
        --set k8gb.coreDNSServer=172.22.222.108 \
        --set-string k8gb.coreDNSPort="30935" \
        --set externaldns.image=registry-dev.21vianet.com/system_containers/external-dns:v0.13.4 \
        --set coredns.image.repository=registry-dev.21vianet.com/system_containers/k8s_crd \
        --set coredns.image.tag=v0.1.2 \
        --set rfc2136.enabled=true \
        --set rfc2136.rfc2136Opts[0].host=172.22.222.102 \
        --set rfc2136.rfc2136Opts[1].port=53 \
        --set k8gb.edgeDNSServers[0]=172.22.222.102:53

helm -n k8gb install k8gb ./k8gb-v0.14.0.tgz -f "" \
        --set k8gb.clusterGeoTag='ShangHai' --set k8gb.extGslbClustersGeoTags='BeiJing' \
        --set k8gb.edgeDNSZone=example.com \
        --set k8gb.dnsZone=example.com \
        --set k8gb.imageRepo=registry-dev.21vianet.com/system_containers/k8gb \
        --set k8gb.imageTag=v0.14.0 \
        --set k8gb.coreDNSServer=172.22.222.117 \
        --set-string k8gb.coreDNSPort="30935" \
        --set externaldns.image=registry-dev.21vianet.com/system_containers/external-dns:v0.13.4 \
        --set coredns.image.repository=registry-dev.21vianet.com/system_containers/k8s_crd \
        --set coredns.image.tag=v0.1.2 \
        --set rfc2136.enabled=true \
        --set rfc2136.rfc2136Opts[0].host=172.22.222.102 \
        --set rfc2136.rfc2136Opts[1].port=53 \
        --set k8gb.edgeDNSServers[0]=172.22.222.102:53
```
