# Задание 1

- Используем регион Yandex Cloud с двумя зонами доступности.
- Каждый сервис разворачивается минимум в двух репликах, распределённых по зонам.
- Статика отдаётся через CDN (например, Yandex Cloud CDN).
- Для динамических запросов — геобалансировка на уровне DNS или HTTP-балансировщика (Application Load Balancer Yandex Cloud), который умеет направлять трафик в ближайший бэкенд.

[схема в drawio](./task-1/start_scheme-1.drawio)
![схема](./task-1/start_scheme.drawio.png)


# Задание 2

## Часть 1
Собираем образ `scaletestapp` из репозитория `https://github.com/Yandex-Practicum/scaletestapp`

Устанавливаем locust

```bash
minikube start --memory=4096 --cpus=2

minikube image load scaletestapp

minikube addons enable metrics-server

kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f hpa-memory.yaml

kubectl apply -f servicemonitor.yaml

minikube service test-app-service --url

locust -f locustfile.py --host http://[service_url]:[service_port]

```

Получили увеличение подов
![image-1](./task-2/image-1.png)
![image-2](./task-2/image-2.png)

## Часть 2

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace

helm install prometheus-adapter prometheus-community/prometheus-adapter \
--set prometheus.url=http://prometheus-server.smonitoring.svc.cluster.local \
--namespace monitoring

kubectl apply -f hpa-rps.yaml

kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1/namespaces/default/pods/*/http_requests_per_second" | jq .
```

Получили увеличение подов
![image-3](./task-2/image-3.png)
![image-4](./task-2/image-4.png)


# Задание 3