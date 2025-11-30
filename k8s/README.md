# 🚀 ArgoCD y Kubernetes para Lab3V

## 📋 Estructura del Proyecto

```
k8s/
├── Dockerfile                    # Dockerfile para construir la imagen
├── application.properties        # Configuración para desarrollo
├── application-kubernetes.properties # Configuración para Kubernetes
├── namespace.yaml               # Namespace lab3v
├── mysql-secret.yaml           # Secretos para MySQL
├── mysql-deployment.yaml       # Deployment y Service para MySQL
├── lab3v-deployment.yaml      # Deployment y Service para la aplicación
├── argocd-application.yaml    # Configuración de la aplicación ArgoCD
├── argocd-project.yaml        # Proyecto ArgoCD
└── README.md                   # Este archivo
```

## 🐳 Pasos para el Despliegue

### 1. Construir y Publicar la Imagen Docker

```bash
# Construir la imagen
docker build -t lab3v:latest .

# Para desarrollo local
docker tag lab3v:latest localhost:5000/lab3v:latest
docker push localhost:5000/lab3v:latest

# Para producción (cambia por tu registry)
docker tag lab3v:latest tu-registry/lab3v:latest
docker push tu-registry/lab3v:latest
```

### 2. Instalar ArgoCD

```bash
# Crear namespace
kubectl create namespace argocd

# Instalar ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 3. Acceder a ArgoCD

```bash
# Port forwarding
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Obtener contraseña
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# URL: https://localhost:8080
# Usuario: admin
# Contraseña: [la obtenida arriba]
```

### 4. Configurar Repositorio Git

**IMPORTANTE**: Cambia `tu-usuario` en los siguientes archivos:
- `argocd-application.yaml` - Linea `repoURL`
- `argocd-project.yaml` - Linea `sourceRepos`

### 5. Aplicar Configuración ArgoCD

```bash
# Aplicar proyecto y aplicación
kubectl apply -f argocd-project.yaml
kubectl apply -f argocd-application.yaml

# O aplicar todo de una vez
kubectl apply -f .
```

## 🔧 Configuración Específica

### Variables de Entorno

Las siguientes variables se configuran automáticamente:
- `SPRING_DATASOURCE_URL`: URL de MySQL en Kubernetes
- `SPRING_DATASOURCE_USERNAME`: Usuario desde secreto
- `SPRING_DATASOURCE_PASSWORD`: Contraseña desde secreto
- `SPRING_PROFILES_ACTIVE`: Profile de Kubernetes

### Health Checks

La aplicación incluye:
- **Liveness Probe**: `/actuator/health` (30s delay)
- **Readiness Probe**: `/actuator/health/readiness` (40s delay)

### Recursos

- **Memoria**: 512Mi request, 1Gi límite
- **CPU**: 250m request, 500m límite
- **Replicas**: 2 (para alta disponibilidad)

## 🌐 Acceso a la Aplicación

### Despliegue Local (Minikube)

```bash
# Exponer el servicio
minikube service lab3v-service --namespace=lab3v
```

### Producción

```bash
# Obtener el LoadBalancer IP
kubectl get svc lab3v-service -n lab3v
```

## 📊 Monitoring y Logs

```bash
# Ver logs de la aplicación
kubectl logs -f deployment/lab3v -n lab3v

# Ver logs de MySQL
kubectl logs -f deployment/mysql -n lab3v

# Ver estado del despliegue
kubectl get pods -n lab3v
kubectl get deployments -n lab3v
kubectl get services -n lab3v
```

## 🔍 Verificación

1. **ArgoCD UI**: https://localhost:8080
2. **Health Checks**: http://<service-ip>:8089/actuator/health
3. **Swagger UI**: http://<service-ip>:8089/swagger-ui.html
4. **API Endpoints**: http://<service-ip>:8089/flight

## 🚨 Troubleshooting

### Problemas Comunes

1. **ImagePullBackOff**:
   - Verifica que la imagen esté publicada
   - Asegúrate que el registry es accesible

2. **CrashLoopBackOff**:
   - Revisa los logs con `kubectl logs`
   - Verifica la conexión a MySQL

3. **Pod pendiente**:
   - `kubectl describe pod <pod-name> -n lab3v`
   - Verifica recursos disponibles

### Comandos Útiles

```bash
# Reiniciar despliegue
kubectl rollout restart deployment/lab3v -n lab3v

# Ver eventos
kubectl get events -n lab3v --sort-by='.lastTimestamp'

# Shell en pod
kubectl exec -it <pod-name> -n lab3v -- /bin/bash

# Eliminar todo
kubectl delete namespace lab3v
```