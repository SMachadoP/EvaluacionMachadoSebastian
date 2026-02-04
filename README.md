Sistema de Monitoreo en Tiempo Real - Kubernetes
Archivos yaml:
- secret.yaml: Define la contraseña de Redis para autenticación segura.
- pv.yaml: PersistentVolume que reserva 1GB de almacenamiento en el nodo de Kubernetes.
- pvc.yaml: PersistentVolumeClaim que solicita espacio del PV para Redis.
- redis-deployment.yaml
    - Despliega Redis con autenticación obligatoria
    - Monta el volumen persistente en /data
    - Configura la contraseña desde el Secret
- redis-service.yaml: Service interno ClusterIP para que otros pods se conecten a Redis.
- productor-deployment.yaml
    - Usa Python 3 Alpine (ligero)
    - Instala el cliente redis-py
    - Script inline que:
      - Se conecta a Redis con autenticación
        - Genera datos aleatorios cada 3 segundos
        - Formato JSON: `{"sensor_id": "rbt-01", "valor": X, "timestamp": "..."}`
        - Guarda en lista Redis llamada "sensor_data"
- cliente-deployment.yaml
    - Despliega Redis Commander
    - Se conecta automáticamente a Redis con la contraseña
- cliente-service.yaml: Service tipo NodePort en puerto 30080 para acceso desde el navegador.

Comandos:
Aplicar todas las configuraciones
kubectl apply -f k8s/secret.yaml
kubectl apply -f k8s/pv.yaml
kubectl apply -f k8s/pvc.yaml
kubectl apply -f k8s/redis-deployment.yaml
kubectl apply -f k8s/redis-service.yaml
kubectl apply -f k8s/productor-deployment.yaml
kubectl apply -f k8s/cliente-deployment.yaml
kubectl apply -f k8s/cliente-service.yaml

Acceder a la interfaz web
minikube service cliente-service

Verificar que el PersistentVolumeClaim está Bound
kubectl get pvc redis-pvc

Verificar que el productor está generando datos
kubectl logs deployment/productor --tail=10

