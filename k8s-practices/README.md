# Prácticas de Kubernetes con kind

Este repositorio contiene guías y manifiestos para realizar prácticas introductorias de Kubernetes usando [kind](https://kind.sigs.k8s.io/). Cada práctica está diseñada para ejecutarse en un entorno local con Docker.

## Objetivos de Aprendizaje

- **Comprender los conceptos fundamentales de Kubernetes**: Pods, Deployments, Services, ConfigMaps, Secrets, Volumes, etc.
- **Adquirir experiencia práctica**: Aprender a crear, gestionar y solucionar problemas en un clúster de Kubernetes.
- **Desarrollar habilidades en `kubectl`**: Dominar los comandos esenciales para interactuar con el clúster.
- **Entender el ciclo de vida de las aplicaciones en Kubernetes**: Desde el despliegue hasta el escalado y la actualización.

## Requisitos

- Docker y [kind](https://kind.sigs.k8s.io/) instalados.
- `kubectl` configurado para el clúster de kind.

## Estructura del Repositorio

| Archivo | Descripción |
|---|---|
| `manifests/base/nginx-pod.yaml` | Manifiesto para crear un Pod con Nginx. |
| `manifests/base/nginx-deployment.yaml` | Deployment para gestionar réplicas de Nginx. |
| `manifests/base/nginx-service.yaml` | Expone el Deployment con un `Service`. |
| `manifests/config/mensaje.txt` | Archivo para crear un ConfigMap. |
| `manifests/config/pod-con-configmap.yaml` | Pod que consume un ConfigMap. |
| `manifests/storage/volumenes.yaml` | Ejemplo de `PersistentVolume` y `PersistentVolumeClaim`. |
| `manifests/config/pod-secreto.yaml` | Pod que usa un Secret para variables de entorno. |
| `manifests/base/nginx-probe.yaml` | Pod con probes de salud y disponibilidad. |
| `manifests/networking/ingress.yaml` | Ingress para exponer el servicio de Nginx. |
| `manifests/storage/statefulset.yaml` | StatefulSet con almacenamiento persistente. |
| `manifests/jobs/job.yaml` | Job para tareas que terminan. |
| `manifests/jobs/cronjob.yaml` | CronJob para tareas programadas. |
| `manifests/networking/network-policy.yaml` | NetworkPolicy para controlar el tráfico. |
| `manifests/autoscaling/hpa.yaml` | HPA para escalar un Deployment. |
| `manifests/daemonset/daemonset.yaml` | DaemonSet para ejecutar un Pod en cada nodo. |
| `manifests/rbac/rbac.yaml` | Ejemplo de ServiceAccount y RBAC. |

## Práctica 1: Clúster con kind y Despliegue de un Pod

### Objetivo
Familiarizarse con la creación de un clúster de Kubernetes local y el despliegue de la unidad de trabajo más básica: el **Pod**.

### Pasos
1. **Crear el clúster con mapeo de puertos**:
   Para acceder a los servicios `NodePort` desde tu máquina, es necesario mapear los puertos del nodo de Kind (un contenedor Docker) a tu `localhost`. Para ello, `kind` utiliza un archivo de configuración.

   Asegúrate de tener el archivo `kind-config.yaml` en la raíz del proyecto con el siguiente contenido:
   ```yaml
   kind: Cluster
   apiVersion: kind.x-k8s.io/v1alpha4
   nodes:
   - role: control-plane
     extraPortMappings:
     - containerPort: 30080
       hostPort: 8080
       protocol: TCP
   ```

   Ahora, crea el clúster con este archivo:
   ```bash
   kind create cluster --name practicas-k8s --config kind-config.yaml
   ```
   **Explicación**: `--config` le indica a `kind` que use nuestro archivo. `extraPortMappings` mapea el puerto `8080` de tu máquina (`hostPort`) al puerto `30080` del contenedor que funciona como nodo del clúster (`containerPort`).

2. **Verificar los nodos**:
   ```bash
   kubectl get nodes
   ```
   **Explicación**: `kubectl` es la herramienta de línea de comandos para interactuar con Kubernetes. Con `get nodes`, se listan todos los nodos del clúster y se verifica que estén en estado `Ready`.

3. **Crear el Pod**:
   ```bash
   kubectl apply -f manifests/base/nginx-pod.yaml
   ```
   **Explicación**: `kubectl apply` crea o actualiza recursos en Kubernetes a partir de un archivo de manifiesto. En este caso, se crea un Pod con un contenedor de Nginx.

4. **Comprobar el Pod y sus logs**:
   ```bash
   kubectl get pods
   kubectl logs nginx
   ```
   **Explicación**: `get pods` muestra el estado de los Pods. `logs nginx` muestra los logs del contenedor `nginx`, útil para depurar.

5. **Desafío**: Edite `nginx-pod.yaml`, cambie la imagen a `httpd` y vuelva a aplicar el manifiesto. Observe cómo Kubernetes actualiza el Pod.

---

## Práctica 2: Deployment y Service

### Objetivo
Aprender a gestionar aplicaciones con **Deployments** y a exponerlas con **Services**. Un Deployment asegura que un número específico de réplicas de un Pod estén siempre en ejecución.

### Pasos
1. **Aplicar el Deployment**:
   ```bash
   kubectl apply -f manifests/base/nginx-deployment.yaml
   ```
   **Explicación**: Se crea un Deployment que gestionará Pods de Nginx. Si un Pod falla, el Deployment lo reemplazará automáticamente.

2. **Ver los Pods**:
   ```bash
   kubectl get pods -l app=nginx
   ```
   **Explicación**: El selector `-l app=nginx` filtra los Pods que tienen la etiqueta `app=nginx`, definida en el manifiesto del Deployment.

3. **Exponer con un Service**:
   ```bash
   kubectl apply -f manifests/base/nginx-service.yaml
   ```
   **Explicación**: Se crea un Service de tipo `NodePort`, que expone el Deployment en el puerto `30080` en el nodo del clúster.

4. **Probar el acceso**:
    ```bash
    curl http://localhost:8080
    ```
   **Explicación**: Gracias al mapeo de puertos en la configuración de Kind, una petición a `localhost:8080` en tu máquina es redirigida al `NodePort` (`30080`) en el clúster, llegando al servicio de Nginx.

5. **Desafío**: Escale el Deployment a 5 réplicas con `kubectl scale deployment nginx-deployment --replicas=5`. Verifique que se creen nuevos Pods.

---

## Práctica 3: ConfigMaps

### Objetivo
Aprender a externalizar la configuración de una aplicación utilizando **ConfigMaps**, lo que permite desacoplar la configuración del código.

### Pasos
1. **Crear el ConfigMap**:
   ```bash
   kubectl create configmap mensaje-config --from-file=manifests/config/mensaje.txt
   ```
   **Explicación**: Se crea un ConfigMap llamado `mensaje-config` a partir del contenido del archivo `mensaje.txt`.

2. **Crear el Pod**:
   ```bash
   kubectl apply -f manifests/config/pod-con-configmap.yaml
   ```
   **Explicación**: Se crea un Pod que monta el ConfigMap como un volumen. El contenido del ConfigMap estará disponible como un archivo dentro del contenedor.

3. **Verificar el contenido**:
   ```bash
   kubectl exec -it pod-con-configmap -- cat /config/mensaje.txt
   ```
   **Explicación**: `exec` permite ejecutar un comando dentro de un contenedor. Aquí, se muestra el contenido del archivo montado desde el ConfigMap.

4. **Desafío**: Modifique `mensaje.txt`, vuelva a crear el ConfigMap y reinicie el Pod para ver el contenido actualizado.

---

## Práctica 4: Volúmenes Persistentes

### Objetivo
Entender cómo proporcionar almacenamiento persistente a los Pods para que los datos sobrevivan a reinicios, utilizando **PersistentVolumes** y **PersistentVolumeClaims**.

### Pasos
1. **Crear los recursos**:
   ```bash
   kubectl apply -f manifests/storage/volumenes.yaml
   ```
   **Explicación**: Este manifiesto crea un `PersistentVolume` (PV), que es una pieza de almacenamiento en el clúster, y un `PersistentVolumeClaim` (PVC), que es una solicitud de almacenamiento por parte de un Pod.

2. **Verificar la creación del archivo**:
   ```bash
   kubectl exec -it pod-con-pv -- cat /data/hola.txt
   ```
   **Explicación**: El Pod escribe en el volumen montado. Se verifica que el archivo se haya creado correctamente.

3. **Comprobar la persistencia**: Elimine el Pod con `kubectl delete pod pod-con-pv` y vuelva a crearlo. El archivo `hola.txt` seguirá existiendo.

4. **Desafío**: Cree un segundo Pod y monte el mismo PVC para compartir datos entre ambos.

---

## Práctica 5: Secrets

### Objetivo
Aprender a gestionar información sensible, como contraseñas o claves de API, de forma segura utilizando **Secrets**.

### Pasos
1. **Crear el Secret**:
   ```bash
   kubectl create secret generic mi-secreto --from-literal=CLAVE_API=12345
   ```
   **Explicación**: Se crea un Secret llamado `mi-secreto` con un valor codificado en Base64.

2. **Desplegar el Pod**:
   ```bash
   kubectl apply -f manifests/config/pod-secreto.yaml
   ```
   **Explicación**: El manifiesto del Pod especifica que el valor del Secret `mi-secreto` se debe inyectar como una variable de entorno `CLAVE_API`.

3. **Comprobar la variable de entorno**:
   ```bash
   kubectl exec -it pod-secreto -- env | grep CLAVE_API
   ```
   **Explicación**: Se listan las variables de entorno del contenedor y se filtra por `CLAVE_API` para verificar que el Secret se ha inyectado correctamente.

4. **Desafío**: Combine un ConfigMap y un Secret para pasar múltiples variables de entorno (una de configuración y otra secreta) al mismo Pod.

---

## Práctica 6: Health Checks (Probes)

### Objetivo
Aprender a configurar **Liveness Probes** y **Readiness Probes** para que Kubernetes pueda gestionar automáticamente la salud de las aplicaciones.

### Pasos
1. **Crear el Pod con Probes**:
   ```bash
   kubectl apply -f manifests/base/nginx-probe.yaml
   ```
   **Explicación**:
   - **Liveness Probe**: Kubernetes la usa para saber si un contenedor está en ejecución. Si falla, Kubernetes reinicia el contenedor.
   - **Readiness Probe**: Kubernetes la usa para saber si un contenedor está listo para recibir tráfico. Si falla, Kubernetes no le enviará peticiones.

2. **Observar los eventos**:
   ```bash
   kubectl describe pod nginx-con-probes
   ```
   **Explicación**: `describe` proporciona información detallada sobre el Pod, incluyendo el estado de las probes y los eventos relacionados.

3. **Desafío**: Modifique la `livenessProbe` en el manifiesto para que apunte a una ruta inexistente (ej. `path: /error`). Aplique los cambios y observe cómo Kubernetes reinicia el Pod después de varios intentos fallidos.

---

## Práctica 7: Ingress

### Objetivo
Aprender a exponer servicios HTTP y HTTPS fuera del clúster utilizando un **Ingress**, que actúa como un enrutador inteligente para las peticiones entrantes.

### Pasos
1. **Asegurarse de tener el Service**: Verifique que el `nginx-service` de la Práctica 2 esté en ejecución.
2. **Aplicar el Ingress**:
   ```bash
   kubectl apply -f manifests/networking/ingress.yaml
   ```
   **Explicación**: Se crea un Ingress que enruta el tráfico del host `local.example.com` al `nginx-service`.

3. **Configurar `/etc/hosts`**:
   Añada la línea `127.0.0.1 local.example.com` a su archivo `/etc/hosts` para resolver el dominio localmente.
   **Explicación**: Esto simula una entrada DNS, haciendo que las peticiones a `local.example.com` se dirijan a su máquina local, donde el Ingress Controller de `kind` las interceptará.

4. **Acceder**: Abra `http://local.example.com` en su navegador.

5. **Desafío**: Agregue una nueva regla al Ingress para enrutar el tráfico de `http://local.example.com/api` a un servicio diferente.

---

## Práctica 8: StatefulSet

### Objetivo
Entender cómo desplegar aplicaciones con estado que requieren identidades de red estables y almacenamiento persistente único por réplica, utilizando **StatefulSets**.

### Pasos
1. **Crear el StatefulSet**:
   ```bash
   kubectl apply -f manifests/storage/statefulset.yaml
   ```
   **Explicación**: Se crea un StatefulSet que desplegará Pods con nombres predecibles (ej. `web-0`, `web-1`) y un PVC único para cada uno.

2. **Observar los PVCs**:
   ```bash
   kubectl get pvc
   ```
   **Explicación**: Verá que se ha creado un PVC por cada réplica del StatefulSet, vinculado a un PV.

3. **Verificar la persistencia**: Escriba un archivo en un Pod (ej. `web-0`), elimínelo y espere a que Kubernetes lo recree. El archivo persistirá.

4. **Desafío**: Escale el StatefulSet a 3 réplicas con `kubectl scale statefulset web --replicas=3` y observe cómo se crea un nuevo Pod con su propio PVC.

---

## Práctica 9: Jobs

### Objetivo
Aprender a ejecutar tareas que se ejecutan hasta completarse y luego terminan, utilizando **Jobs**. Ideal para procesos batch, migraciones de datos, etc.

### Pasos
1. **Crear el Job**:
   ```bash
   kubectl apply -f manifests/jobs/job.yaml
   ```
   **Explicación**: Se crea un Job que ejecuta un comando simple (`echo "Hola Mundo"` y `sleep 20`) y luego finaliza.

2. **Revisar los logs**:
   ```bash
   kubectl logs job/fecha-job
   ```
   **Explicación**: Una vez que el Job se completa, el Pod no se elimina para que pueda inspeccionar los logs.

3. **Desafío**: Modifique el comando en `job.yaml` para que falle (ej. `exit 1`). Observe cómo el Job reintenta la ejecución según el `backoffLimit`.

---

## Práctica 10: CronJobs

### Objetivo
Aprender a programar la ejecución de Jobs en momentos específicos o a intervalos regulares, utilizando **CronJobs**.

### Pasos
1. **Desplegar el CronJob**:
   ```bash
   kubectl apply -f manifests/jobs/cronjob.yaml
   ```
   **Explicación**: Se crea un CronJob que se ejecuta cada minuto (`schedule: "*/1 * * * *"`), creando un nuevo Job en cada ejecución.

2. **Listar los Jobs**:
   ```bash
   kubectl get jobs
   ```
   **Explicación**: Después de unos minutos, verá varios Jobs creados por el CronJob.

3. **Desafío**: Cambie la programación a cada 5 minutos (`schedule: "*/5 * * * *"`) y aplique los cambios.

---

## Práctica 11: Network Policies

### Objetivo
Aprender a controlar el flujo de tráfico entre Pods utilizando **NetworkPolicies**, implementando un modelo de seguridad de "confianza cero".

### Pasos
1. **Aplicar la NetworkPolicy**:
   ```bash
   kubectl apply -f manifests/networking/network-policy.yaml
   ```
   **Explicación**: Esta política solo permite el tráfico entrante a los Pods con la etiqueta `app=nginx` si el tráfico proviene de Pods con la etiqueta `access=granted`.

2. **Probar el acceso permitido**:
   Cree un Pod con la etiqueta `access=granted` e intente acceder al `nginx-service`. La conexión funcionará.
   ```bash
   kubectl run busybox --image=busybox --labels=access=granted --rm -it -- wget --spider nginx-service
   ```

3. **Desafío**: Intente acceder desde un Pod sin la etiqueta `access=granted`. La conexión será rechazada.
   ```bash
   kubectl run busybox --image=busybox --rm -it -- wget --spider nginx-service
   ```

---

## Práctica 12: Autoscaling con HPA

### Objetivo
Aprender a escalar automáticamente el número de Pods en un Deployment según la demanda, utilizando el **HorizontalPodAutoscaler (HPA)**.

### Pasos
1. **Aplicar el HPA**:
   ```bash
   kubectl apply -f manifests/autoscaling/hpa.yaml
   ```
   **Explicación**: Se crea un HPA que monitorea el uso de CPU del `nginx-deployment` y aumenta o disminuye el número de réplicas para mantener un promedio de 50% de utilización.

2. **Generar carga**:
   Para generar carga, puede ejecutar un bucle `while` en un Pod de prueba.
   ```bash
   kubectl run -it --rm load-generator --image=busybox /bin/sh
   while true; do wget -q -O- http://nginx-service; done
   ```
   Observe cómo el HPA crea nuevas réplicas con `kubectl get hpa`.

3. **Desafío**: Ajuste el `averageUtilization` a `80` y observe cómo el HPA reacciona de manera diferente a la misma carga.

---

## Práctica 13: DaemonSets

### Objetivo
Aprender a desplegar un Pod en cada nodo del clúster utilizando **DaemonSets**, útil para agentes de monitoreo, logging o almacenamiento.

### Pasos
1. **Crear el DaemonSet**:
   ```bash
   kubectl apply -f manifests/daemonset/daemonset.yaml
   ```
   **Explicación**: Se crea un DaemonSet que asegura que una copia del Pod `fluentd-elasticsearch` se ejecute en cada nodo del clúster.

2. **Comprobar los Pods**:
   ```bash
   kubectl get pods -o wide
   ```
   **Explicación**: Verá un Pod del DaemonSet por cada nodo, asignado a ese nodo específico.

3. **Desafío**: Agregue un nuevo nodo a su clúster de `kind` y observe cómo el DaemonSet despliega automáticamente un nuevo Pod en él.

---

## Práctica 14: ServiceAccounts y RBAC

### Objetivo
Entender cómo controlar el acceso a la API de Kubernetes utilizando **ServiceAccounts**, **Roles** y **RoleBindings** (RBAC - Role-Based Access Control).

### Pasos
1. **Desplegar los recursos RBAC**:
   ```bash
   kubectl apply -f manifests/rbac/rbac.yaml
   ```
   **Explicación**:
   - **ServiceAccount**: `lector-pods` es una identidad para procesos.
   - **Role**: `lector-pods-role` define permisos (en este caso, `get`, `watch`, `list` para Pods).
   - **RoleBinding**: `lector-pods-binding` asigna el Role a la ServiceAccount.

2. **Usar la ServiceAccount**:
   Intente listar Pods usando el token de la ServiceAccount.
   **Explicación**: Esto demuestra cómo un proceso (o un usuario) puede autenticarse con permisos limitados.

3. **Desafío**: Extienda el `Role` para permitir también la creación de Pods (`create`). Vuelva a probar los permisos.

---

## Limpieza

Para eliminar el clúster y todos los recursos creados:
```bash
kind delete cluster --name practicas-k8s
```