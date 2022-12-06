# Redbee Challenge - Simpsons Quotes API

### Requerido
 - Docker Desktop: https://www.docker.com/products/docker-desktop/
 - Kubernetes on Docker Desktop: https://docs.docker.com/desktop/kubernetes/
 
 
 ## Subir la imagen del contenedor a un registry
 **Docker Hub**
 - Crear una cuenta en Docker Hub: https://hub.docker.com/
 - Crear un repositorio en Docker Hub: https://docs.docker.com/docker-hub/repos/

**Construir la imagen de contenedor**
Desde el repositorio clonado en local

```bash
   # Ingresar a la carpeta "api"
   cd simpsons-quotes/api/
   
   # Build de la imagen. El tag queda a criterio
   docker build -t simpsons-quotes:0.1.3 .
```

**Login a Docker Hub desde local**

```bash
   # Ingresar las credenciales utilizadas en la creación de la cuenta en Docker Hub
   docker login
```
**Push al Registry**
```bash
   # Taggear la imagen de contenedor con el formato de Docker Hub
   # Formato -> usuario/nombre-imagen:tag . Ejemplo a continuación
   docker tag simpsons-quotes:0.1.3 jothanpe/simpsons-quotes:v1.0.0

   # Subir la imagen al registry
   docker push jothanpe/simpson-quotes-api:v1.0.0
```

## Pasos previos al deploy en el cluster de Kubernetes
**Instalar el Nginx Ingress Controller**
```bash
   # Comando para la instalación
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.5.1/deploy/static/provider/cloud/deploy.yaml

   # En pasos posteriores se creará un ingress.
   # En caso se produzca el siguiente error: "certificate signed by unknown authority"
   # ejecutar el siguiente comando:
   kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
```
**Habilitar una ruta para el File Sharing de la BD**
Se puede habilitar el File Sharing entre Docker Desktop y el Host siguiendo esta documentación: https://docs.docker.com/desktop/settings/mac/#file-sharing

La ruta seleccionada deberá tener el archivo **alta_db.sql** ubicado en la ruta "simpsons-quotes/db/" del repositorio. Esto para la inicialización de la BD cuando arranque el contenedor y sin la necesidad de conectarse al contenedor y ejecutar el script manualmente.

## Deploy de los manisfiestos en Kubernetes
**Creación de los Secrets**
```bash
   cd simpsons-quotes/manifests/

   # Tanto el contenedor de la BD como el del API harán referencia a este secret.
   kubectl apply -f secrets.yaml
```
**Deploy de la BD**
```bash
   # Modificar el hostPath (línea 14) referenciado la ruta con la que se habilitó el File Sharing.
   # No olvidar que la ruta debe contener el archivo alta_db.sql
   kubectl apply -f db.yaml
```
**Deploy del API**
```bash
   # De ser necesario, modificar el nombre y tag de la imagen (línea 19).
   kubectl apply -f api.yaml
```
**Deploy del Ingress**
```bash
   # Finalmente hacer el apply del Ingress que utiliza el Nginx Controller
   kubectl apply -f ingress.yaml
```
## Validación
Por default con la configuración realizada previamente se debe habilitar el siguiente DNS interno para probar el API.

    http://kubernetes.docker.internal

Como también se puede realizar pruebas utilizando el localhost

    http://127.0.0.1

Ejecutar lo siguiente para realizar la prueba del API funcional
```bash
   curl -vvv "http://kubernetes.docker.internal/quotes" -s
   # Como también
   curl -vvv "http://127.0.0.1/quotes" -s
```
Para el API Docs desde el navegador esta es la URL

    http://kubernetes.docker.internal/docs
    http://127.0.0.1/docs
