# 🐳 FastAPI K8s App – Sistema Distribuido con Minikube
<img width="1024" height="1024" alt="image" src="https://github.com/user-attachments/assets/770b4598-52fa-414d-bce6-c5e0d7984eee" />


Este proyecto implementa una arquitectura de microservicios distribuida utilizando **FastAPI, Redis, PostgreSQL y Nginx**, desplegada sobre un clúster de **Kubernetes en Minikube**.

---

## 📦 Componentes del sistema

| Componente           | Función |
|----------------------|----------|
| **FastAPI + Uvicorn** | API stateless que expone los endpoints `/` y `/db`. |
| **Redis**            | Sistema de almacenamiento en caché y contador de visitas. |
| **PostgreSQL**       | Base de datos relacional para la persistencia de datos. |
| **Nginx**            | Balanceador de carga para distribuir tráfico entre múltiples réplicas. |

---

## 📁 Estructura del proyecto

``` 
fastapi_k8s_app/
├── app/
│   └── main.py                # Código de la API FastAPI
├── k8s/
│   ├── app.yaml               # Despliegue y servicio para FastAPI
│   ├── redis.yaml             # Redis deployment + service
│   ├── postgres.yaml          # PostgreSQL deployment + PVC + service
│   └── nginx.yaml             # Configuración balanceador Nginx
├── Dockerfile                 # Imagen personalizada para FastAPI
├── build_and_reload.sh        # Script de despliegue sin Docker Desktop
└── README.md                  # Este archivo
```
# 🚀 Cómo desplegar

## 📋 Requisitos

- Docker Desktop (opcional)
- Minikube
- kubectl

Verifica instalación:

```bash
minikube version
kubectl version --client
```
## PASO 1 — Instalar kubectl
Ejecuta esto:
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```
Luego:
```
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```
Verifica:
```
kubectl version --client
```
Si muestra versión → ✅ listo.

## 🚀 PASO 2 — Instalar Minikube

Ejecuta:
```
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```
Luego:
```
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```
Verifica:
```
minikube version
```
Debe mostrar todas la verciones → ✅ listo.
```
docker --version
minikube version
kubectl version --client
```
## Iniciar el cluster
```
minikube start --driver=docker
```
Luego verificas que esté funcionando:
```
kubectl get nodes
```
Eso levanta Kubernetes usando Docker.

## 1.Inicia Minikube
```
minikube start
```
## 2. Construye la imagen dentro de Minikube
```
minikube image build -t fastapi-app:latest .
```
💡 Si usas Docker Desktop y no estás en entorno multinodo, puedes usar:
```
docker build -t fastapi-app:latest .
minikube image load fastapi-app:latest

```
## 3. Despliega todos los recursos de Kubernetes
```
kubectl apply -f k8s/
```
Esto crea:
- Deployments
- Services
- Configuración de Nginx

## 4. Verifica el estado de los Pods
```
kubectl get pods
```
## 5.  Obtén la URL pública para acceder a la app
```
minikube service nginx --url
```
Accede desde el navegador usando la URL generada.
## 📬 Endpoints disponibles

| <small>Método</small> | <small>Endpoint</small> | <small>Descripción</small> |
|---------------------------|------------------------|----------------------------|
| GET                       | /                      | Retorna mensaje y contador de visitas almacenado en Redis.    |
| GET                       | /db                    | Ejecuta las tareas         |

## 🔄 Escalabilidad y tolerancia a fallos
**Escalar horizontalmente**
```
kubectl scale deployment fastapi-app --replicas=5
```
Esto crea múltiples instancias de la API.

## 💥 Simular caída de una réplica
```
kubectl delete pod <nombre-del-pod>
```
Kubernetes recreará automáticamente el pod eliminado.
Nginx continuará balanceando entre las réplicas disponibles.

## 🛡️ Pruebas de Resiliencia

Estas pruebas permiten validar la tolerancia a fallos y el comportamiento del sistema ante interrupciones.

## 🔧 Opción A – Simular caída de NGINX (Pod)
```
kubectl delete pod -l app=nginx
```
Esto simula una falla inesperada. Kubernetes automáticamente levantará un nuevo pod gracias al Deployment.
Monitorear recreación:
```
kubectl get pods -l app=nginx -w
```
✅ Recomendado para probar auto-recuperación sin perder el recurso de servicio.

## 🔧 Opción B – Escalar NGINX a 0 (simular mantenimiento)
```
kubectl scale deployment nginx --replicas=0
```
Restaurar servicio:
```
kubectl scale deployment nginx --replicas=1
```
🔁 Útil para mantenimiento controlado.

## ❌ Opción NO recomendada – Eliminar el servicio de NGINX
```
kubectl delete svc nginx
```
⚠️ Esto elimina el balanceador de carga y la URL pública de Minikube dejará de funcionar. Solo usar si deseas reconfigurar el servicio desde cero.

## 🧪 Recomendaciones de prueba

- Ejecutar pruebas con múltiples réplicas activas.
- Verificar que la API responde tras recuperación.
- Usar **curl** o navegador para observar interrupciones mínimas.
- Monitorear pods en tiempo real:
```
kubectl get pods -w
```
## 🧼 Limpieza

Eliminar todos los recursos creados:
```
kubectl delete -f k8s/
```
Opcionalmente detener Minikube:
```
minikube stop
```
# 👨‍💻 Autor
**Omar Gomez** - DevOps & Cloud Enthusiast
























  
















