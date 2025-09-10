# Как работает кластер k8s, из каких компонентов он состоит, что такое кастомные ресурсы

## 1. Как работает кластер Kubernetes

* **Кластер k8s** = *control plane* (управление) + *worker nodes* (рабочие узлы).
* Control plane решает **где и как** запускать Pods.
* Worker node запускает контейнеры и сообщает о своём состоянии.
* Все ресурсы кластера описываются в **etcd** (распределённое key-value хранилище).

👉 Схема:

1. DevOps пишет манифест (YAML).
2. `kubectl apply -f` → объект сохраняется в etcd.
3. Control plane планирует (scheduler).
4. Node запускает Pod через контейнерный runtime.
5. Kubelet и Controller Manager следят за соответствием **desired state** ↔ **current state**.

---

## 2. Основные компоненты кластера

### 2.1. Control Plane

* **API Server** (`kube-apiserver`)

    * Центральная точка входа, принимает запросы (`kubectl`, контроллеры, UI).
    * Работает с etcd.

* **etcd**

    * Распределённое key-value хранилище.
    * Хранит все объекты кластера (Pods, Services, Deployments).

* **Scheduler**

    * Определяет, на какой node запустить Pod.
    * Учитывает ресурсы (CPU, RAM), taints/tolerations, affinity/anti-affinity.

* **Controller Manager**

    * Следит за объектами → доводит текущий state до желаемого.
    * Примеры контроллеров: Node Controller, Deployment Controller, ReplicaSet Controller.

---

### 2.2. Worker Node

* **Kubelet**

    * Агент на каждом узле.
    * Запускает контейнеры через CRI (Container Runtime Interface, напр. containerd).
    * Отчитывается в API Server.

* **Kube-proxy**

    * Настраивает сетевые правила (iptables/ipvs).
    * Реализует Service → Pod маршрутизацию.

* **Container Runtime**

    * Реальный запуск контейнеров (Docker, containerd, CRI-O).

---

### 2.3. Дополнительные аддоны

* **CoreDNS** — DNS внутри кластера.
* **Ingress Controller** (nginx, traefik) — внешние маршруты.
* **Dashboard/metrics-server** — метрики и UI.

---

## 3. Кастомные ресурсы (CRD)

### 3.1. Что это

* **CRD (CustomResourceDefinition)** — механизм расширения API Kubernetes.
* Позволяет добавлять свои ресурсы (например, `Database`, `Backup`, `KafkaCluster`).

### 3.2. Как это работает

1. Создаём CRD → в API появляется новый **kind**.
2. Можно применять YAML с новым типом.
3. Логика управляется **оператором** (controller), который реагирует на CRD.

### 3.3. Пример CRD

**crd.yaml**

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.mycompany.com
spec:
  group: mycompany.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                engine:
                  type: string
                size:
                  type: string
  scope: Namespaced
  names:
    plural: databases
    singular: database
    kind: Database
```

**database.yaml**

```yaml
apiVersion: mycompany.com/v1
kind: Database
metadata:
  name: mydb
spec:
  engine: postgres
  size: small
```

👉 Теперь `kubectl get databases` будет работать как для нативных ресурсов.

### 3.4. Операторы

* Контроллеры, которые управляют CRD (например, **Postgres Operator**).
* Реализуют логику: при создании `Database` создаётся StatefulSet, PVC, Service.

---

## 4. Выжимка для собеседования

* **Кластер k8s** = control plane + worker nodes.
* **Control plane**: API Server (точка входа), etcd (хранилище), Scheduler (где запускать Pod), Controller Manager (следит за состоянием).
* **Worker node**: Kubelet (агент), Kube-proxy (сеть), Container Runtime (containerd/Docker/CRI-O).
* **Core принцип**: desired state (из манифеста) ↔ current state (в кластере).
* **CRD**: расширение API → новые ресурсы (`kind: Database`), управляемые контроллерами/операторами.
* **Операторы**: автоматизируют жизненный цикл приложений (БД, кэши, кластеры).
