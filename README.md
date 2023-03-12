# Examensarbete
Det här kodförråd är för mitt examensarbet som ingår kongfiguration filer för Helm Charts som jag använde i arbetet. 

- prometheus-values.yaml
    ändras för att ServiceMonitors hittas i default     
- trivy-values.yaml
    värder för att använda Trviy Operator 
- service.yaml 

### Verktyg 
Verktyg som jag använde för arbetet är: 
 - kind v1.23.1 
 - Trviy Operator v 0.11.1
 - kube-prometheus-stack
    - Prometheus 
    - Grafana 
 - Lens 

### Installation 
Så här installerade jag alla verktyg 

#### kind
```
kind create cluster --image kindest/node:v1.23.1 --config - <<EOF
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: examensarbete 
networking: 
  ipFamily: dual 
  apiServerAddress: 127.0.0.1  
kubeadmConfigPatches:
- |-
  kind: ClusterConfiguration
  # configure controller-manager bind address
  controllerManager:
    extraArgs:
      bind-address: 0.0.0.0
  # configure etcd metrics listen address
  etcd:
    local:
      extraArgs:
        listen-metrics-urls: http://0.0.0.0:2381
  # configure scheduler bind address
  scheduler:
    extraArgs:
      bind-address: 0.0.0.0
- |-
  kind: KubeProxyConfiguration
  # configure proxy metrics bind address
  metricsBindAddress: 0.0.0.0
nodes:
- role: control-plane
- role: worker
EOF
```
#### kube-prometheus-stack  
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

```
helm repo update
```

```
helm install --wait --timeout 15m --namespace monitoring --create-namespace --repo http://prometheus-community.github.io./helm-charts prom kube-prometheus-stack --values prometheus-value.yaml 

```
#### Trivy-Operator 
```helm install trivy-operator aqua/trivy-operator \
  --namespace trivy-op \
  --create-namespace \
  --version 0.11.1 \
  --values trivy-values.yaml
```
På grund av problem med port-foward med service, jag skapade `service.yaml` file och gjorde följande också. 

```
kubectl delete service trivy-operator -n trivy-op
kubectl apply -f service.yaml -n trivy-op
```
### Använder Prometheus och Grafana dashboard
Prometheus 
```
kubectl port-forward service/prom-kube-prometheus-stack-prometheus -n monitoring 9090:9090
```

Grafana
```
kubectl port-forward service/prom-grafana -n monitoring 3000:80
```
har tillgång till Grafana dashboard med usernamn och lösenord. 
