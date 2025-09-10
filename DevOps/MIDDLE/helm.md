# Что такое helm, для чего он используется и как написать helm chart

## 1. Что такое Helm

* **Helm** — пакетный менеджер для Kubernetes.
* Пакет = **Chart**: набор манифестов k8s + шаблоны + значения.
* Решает: повторное использование, параметризацию, версионирование, релизы/обновления/откаты.

---

## 2. Где Helm помогает

* Деплой одного и того же сервиса в dev/stage/prod с разными параметрами.
* Управление зависимостями (БД, кэши) как под-чартами.
* Единый источник правды: настроенные `values` под окружения.
* Быстрый роллбек релиза.

---

## 3. Структура Helm-чарта

```
myapp/
  Chart.yaml           # метаданные чартa (имя, версия)
  values.yaml          # значения по умолчанию
  values-prod.yaml     # override для продакшена (опционально)
  templates/
    deployment.yaml    # шаблон Deployment
    service.yaml       # шаблон Service
    ingress.yaml       # шаблон Ingress (опц.)
    _helpers.tpl       # функции/шаблоны (имена, лейблы)
    NOTES.txt          # подсказка после install/upgrade
```

**Chart.yaml (Helm 3, SemVer)**

```yaml
apiVersion: v2
name: myapp
description: My Spring Boot app
type: application
version: 1.2.0        # версия чарта (SemVer)
appVersion: "1.0.7"   # версия приложения (для образов, меток)
dependencies:         # опционально (subcharts)
  - name: redis
    version: 18.x.x
    repository: https://charts.bitnami.com/bitnami
    condition: redis.enabled
```

---

## 4. values.yaml — параметры

```yaml
replicaCount: 2

image:
  repository: registry.example.com/myapp
  tag: "1.0.7"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80
  targetPort: 8080

ingress:
  enabled: true
  className: nginx
  host: myapp.example.com
  tlsSecret: myapp-tls

resources:
  requests: { cpu: "200m", memory: "256Mi" }
  limits:   { cpu: "500m", memory: "512Mi" }

env:
  SPRING_PROFILES_ACTIVE: "prod"
  SPRING_DATASOURCE_URL: "jdbc:postgresql://db:5432/mydb"
```

---

## 5. Шаблоны (templates/\*) — Go-templating

### 5.1. `_helpers.tpl` — функции/«сниппеты»

```gotemplate
{{- define "myapp.fullname" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" -}}
{{- end -}}

{{- define "myapp.labels" -}}
app.kubernetes.io/name: {{ include "myapp.fullname" . }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end -}}
```

### 5.2. Deployment (пример)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
  labels:
    {{- include "myapp.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ include "myapp.fullname" . }}
  template:
    metadata:
      labels:
        app: {{ include "myapp.fullname" . }}
    spec:
      containers:
        - name: app
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: {{ .Values.service.targetPort }}
          env:
            {{- range $k, $v := .Values.env }}
            - name: {{ $k }}
              value: {{ $v | quote }}
            {{- end }}
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          readinessProbe:
            httpGet: { path: /actuator/health, port: {{ .Values.service.targetPort }} }
            initialDelaySeconds: 10
            periodSeconds: 5
```

### 5.3. Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "myapp.fullname" . }}
spec:
  type: {{ .Values.service.type }}
  selector:
    app: {{ include "myapp.fullname" . }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.targetPort }}
```

### 5.4. Ingress (условно)

```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "myapp.fullname" . }}
spec:
  ingressClassName: {{ .Values.ingress.className }}
  tls:
    - hosts: [ {{ .Values.ingress.host }} ]
      secretName: {{ .Values.ingress.tlsSecret }}
  rules:
    - host: {{ .Values.ingress.host }}
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: {{ include "myapp.fullname" . }}
                port: { number: {{ .Values.service.port }} }
{{- end }}
```

---

## 6. Команды Helm (жизненный цикл)

```bash
helm create myapp                 # сгенерировать заготовку чарта
helm template myapp ./myapp -f values.yaml         # срендерить манифесты локально
helm lint ./myapp                 # статическая проверка чарта
helm install rel1 ./myapp -n staging --create-namespace
helm upgrade rel1 ./myapp -n staging -f values-staging.yaml
helm rollback rel1 1 -n staging   # откат к ревизии 1
helm uninstall rel1 -n staging
```

**Overrides значений:**

```bash
helm upgrade rel1 ./myapp -n prod -f values-prod.yaml \
  --set image.tag=1.0.8 --set replicaCount=3
```

---

## 7. Зависимости (subcharts)

* Описать в `Chart.yaml -> dependencies`.
* `helm dependency update` — подтянуть чарты в `charts/`.
* Условное подключение: `condition: redis.enabled` + `values.yaml: redis.enabled: false`.

---

## 8. Практики и приёмы

* **Helm 3**: без Tiller (все действия от имени kubectl-подобного клиента).
* Старайтесь **весь k8s-язык держать в шаблонах**, бизнес-параметры — в `values`.
* Используйте `include`, `nindent`, `toYaml`, `required`, `default`, `tpl`.
* Разделяйте values на окружения (`values-dev/stage/prod.yaml`).
* **Версионирование**: `version` (chart) ≠ `appVersion` (приложение).
* **Секреты**: не храните в plain-тексте — используйте Sealed Secrets/External Secrets/CSI Secret Store.
* **Проверки**: `helm lint`, `helm template` в CI + e2e в «песочнице».
* **Тесты Helm**: добавляйте манифесты с аннотацией `helm.sh/hook: test`.
* Пакуйте/публикуйте: `helm package ./myapp` → `helm push` в OCI-реестр (`helm registry login`).

---

## 9. Мини-пример: тест-хук и NOTES

**templates/tests/smoke.yaml**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "myapp.fullname" . }}-smoke"
  annotations:
    "helm.sh/hook": test
spec:
  restartPolicy: Never
  containers:
    - name: curl
      image: curlimages/curl:8
      args: ["-fsS", "http://{{ include "myapp.fullname" . }}:{{ .Values.service.port }}/actuator/health"]
```

**templates/NOTES.txt**

```
{{- if .Values.ingress.enabled -}}
App URL: https://{{ .Values.ingress.host }}
{{- else -}}
App Service: {{ include "myapp.fullname" . }}:{{ .Values.service.port }}
{{- end -}}
```

---

## 10. Частые ошибки

* Смешение логики в шаблоне и «значений» (сложно сопровождать).
* Несогласованность `Chart.version` и `appVersion`.
* Отсутствие `helm lint`/`helm template` в CI → поломки на кластере.
* Сырые Secrets в values — утечки в Git/CI-логах.
* Жёстко пришитые имена/лейблы → конфликты при нескольких релизах.

---

## 11. Выжимка для собеседования

* **Helm** — пакетный менеджер k8s; чарт = шаблоны манифестов + `values`.
* **Helm 3** без Tiller; основные команды: `install/upgrade/rollback/uninstall`, `lint`, `template`.
* **Chart.yaml** (v2): `name`, `version` (SemVer), `appVersion`, `dependencies`.
* **Шаблоны**: Go-templating; ключевые функции `include`, `nindent`, `toYaml`, `required`, `default`, `tpl`; контексты `.Values`, `.Release`, `.Chart`, `.Capabilities`.
* **Зависимости**: subcharts + `condition` в values.
* **Практики**: чёткое разделение values по окружениям, секреты через внешние механизмы, CI с lint/template, тест-хуки, NOTES.
* **Релизы**: параметризуемая установка, безопасные обновления, быстрый rollback.
