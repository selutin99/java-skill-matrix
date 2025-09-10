# Основные ресурсы k8s, как они описываются и для чего нужны

## 1. Основные ресурсы Kubernetes

### 1.1. Pod

* **Наименьшая сущность в k8s**, объединяет один или несколько контейнеров.
* Делит: сеть (IP), тома, namespace.
* Обычно не создают напрямую (лучше через Deployment/Job).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  containers:
    - name: app
      image: myapp:1.0
      ports:
        - containerPort: 8080
```

---

### 1.2. Deployment

* **Декларативное управление Pod’ами**.
* Обеспечивает **реплики**, **обновления** (rolling update).
* Используется для stateless сервисов.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: app
          image: myapp:1.0
          ports:
            - containerPort: 8080
```

---

### 1.3. Service

* **Сетевой доступ** к Pod’ам.
* Типы:

    * **ClusterIP** (по умолчанию) — доступ внутри кластера.
    * **NodePort** — доступ извне по `<node_ip>:<port>`.
    * **LoadBalancer** — через облачный балансировщик.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc
spec:
  selector:
    app: myapp
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

---

### 1.4. Ingress

* **Правила маршрутизации HTTP/HTTPS**.
* Работает через контроллер (nginx, traefik).
* Поддержка TLS.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: myapp-svc
                port:
                  number: 80
```

---

### 1.5. ConfigMap

* Хранение **настроек и конфигов**.
* Подключение как переменные окружения или файлы.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
data:
  SPRING_PROFILES_ACTIVE: prod
```

---

### 1.6. Secret

* Хранение **секретных данных** (пароли, ключи, токены).
* Кодируются base64 (но это не шифрование!).

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: dXNlcg==   # base64(user)
  password: cGFzcw==   # base64(pass)
```

---

### 1.7. Volume / PersistentVolumeClaim

* **Volume** — подключение хранилища в Pod.
* **PVC (PersistentVolumeClaim)** — запрос на диск у администратора/динамического провайдера.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-data
spec:
  accessModes: [ "ReadWriteOnce" ]
  resources:
    requests:
      storage: 1Gi
```

---

### 1.8. Namespace

* Логическая изоляция ресурсов внутри кластера.
* Удобно для dev/test/prod окружений.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: staging
```

---

### 1.9. Job / CronJob

* **Job** — выполнение задачи один раз (batch).
* **CronJob** — периодический запуск (по расписанию).

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cleanup
spec:
  schedule: "0 2 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: cleanup
              image: alpine
              command: ["sh","-c","echo cleanup"]
          restartPolicy: OnFailure
```

---

### 1.10. DaemonSet

* Запуск Pod’а на **каждом узле** (логирование, мониторинг, агент).

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: log-agent
spec:
  selector:
    matchLabels:
      app: log-agent
  template:
    metadata:
      labels:
        app: log-agent
    spec:
      containers:
        - name: agent
          image: fluentd:latest
```

---

## 2. Как описываются ресурсы

* Формат: **YAML манифест**.
* Общая структура:

  ```yaml
  apiVersion: <группа/версия API>
  kind: <тип ресурса>
  metadata:
    name: <имя>
    labels: {key: value}
  spec:
    ... описание ресурса ...
  ```
* Применение:

  ```bash
  kubectl apply -f deployment.yaml
  kubectl get pods -n staging
  ```

---

## 3. Выжимка для собеседования

* **Pod** — базовая единица, контейнеры внутри.
* **Deployment** — управление Pod’ами: реплики, rolling update.
* **Service** — сетевой доступ: ClusterIP, NodePort, LoadBalancer.
* **Ingress** — маршрутизация HTTP/HTTPS, TLS.
* **ConfigMap/Secret** — конфиги и секреты (env/files).
* **PVC/PV** — работа с постоянным хранилищем.
* **Namespace** — логическая изоляция ресурсов.
* **Job/CronJob** — фоновые задачи, планировщик.
* **DaemonSet** — Pod на каждом узле.
* Все ресурсы описываются YAML-манифестами (`apiVersion`, `kind`, `metadata`, `spec`).
