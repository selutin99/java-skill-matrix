# Концепция service mesh, какие задачи она решает и какие есть реализации. Как работает istio

## 1. Концепция Service Mesh

* **Service Mesh** — это инфраструктурный слой, который берёт на себя сетевое взаимодействие между микросервисами.
* Идея: вынести коммуникацию из кода приложения в **sidecar-прокси** (обычно Envoy).
* Разработчик пишет бизнес-логику, а mesh решает задачи: маршрутизация, безопасность, наблюдаемость.

👉 Обычно реализуется как **sidecar pattern**: рядом с Pod запускается прокси (Envoy), через него идут все входящие/исходящие запросы.

---

## 2. Задачи, которые решает service mesh

1. **Маршрутизация и балансировка**

    * Роутинг по версиям (A/B, canary).
    * Балансировка нагрузки.

2. **Безопасность**

    * mTLS (все сервисы общаются по зашифрованным каналам).
    * Аутентификация и авторизация на сетевом уровне (RBAC).

3. **Наблюдаемость (observability)**

    * Метрики (latency, error rate, RPS).
    * Логи трафика.
    * Трассировка (traceId/SpanId автоматически прокидываются).

4. **Управление трафиком**

    * Ограничение скорости (rate limiting).
    * Ретраи, timeouts, circuit breaker.
    * Fault injection (тестирование отказоустойчивости).

5. **Политики и безопасность**

    * Centralized policy enforcement.
    * Сетевые ACL для сервисов.

---

## 3. Реализации Service Mesh

* **Istio** — самая популярная (Envoy как data plane).
* **Linkerd** — лёгкая, упрощённая альтернатива (свой прокси на Rust/Go).
* **Consul Connect** (HashiCorp) — mesh на базе Consul.
* **Kuma** (от Kong) → CNCF проект.
* **AWS App Mesh**, **Google Anthos Service Mesh**, **OpenShift Service Mesh** — managed-решения.

---

## 4. Как работает Istio

### 4.1. Архитектура

* **Data plane**: Envoy sidecar в каждом Pod.
* **Control plane**: компоненты Istio, которые управляют конфигурацией.

Основные компоненты:

* **Pilot** — конфигурация маршрутизации, сервис discovery.
* **Citadel** — безопасность: выдаёт TLS-сертификаты для mTLS.
* **Mixer** (устарел, часть вынесена) — политики и телеметрия.
* **Istiod** (в современных версиях объединяет Pilot, Citadel, Galley).

### 4.2. Принцип работы

1. Запускаем Pod → к нему автоматически (через admission webhook) добавляется Envoy sidecar.
2. Все запросы проходят через Envoy.
3. Envoy конфигурируется control plane’ом (Istiod).
4. Маршрутизация, ретраи, TLS и метрики применяются прозрачно для приложения.

### 4.3. Пример использования (Kubernetes)

**Deployment (app v1, v2):**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-v1
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: myapp
        version: v1
    spec:
      containers:
        - name: app
          image: myapp:1.0
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-v2
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: myapp
        version: v2
    spec:
      containers:
        - name: app
          image: myapp:2.0
```

**VirtualService (маршрутизация 90% → v1, 10% → v2):**

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
    - myapp.default.svc.cluster.local
  http:
    - route:
        - destination:
            host: myapp.default.svc.cluster.local
            subset: v1
          weight: 90
        - destination:
            host: myapp.default.svc.cluster.local
            subset: v2
          weight: 10
```

**DestinationRule (определяем версии):**

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: myapp
spec:
  host: myapp.default.svc.cluster.local
  subsets:
    - name: v1
      labels:
        version: v1
    - name: v2
      labels:
        version: v2
```

👉 Это позволяет делать **canary release без изменения приложения**.

### 4.4. Observability в Istio

* Автоматически собираются метрики (Prometheus).
* Grafana дашборды (latency, RPS, error rate).
* Трассировка (Jaeger/Tempo) с traceId автоматически.
* Kiali — визуализация mesh-графа.

---

## 5. Выжимка для собеседования

* **Service Mesh** — инфраструктурный слой для сервис-2-сервис коммуникаций (обычно sidecar-прокси).
* **Решает задачи**: маршрутизация, балансировка, mTLS, наблюдаемость (метрики/логи/трейсы), политики (ACL, rate limiting).
* **Популярные реализации**: Istio (самая функциональная), Linkerd (лёгкая), Consul Connect, Kuma, AWS App Mesh.
* **Istio**:

    * Data plane → Envoy sidecar.
    * Control plane → Istiod (Pilot + Citadel + др.).
    * Возможности: canary/blue-green, retries, circuit breaking, fault injection, observability.
* **Пример**: VirtualService + DestinationRule → управление трафиком между версиями сервиса.
* **Главная идея**: приложение не знает о mesh, всё решает инфраструктура.
