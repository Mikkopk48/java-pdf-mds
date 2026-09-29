## ☸️ Kubernetes

[⬆️ Volver al índice](./README.md)

---

## 📑 Contenidos de esta sección

### 🔹 Conceptos Fundamentales
1. [¿Qué es Kubernetes?](#1-qué-es-kubernetes-y-para-qué-sirve)
2. [Arquitectura de Kubernetes](#2-arquitectura-de-kubernetes)
3. [Pods](#3-qué-es-un-pod)
4. [Services](#4-qué-es-un-service)
5. [Deployments](#5-qué-es-un-deployment)

### 🔹 Recursos Clave
6. [ConfigMaps y Secrets](#6-configmaps-y-secrets)
7. [Volumes y PersistentVolumes](#7-volumes-y-persistentvolumes)
8. [Namespaces](#8-qué-son-los-namespaces)
9. [Ingress](#9-qué-es-un-ingress)

### 🔹 Escalabilidad y Alta Disponibilidad
10. [ReplicaSets](#10-qué-es-un-replicaset)
11. [HorizontalPodAutoscaler](#11-horizontalpodautoscaler-hpa)
12. [Liveness y Readiness Probes](#12-liveness-y-readiness-probes)

### 🔹 Comandos Esenciales
13. [kubectl comandos básicos](#13-comandos-kubectl-esenciales)

---

### 🔹 Conceptos Fundamentales

#### 1. ¿Qué es Kubernetes y para qué sirve?

**Respuesta:**

**Kubernetes (K8s)** es una plataforma de **orquestación de contenedores** que automatiza el despliegue, escalado y gestión de aplicaciones containerizadas.

**Problemas que resuelve:**

```
Sin Kubernetes:
- ¿Dónde ejecutar contenedores?
- ¿Cómo escalar cuando hay más tráfico?
- ¿Cómo reemplazar contenedores que fallan?
- ¿Cómo hacer rolling updates sin downtime?
- ¿Cómo balancear carga entre instancias?

Con Kubernetes:
✅ Deployment automático
✅ Auto-scaling
✅ Self-healing
✅ Rolling updates & rollbacks
✅ Load balancing
✅ Service discovery
✅ Secret management
```

**Beneficios:**

1. **Orquestación**: Gestiona múltiples contenedores
2. **Escalabilidad**: Escala horizontal automáticamente
3. **Alta disponibilidad**: Reinicia contenedores fallidos
4. **Portabilidad**: Ejecuta en cualquier cloud (AWS, Azure, GCP) o on-premise
5. **Declarativo**: Defines el estado deseado, K8s lo mantiene

**Casos de uso:**
- Microservicios en producción
- Aplicaciones con alta demanda variable
- Ambientes multi-cloud/hybrid
- CI/CD pipelines

---

#### 2. Arquitectura de Kubernetes

**Respuesta:**

Kubernetes tiene una arquitectura **maestro-trabajador**:

```
┌─────────────────────────────────────────┐
│           Control Plane                 │
│  ┌──────────┐  ┌──────────────────┐   │
│  │ API      │  │ etcd (key-value) │   │
│  │ Server   │  │ storage          │   │
│  └──────────┘  └──────────────────┘   │
│  ┌──────────┐  ┌──────────────────┐   │
│  │Scheduler │  │ Controller       │   │
│  │          │  │ Manager          │   │
│  └──────────┘  └──────────────────┘   │
└─────────────────────────────────────────┘
           │
           │ (gestiona)
           ▼
┌────────────────────────────────────────┐
│          Worker Nodes                  │
│  ┌──────────────────────────────────┐ │
│  │ Node 1                           │ │
│  │  ┌────────┐  ┌────────┐         │ │
│  │  │ kubelet│  │kube-   │         │ │
│  │  │        │  │proxy   │         │ │
│  │  └────────┘  └────────┘         │ │
│  │  ┌────────┐  ┌────────┐         │ │
│  │  │ Pod 1  │  │ Pod 2  │         │ │
│  │  └────────┘  └────────┘         │ │
│  └──────────────────────────────────┘ │
│  ┌──────────────────────────────────┐ │
│  │ Node 2                           │ │
│  │  ...                             │ │
│  └──────────────────────────────────┘ │
└────────────────────────────────────────┘
```

**Control Plane (Master):**

- **API Server**: Punto de entrada, expone API REST
- **etcd**: Base de datos clave-valor (almacena estado del cluster)
- **Scheduler**: Decide en qué nodo ejecutar pods
- **Controller Manager**: Mantiene estado deseado (replicas, health checks)

**Worker Nodes:**

- **kubelet**: Agente que ejecuta en cada nodo, gestiona pods
- **kube-proxy**: Gestiona networking y load balancing
- **Container Runtime**: Docker, containerd, CRI-O

**Flujo de despliegue:**
```
1. kubectl aplica deployment.yaml
2. API Server recibe y guarda en etcd
3. Controller Manager detecta nuevo deployment
4. Scheduler asigna pods a nodos
5. kubelet en los nodos ejecuta contenedores
6. kube-proxy configura red y load balancing
```

---

#### 3. ¿Qué es un Pod?

**Respuesta:**

Un **Pod** es la unidad más pequeña en Kubernetes. Representa uno o más contenedores que comparten red y almacenamiento.

**Características:**
- Cada pod tiene una IP única en el cluster
- Contenedores en el mismo pod comparten localhost
- Es efímero (si muere, se crea uno nuevo con otra IP)

**Ejemplo básico:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mi-app
  labels:
    app: backend
spec:
  containers:
  - name: app-container
    image: mi-app:1.0
    ports:
    - containerPort: 8080
    env:
    - name: ENVIRONMENT
      value: "production"
    resources:
      requests:
        memory: "256Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "500m"
```

**Multi-container pod (sidecar pattern):**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-logging
spec:
  containers:
  # Contenedor principal
  - name: app
    image: mi-app:1.0
    ports:
    - containerPort: 8080
    volumeMounts:
    - name: logs
      mountPath: /var/log/app

  # Sidecar: envía logs a servicio externo
  - name: log-shipper
    image: fluent/fluentd:v1.16
    volumeMounts:
    - name: logs
      mountPath: /var/log/app

  volumes:
  - name: logs
    emptyDir: {}
```

**¿Cuándo usar un pod vs varios?**

**Un contenedor por pod** (común):
- Microservicios independientes
- Aplicaciones simples

**Múltiples contenedores por pod**:
- Sidecar logging/monitoring
- Proxy (Envoy, Nginx)
- Adaptadores de formato

---

#### 4. ¿Qué es un Service?

**Respuesta:**

Un **Service** proporciona una **IP estable** y **DNS name** para acceder a un conjunto de pods, con load balancing automático.

**Problema:** Los pods son efímeros (IPs cambian al recrearse)
**Solución:** Service proporciona endpoint estable

**Tipos de Services:**

**1. ClusterIP (default):** Solo accesible dentro del cluster
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP
  selector:
    app: backend  # Selecciona pods con esta label
  ports:
  - port: 80          # Puerto del service
    targetPort: 8080  # Puerto del pod
```

**2. NodePort:** Expone servicio en puerto del nodo
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: NodePort
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080  # Puerto accesible externamente (30000-32767)
```

**3. LoadBalancer:** Crea un load balancer externo (cloud provider)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: LoadBalancer
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 8080
```

**Service Discovery:**
```yaml
# Frontend puede acceder a backend usando DNS
# http://backend-service:80
# http://backend-service.default.svc.cluster.local:80

apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  containers:
  - name: frontend
    image: frontend:1.0
    env:
    - name: BACKEND_URL
      value: "http://backend-service:80"
```

---

#### 5. ¿Qué es un Deployment?

**Respuesta:**

Un **Deployment** gestiona el ciclo de vida de pods con capacidad de rolling updates, rollbacks y escalado.

**Ejemplo completo:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
  labels:
    app: backend
spec:
  replicas: 3  # 3 pods idénticos
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: mi-backend:1.0
        ports:
        - containerPort: 8080
        env:
        - name: DB_HOST
          value: postgres-service
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
```

**Rolling Update:**
```bash
# Actualizar imagen
kubectl set image deployment/backend-deployment backend=mi-backend:2.0

# Ver progreso
kubectl rollout status deployment/backend-deployment

# Historial de versiones
kubectl rollout history deployment/backend-deployment

# Rollback a versión anterior
kubectl rollout undo deployment/backend-deployment

# Rollback a versión específica
kubectl rollout undo deployment/backend-deployment --to-revision=2
```

**Estrategias de despliegue:**
```yaml
spec:
  strategy:
    type: RollingUpdate  # Default
    rollingUpdate:
      maxSurge: 1        # Máximo 1 pod extra durante update
      maxUnavailable: 0  # 0 pods pueden estar down (zero downtime)
```

---

#### 6. ConfigMaps y Secrets

**Respuesta:**

**ConfigMap:** Almacena configuración no sensible
**Secret:** Almacena datos sensibles (passwords, tokens) en base64

**ConfigMap:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  application.properties: |
    server.port=8080
    spring.application.name=mi-app
  DATABASE_URL: "jdbc:postgresql://postgres:5432/mydb"
  LOG_LEVEL: "INFO"
```

**Secret:**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  # Valores en base64
  DB_PASSWORD: c3VwZXJzZWNyZXQ=  # echo -n 'supersecret' | base64
  API_KEY: bXlhcGlrZXkxMjM=
```

**Usar en Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  template:
    spec:
      containers:
      - name: backend
        image: mi-app:1.0
        
        # ConfigMap como variables de entorno
        envFrom:
        - configMapRef:
            name: app-config
        
        # Secret como variables de entorno
        env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: DB_PASSWORD
        
        # ConfigMap como archivo
        volumeMounts:
        - name: config-volume
          mountPath: /app/config
      
      volumes:
      - name: config-volume
        configMap:
          name: app-config
```

**Crear desde línea de comandos:**
```bash
# ConfigMap desde archivo
kubectl create configmap app-config --from-file=application.properties

# Secret desde literal
kubectl create secret generic app-secrets \
  --from-literal=DB_PASSWORD=supersecret \
  --from-literal=API_KEY=mykey123
```

---

#### 11. HorizontalPodAutoscaler (HPA)

**Respuesta:**

**HPA** escala automáticamente el número de pods basándose en métricas (CPU, memoria, custom).

**Ejemplo básico:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # Escala si CPU > 70%
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80  # Escala si memoria > 80%
```

**Con métricas custom:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend-deployment
  minReplicas: 2
  maxReplicas: 20
  metrics:
  # CPU
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  
  # Requests por segundo (custom metric)
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "1000"
```

**Requisitos:**
- Metrics Server instalado en cluster
- Resources (requests/limits) definidos en pods

**Comandos:**
```bash
# Ver HPA
kubectl get hpa

# Ver métricas de pods
kubectl top pods

# Ver métricas de nodos
kubectl top nodes
```

---

#### 13. Comandos kubectl esenciales

**Respuesta:**

**Gestión de recursos:**
```bash
# Aplicar configuración
kubectl apply -f deployment.yaml
kubectl apply -f directorio/

# Crear desde línea de comandos
kubectl create deployment nginx --image=nginx:latest

# Eliminar recursos
kubectl delete deployment mi-app
kubectl delete -f deployment.yaml

# Ver recursos
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get all  # Todo en namespace actual

# Describir recurso (detalles + eventos)
kubectl describe pod mi-pod
kubectl describe deployment mi-deployment

# Logs
kubectl logs mi-pod
kubectl logs -f mi-pod  # Follow
kubectl logs mi-pod -c contenedor  # Específico en multi-container
kubectl logs --tail=100 mi-pod  # Últimas 100 líneas

# Ejecutar comando
kubectl exec -it mi-pod -- bash
kubectl exec mi-pod -- ls /app

# Port forwarding
kubectl port-forward pod/mi-pod 8080:80
kubectl port-forward service/mi-service 8080:80
```

**Escalado:**
```bash
# Escalar deployment
kubectl scale deployment mi-app --replicas=5

# Autoscale
kubectl autoscale deployment mi-app --min=2 --max=10 --cpu-percent=80
```

**Actualizaciones:**
```bash
# Actualizar imagen
kubectl set image deployment/mi-app contenedor=nueva-imagen:v2

# Ver estado del rollout
kubectl rollout status deployment/mi-app

# Pausar rollout
kubectl rollout pause deployment/mi-app

# Reanudar
kubectl rollout resume deployment/mi-app

# Rollback
kubectl rollout undo deployment/mi-app

# Historial
kubectl rollout history deployment/mi-app
```

**Debugging:**
```bash
# Ver eventos
kubectl get events --sort-by=.metadata.creationTimestamp

# Ver uso de recursos
kubectl top pods
kubectl top nodes

# Ver configuración como YAML
kubectl get pod mi-pod -o yaml
kubectl get deployment mi-deployment -o json

# Edit en vivo (no recomendado)
kubectl edit deployment mi-app
```

**Namespaces:**
```bash
# Listar namespaces
kubectl get namespaces

# Usar namespace específico
kubectl get pods -n mi-namespace

# Cambiar namespace por defecto
kubectl config set-context --current --namespace=mi-namespace
```

[⬆️ Volver al índice de esta sección](#-contenidos-de-esta-sección)

---

[⬅️ Anterior: Docker](./07-docker.md) | [🏠 Volver al Inicio](./README.md) | [Siguiente: AWS ➡️](./09-aws.md)
