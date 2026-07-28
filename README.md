# Aplicación de ejemplo — Evaluación práctica (Docker, Kubernetes y CI/CD)

Aplicación mínima y neutral, provista únicamente para esta evaluación. No pertenece a ningún proyecto previo del curso: todos los estudiantes parten exactamente del mismo punto.

## Qué hace

Un servidor HTTP simple, sin dependencias externas, que expone:

- `GET /` — responde `200` con un mensaje, el nombre y la versión de la aplicación, en JSON.
- `GET /health` — responde igual que `/`, útil como endpoint de verificación de salud.
- Cualquier otra ruta — responde `404`.

Escucha por defecto en el puerto **8080** (configurable con la variable de entorno `PORT`).

## Cómo ejecutarla localmente

```
node server.js
curl http://localhost:8080/
```

## Cómo correr las pruebas

```
npm test
```

Usa el ejecutor de pruebas integrado de Node.js (`node --test`), por lo que no requiere instalar dependencias adicionales.

## Uso en la evaluación

Esta aplicación es el punto de partida para los tres retos de la evaluación práctica. Los archivos de Docker, Kubernetes y del pipeline de CI/CD **no se incluyen aquí a propósito**: se describen en el enunciado de la evaluación, con el defecto correspondiente, para que usted los recree y los corrija.

## Evidencia inicial obligatoria
•	Captura o registro de la instalación de dependencias.
```powershell
npm install
```
![alt text](/images/npm-install.png)
•	Captura o registro de la ejecución exitosa de las pruebas iniciales.
```powershell
npm test
```
![alt text](/images/npm-test.png)
•	Captura o registro de la aplicación respondiendo localmente.
```powershell
npm start
```
![alt text](/images/8080-local.png)
## Evidencias Reto 1
•	Captura o registro de la construcción de la imagen.
```powershell
docker build -t nombre-imagen:etiqueta .
```
![alt text](/images/Creacion-imagen.png)
•	Captura o registro del contenedor en ejecución.
```powershell
docker run -d --name nombre-contenedor -p puerto-anfitrion:puerto-contenedor nombre-imagen:etiqueta
docker run -d --name app-reto1-inicial -p 3000:3000 repaso-sd:reto1-inicial
```
![alt text](/images/docker-ejecutandose.png)
•	Captura o registro del intento fallido de acceso inicial.
![alt text](/images/Error.png)
•	Captura o registro que permita identificar el problema.
![alt text](/images/log-Error.png)
•	Captura o registro del archivo corregido.
![alt text](/images/Dockerfile-corre.png)
•	Captura o registro de la aplicación respondiendo correctamente desde la máquina anfitriona.
```powershell
docker build -t nombre-imagen:etiqueta .
# Reconstruir Imagen
docker build -t repaso-sd:reto1-corregido .
# Detener contenedor falloso
docker stop app-reto1-inicial
# Ejecutar Contenedor Corregido
docker run -d --name app-reto1-corregido -p 8080:8080 repaso-sd:reto1-corregido
```
![alt text](/images/contenedor-8080.png)

## Evidencias Reto 2
•	Captura o registro de la aplicación del manifiesto inicial.
```powershell
# Iniciar Minikube
minikube start --driver=docker
# Aplica los archivos de k8s
kubectl apply -f .\k8s\
```
Pequeño cambio en el deployment
Se utiliza esta línea
image: repaso-sd:reto1-corregido
![alt text](/images/manifiesto-inicial.png)
•	Captura o registro de los pods en estado Running.
Toca cambiar
```yaml
    matchLabels:
      app: webapp
```
POR 
```yaml
    matchLabels:
      app: web
```
```powershell
# Aplica solo Deployment
kubectl apply -f .\k8s\deployment.yaml
# Cargar la imagen local en minikube
minikube image load repaso-sd:reto1-corregido
# Observa la recreación
kubectl get pods -l app=web -w
```
![alt text](/images/pods-running.png)

•	Captura o registro del Service sin endpoints o sin destinos válidos.
```powershell
# Consultar Service
kubectl get service web-service
# Verificar Endpoints
kubectl get endpoints web-service
# Descripcion 
kubectl describe service web-service
```
![alt text](/images/Service-endpoints.png)

•	Captura o registro del manifiesto corregido.
Toca cambiar
```yaml
spec:
  selector:
    app: webapp
```
POR 
```yaml
spec:
  selector:
    app: web
```
![alt text](/images/Service-corregido.png)
```powershell
# Aplicar service
kubectl apply -f .\k8s\service.yaml
```
•	Captura o registro del Service con endpoints poblados.
```powershell
# Verificar Endpoints
kubectl get endpoints web-service
```
![alt text](/images/Endpoints-poblados.png)

•	Captura o registro de una petición exitosa hacia la aplicación usando el servicio de Kubernetes.

```powershell
# Abrir Service en un puerto local
kubectl port-forward service/web-service 8081:80
```
```powershell
# Llamarlo
curl.exe -i http://localhost:8081
```
![alt text](/images/Peticion-exitosa.png)

## Evidencias Reto 3 - CI/CD
•	Captura o registro del pipeline inicial.
![alt text](/images/registro-ci-cd.png)
![alt text](/images/Actions1-pipeline.png)

•	Captura o registro de una prueba fallida provocada intencionalmente.
PROVOCAR FALLA CAMBIANDO:
```js
 assert.strictEqual(response.status, 200);
  assert.ok(parsed.message, 'la respuesta debe incluir un mensaje');
  assert.ok(parsed.version, 'la respuesta debe incluir una version');
```
POR
```js
 assert.strictEqual(response.status, 500);
```
```powershell
npm test
```
![alt text](/images/Prueba-fallo.png)
•	Captura o registro que demuestre el comportamiento defectuoso del pipeline inicial.
![alt text](/images/Comportamiento-pipe.png)
•	Captura o registro del archivo de workflow corregido.
Se agrega esta Linea despues de:
```yaml
deploy:
    needs: build-test
```
![alt text](/images/ci-cd_Corregido.png)
•	Captura o registro de una ejecución con pruebas fallidas donde el despliegue no se ejecute.
Mantener el 500 en server.test.js
![alt text](/images/deploy-fallido.png)

•	Captura o registro de una ejecución final exitosa con pruebas aprobadas y despliegue ejecutado.
```yml
name: ci-cd

on:
  push:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: alanissette16/practica

jobs:
  build-test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - run: npm ci

      - run: npm test

  deploy:
    needs: build-test
    runs-on: ubuntu-latest

    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v4

      - name: Iniciar sesión en GHCR
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Construir y publicar imagen
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
```
![alt text](/images/workflow-fun.png)
## Evidencias Reto 4
•	Captura o registro del estado inicial del Deployment.
```powershell
kubectl get deployment web-deployment

kubectl get pods -l app=web -o wide

kubectl describe deployment web-deployment
```
CAPTURA TERMINAL
•	Captura o registro del cambio aplicado para soportar mayor tráfico.
cambia:

replicas: 2

por:

replicas: 6
```powershell
# Aplicar el cambio
kubectl apply -f .\k8s\deployment.yaml

kubectl get deployment web-deployment

kubectl get pods -l app=web
```
CAPTURA TERMINAL

•	Captura o registro de la estrategia de despliegue utilizada.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 6
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: web
```
```powershell
kubectl apply -f .\k8s\deployment.yaml
```

•	Captura o registro de tráfico de prueba durante el despliegue.
```powershell
# Preparar la imagen de la nueva versión
docker build -t repaso-sd:reto4-v2 .
# cárgar nueva imagen en Minikube
minikube image load repaso-sd:reto4-v2

kubectl port-forward service/web-service 8081:80
# Peticiones
while ($true) { curl.exe -s http://localhost:8081; Start-Sleep 1 }
```

•	Captura o registro que demuestre que la aplicación siguió respondiendo durante o después de la actualización.
