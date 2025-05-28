# K8GB on kubernetes

参考：

https://mp.weixin.qq.com/s/BM51ysd6F6XBQQiMnKn_GA


镜像列表

```
docker pull absaoss/k8gb:v0.14.0
docker pull registry.k8s.io/external-dns/external-dns:v0.15.1
docker pull absaoss/k8s_crd:v0.1.2

registry-dev.21vianet.com/system_containers/k8gb:v0.15.0 镜像ID：2ced6b6423b6
registry-dev.21vianet.com/system_containers/external-dns:v0.15.1 镜像ID：d5b39e9ba673
registry-dev.21vianet.com/system_containers/k8s_crd:v0.1.2 镜像ID：7d495260bb4e
```

## 安装部署

### 创建初始 init ingress

ingress 需要添加 k8gb.io/ip-source: "true" label，否则 k8gb operator 起不来, 或者 CoreDNS Service 使用 loadBlance

```bash
[~]$ kubectl create -f /tmp/aaa.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  labels:
    app: init-ingress
    k8gb.io/ip-source: "true"
  name: init-ingress
  namespace: k8gb
spec:
  ingressClassName: nginx
  rules:
    - host: init.cloud.example.com
      http:
        paths:
          - backend:
              service:
                name: k8gb-coredns
                port:
                  name: udp-5353
            path: /
            pathType: Prefix
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
