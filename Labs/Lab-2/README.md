# Instrucciones Laboratorio 2

## Desplegar aplicaciones desde linea de comando y uso de configmaps, variables de entorno, secrets y almacenamiento persistente estático

Requisitos:

* Logarse en el cluster con el comando "oc login ..."

* Definir la variable GUID:

      $ export GUID=<iniciales>

### 1. Uso de Variables de entorno

  1.1. Crear el proyecto y desplegar una aplicacion desde un repositorio de Git:

    $ oc new-project $GUID-env
    $ oc create deployment my-nginx --image=nginx --port=80

  1.2. Ver los eventos:

    $ oc get pods
    $ oc describe pod <nombre del pod>
    $ oc adm policy add-scc-to-user anyuid system:serviceaccount:<proyecto>:default
    $ oc delete pod <nombre del pod>

  1.3. Crear un servicio y route para la app:

    $ oc expose deploy/my-nginx
    service/my-nginx exposed
    
    $ oc get svc
    NAME       TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
    my-nginx   ClusterIP   10.2.83.169   <none>        80/TCP    73s

    $ oc expose svc my-nginx
    route.route.openshift.io/my-nginx exposed
    
    $ oc get route
    NAME       HOST/PORT                                     PATH   SERVICES   PORT   TERMINATION   WILDCARD
    my-nginx   my-nginx-lgp-formacion.apps.ocpdes.rtve.int          my-nginx   80                   None

  1.4 Ver los recursos del projecto. Comprobar como se han creado los objetos deployment config, pods, secrets, etc:

    $ oc get all

  1.5. Comprobar que variables de entorno estan definidas en el pod en ese momento:

    $ oc exec <pod-nginx> -- printenv |grep APP

  1.6. Configurar dos variables de entorno a la aplicacion:

    $ oc set env deploy/my-nginx APP_VAR_1=Value1 APP_VAR_2=Value2

  1.7. Esperar a que la aplicacion redespliegue:

    $ watch oc get pod

  1.8. Cuando el pod este corriendo pulsar Ctrl+C para salir del watch y continuar.

  1.9. Comprobar nuevamenete las variables de entorno definidas dentro del pod:

    $ oc exec <pod-nginx> -- printenv |grep APP
    APP_VAR_1=Value1
    APP_VAR_2=Value2
    
  1.10. Borrar la segunda variable de entorno y volver a comprobarlas con el "oc exec":

    $ oc set env dc/my-nginx APP_VAR_2-

  1.9. Comprobar que la variable ya no esta definida:

    $ oc exec <pod-nginx> -- printenv |grep APP
    APP_VAR_1=Value1
   
### 2. Uso de ConfigMaps

  2.1. Crear un ConfigMap con 2 variables de entorno:

    $ oc create configmap nginx-config \
     --from-literal=APP_VAR_3=Value3 \
     --from-literal=APP_VAR_4=Value4

  2.2. Actualizar el deployment de la aplicacion para añadir las variables definidas en el configmap:

    $ oc edit deploy my-nginx
    
    spec:
      containers:
      - env:
        - name: APP_VAR_1
          value: Value1
        - name: APP_VAR_3
          valueFrom:
            configMapKeyRef:
              key: APP_VAR_3
              name: nginx-config
        - name: APP_VAR_4
          valueFrom:
            configMapKeyRef:
              key: APP_VAR_4
              name: nginx-config

  2.3. Para salir del edit hay que guardar la configuracion ":wq!"

  2.4. Esperar a que la aplicacion redespliegue:

    $ watch oc get pod

  2.5. Cuando el pod este corriendo pulsar Ctrl+C para salir del watch y continuar.

  2.6. Comprobar nuevamenete las variables de entorno definidas dentro del pod:

    $ oc exec <pod-nginx> -- printenv |grep APP
    APP_VAR_4=Value4
    APP_VAR_1=Value1
    APP_VAR_3=Value3

  2.7. Definir una variable de entorno que lee el contenido de un fichero:

    $ oc set env deploy/my-nginx READ_FROM_FILE=/data/configfile.txt

  2.8. Crear el fichero de configuracion:

    $ echo "This is a very important Config File" > configfile.txt

  2.9. Crear un ConfigMapp:

    $ oc create configmap nginx-config-file --from-file=configfile.txt

  2.10. Añadir el ConfigMap a la configuracion del deployment:

    $ oc set volume deploy/my-nginx --add --overwrite --name=config-volume -m /data/ -t configmap --configmap-name=nginx-config-file

  2.11. Encontrar el pod que este Running:

    $ oc get pods

  2.12. Abrir una shell en el pod (en este ejemplo es el printenv-6-n8zj6) y comprobar el contenido del fichero:

    $ oc rsh <pod-nginx>

    sh-4.2$ ls /data/configfile.txt
    /data/configfile.txt

    sh-4.2$ cat /data/configfile.txt
    This is a very important Config File

## 3. Uso de secrets para configurar variables de entorno

  3.1. Crear un secret desde un fichero:

    $ echo 'r3dh4t1!' > ./password.txt
    $ echo 'admin' > ./user.txt
    $ oc create secret generic nginx-secret --from-file=app_user=user.txt --from-file=app_password=password.txt
    $ oc describe secret nginx-secret

  3.2. Comprobar el secret:

    $ oc get secret nginx-secret -o yaml

    apiVersion: v1
    data:
      app_password: cjNkaDR0MSEK
      app_user: YWRtaW4K
    kind: Secret
    [...]

  3.3. Decodificar el UserID y el pass usando *base64 --decode*:

    $ echo "cjNkaDR0MSEK" | base64 --decode
    $ echo "YWRtaW4K" | base64 --decode  

  3.4. Añadir el secret a la app *printenv*:

    $ oc set env deploy/my-nginx --from=secret/nginx-secret

  3.5. Verificar que el valor de las variables es el correcto:

    $ oc set env deploy/my-nginx --list

  3.6. Configurar las mismas variables de entorno para el MySQL añadiendo el prefijo *MYSQL_*:

    $ oc set env deploy/my-nginx --from=secret/nginx-secret --prefix=MYSQL_
    $ oc set env deploy/my-nginx --list

## 4. Uso de almacenamiento persistente

  4.1. Crear un PVC: 

     $ vi pvc-dinamic.yaml
     kind: PersistentVolumeClaim
     apiVersion: v1
     metadata:
       name: test-claim
       namespace: lgp-formacion
     spec:
       accessModes:
       - ReadWriteMany
       resources:
         requests:
           storage: 1Mi
       storageClassName: ibm-spectrum-scale-csi-consistency-group-delete
       volumeMode: Filesystem

        $ oc create -f pvc-dinamic.yaml

  4.2. Comprobar que se ha creado el PVC

       $ oc get pvc -n $GUID-env

  4.3. Configurar en el deploy de nginx como volumen el pvc que hemos creado y que lo monte en /usr/share/nginx/html.

       $ oc set volume deployment/my-nginx --add --name=myvol --mount-path=/usr/share/nginx/html -t pvc --claim-name=<nombre-pvc>

  4.4. Comprobar que el pod de nginx arranca correctamente y que tiene montado el PVC

       $ oc get pod
       $ oc describe pod nginx-xxx

  4.5. Eliminar el volumen del deployment de nginx que añadimos en el punto 4.8, borrar tambien el pvc y el pv:

       $ oc set volume deployment/my-nginx --remove --name=myvol
       $ oc delete pvc <nombre-pvc>



