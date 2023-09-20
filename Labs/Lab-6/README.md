# Instrucciones Laboratorio 6

## Visualizando Logs

1. Crear un nuevo proyecto:
       $ oc new-project <nombre-proyecto>
2. Desplegar aplicación desde la consola de Openshift vista Developer:
   
![alt Despliegue][imagen10]

[imagen10]: images/Despliegue.png

3. Desde el repositorio de código https://github.com/redhat-gpte-devopsautomation/PrintEnv

![alt Despliegue-1][imagen11]

[imagen11]: images/Despliegue-1.png

4. Comprobar que la aplicacion despliega correctamente:

       $ oc get all
       NAME                             READY   STATUS      RESTARTS   AGE
       pod/print-env-1-build            0/1     Completed   0          15m
       pod/print-env-5dd96db85d-8lt4v   1/1     Running     0          15m
       
       NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
       service/print-env   ClusterIP   10.2.108.239   <none>        8080/TCP   15m
       
       NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
       deployment.apps/print-env   1/1     1            1           15m
       
       NAME                                   DESIRED   CURRENT   READY   AGE
       replicaset.apps/print-env-5dd96db85d   1         1         1       15m
       replicaset.apps/print-env-64db4cbf99   0         0         0       15m
       
       NAME                                       TYPE     FROM   LATEST
       buildconfig.build.openshift.io/print-env   Source   Git    1
       
       NAME                                   TYPE     FROM          STATUS     STARTED          DURATION
       build.build.openshift.io/print-env-1   Source   Git@fa7b607   Complete   15 minutes ago   34s
       
       NAME                                       IMAGE REPOSITORY                                        TAGS     UPDATED
       imagestream.image.openshift.io/print-env   image-registry.apps.ocpdes.rtve.int/lgp-lab/print-env   latest   15 minutes ago

       NAME                                 HOST/PORT                                PATH   SERVICES    PORT       TERMINATION     WILDCARD
       route.route.openshift.io/print-env   print-env-lgp-lab.apps.ocpdes.rtve.int          print-env   8080-tcp   edge/Redirect   None

5. Comprobar los logs de la aplicacion por linea de comandos:

        $ oc logs <nombre del pod>

6. Comprobar los eventos del proyecto:

        $ oc get events

3. Crear un filtro en Kibana para ver los logs de la aplicacion printenv a partir del nombre del proyecto, en el ejemplo es lgp-lab:

![alt Kibana-1][imagen9]

[imagen9]: images/kibana-1.png

![alt Kibana-2][imagen12]
        
[imagen12]: images/kibana-2.png

4. Comprobar que seleccionando ese filtro unicamente aparecen los logs relacionados con el namespace por el que se esta filtrando:

![alt Kibana-3][imagen13]

[imagen13]: images/kibana-3.png

5. Hacer una busqueda por algun mensaje que salga en los logs del pod, por ejemplo "DEBUG_PORT=5858" o cualquier otro:

![alt Kibana-4][imagen14]

[imagen14]: images/kibana-4.png

## Monitorizacion

1. Acceder a las métricas. En la consola de OpenShift opcion **Observe** -> **Targets**

![alt Prometheus][imagen1]

[imagen1]: images/lab-prometheus1.png

2. Comprobar si todos los Targets estan en estado **UP**

3. Ir a la Opcion **Metrics** para probar varias queries de Prometheus.

4. En la entrada **Expresion** escribir la siguiente lo siguiente y pulsar enter:

        max by (namespace, name) (cluster_operator_up{job="cluster-version-operator"} == 0) 

5. Comprobar si hay algun operador que esté dando error:

6. Probar mas queries:

* node_memory_MemTotal_bytes / 1024 /1024
* node_memory_MemFree_bytes /1024 /1024
* round(node_memory_MemFree_bytes/1024/1024)

7. Borrar la Expresion y empezar a escribir **node**. Ver la lista de expresiones que se despliega y experimente con otras queries de nodo. Ejemplo:

* node_namespace_pod_container:container_cpu_usage_seconds_total:sum_irate

## Dashboards de Grafana

1. Acceder a los Dashboards. En la consola de OpenShift opcion **Observe** -> **Dashboards**

![alt Grafana][imagen5]

[imagen5]: images/lab-grafana1.png

2. Buscar la grafica **Kubernetes / Compute Resources / Cluster**. Esta grafica muestra un overview del cluster incluida la utilización de cpu, memoria.

3. Explorer los gráficos por ejemplo:

* Ver los gráficos para un solo proyecto.
* Cambiando los rangos de tiempo (parte superior derecha de la ventana)
* Ver graficos para cada nodo.

4. Explorar los graficos de utilización de los nodos **USE Method / Node** y **USE Method / Cluster**

## Alertas

1. Ir a las alertas en la consola de Openshif **Observe** -> **Alerting**
2. Explorar las alertas activas en el clúster
3. Explorar las Alerting Rules
