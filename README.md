# Webinar Itaú-Inmetrics - Construindo painéis de observabilidade

# Ambiente
## Provedor Cloud
- AWS

## Sistema Operacional
- Ubuntu 20.04 - 

## Kubernetes
- Minikube v1.20.2
- Kubectl v1.21.0
- Helm v3.5.4
- Kompose

### Helm Charts
- Loki-Stack loki-stack-2.4.0
- Prometheus Stacl kube-prometheus-stack-15.4.6

# Implementação

## Passos

### 1 - Inicializando Kubernetes

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
dpkg -i minikube_latest_amd64.deb
apt-get remove docker docker-engine docker.io containerd runc
apt-get update
apt-get install     apt-transport-https     ca-certificates     curl     gnupg     lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo   "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update
apt-get install docker-ce docker-ce-cli containerd.io
systemctl enable docker.service
systemctl enable containerd.service
docker ps
curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
apt-get update
apt-get install -y kubectl
minikube start --cpus=4 --memory=8g
minikube addons enable ingress
minikube addons enable ingress-dns

### Helm
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

## 2- Instalando Jaeger

Dentro da pasta yager execute o comando abaixo
```bash
kubectl apply -f app.yml
```

### 3 - Instalando Prometheus Stack
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm upgrade --install prometheus prometheus-community/kube-prometheus-stack --values=prom-oper-values.yaml
```

## 4 - Instalando a Aplicação Demo
Dentro da pasta hotrod execute o comando abaixo
```bash
kubectl apply -f app.yml
```

### 5 - Instalando Loki Stack
```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm upgrade --install prometheus prometheus-community/kube-prometheus-stack --values=prom-oper-values.yaml
```


## Acessando o Grafana

kubectl port-forward --address 0.0.0.0 svc/prometheus-grafana 3000:80 -n default

## Acessando o HotRod

kubectl port-forward --address 0.0.0.0 svc/hotrod 8080:8080 -n default


## Implementando Tracing em suas aplicações

### Linguagem Go
```go
func (b Factory) For(ctx context.Context) Logger {
	if span := opentracing.SpanFromContext(ctx); span != nil {
		logger := spanLogger{span: span, logger: b.logger}

		if jaegerCtx, ok := span.Context().(jaeger.SpanContext); ok {
			logger.spanFields = []zapcore.Field{
				zap.String("trace_id", jaegerCtx.TraceID().String()),
				zap.String("span_id", jaegerCtx.SpanID().String()),
			}
		}

		return logger
	}
	return b.Bg()
}
```

### Linguagem Python
```python
with tracer.start_span('fourth-span') as span4:
    span4.set_tag('fourth-tag', '60')
    with tracer.start_span('fifth-span', child_of=span4) as span5:
        span5.set_tag('fifth-tag', '80')
```