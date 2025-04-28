
1. Craete image with volume
2. Copy into image
3. Create podman network
4. Create volumes for images
5. build dockerfile with predefined parameters and start command within dockerfile
6. set parameters to docker file
7. 

# OCP/CRC quick quecards

## OC cli help
```
kubectl version --client
kubectl --help
kubectl create --help
kubectl explain pod
```

## OC cluster top info
```
oc whoami --show-console
oc new-project myapp
oc project myapp
oc cluster-info
oc api-versions
```
#### OC api-resources
```
oc api-resources
oc api-resources --namespaced
oc api-resources --namespaced=false
oc api-resources --namespaced=false -o name | wc -l
oc api-resources --api-group ''
oc api-resources --api-group apps
oc api-resources --api-group config.openshift.io
```

## OC cluster health
```
#operator lifecycle manager (OLM) Operator
oc get operators
oc get pods -n openshift-dns-operator

#cluster version operator (CVO) manages
oc get clusteroperators
oc get clusteroperator dns -o yaml

oc describe clusteroperators openshift-apiserver

oc get pod -n openshift-apiserver-operator <pod-name-from-prev-command> -o json | jq .status
oc get pod -n openshift-dns-operator dns-operator-***** -o json | jq .status

oc get pod -n openshift-dns-operator dns-operator-**** -o json | jq .status 
oc get pod -n oepnshift-dns-operator

oc adm top pods -A --sum
oc get pods -n openshift-etcd --show-labels

oc adm top pods -n openshift-etcd --containers

oc adm top pods -n openshift-etcd --containers 


oc get all -n openshift-monitoring --show-kind
oc logs alertmanager-main-0 -n openshift-monitoring
oc get events -n openshift0image-registry
oc get nodes
oc adm top node

oc get node master01 -o jsonpath=\
'Allocatable: {.status.allocatable.cpu}{"\n"}'\
'Capacity: {.status.capacity.cpu}{"\n"}'

oc get node master01 -o jsonpath='{.status allocatable.pods}{"\n"}'

```



## SKOPEO image manipulation
```
oc login -u developer -p developer \
  https://api.ocp4.example.com:6443

oc new-project pods-images

skopeo login registry.ocp4.example.com:8443

# view the configuration for the image. 
skopeo inspect --config \
docker://registry.ocp4.example.com:8443/redhattraining/bitnami-mysql:8.0.31 

# retrieve a list of available tags for the registry
skopeo list-tags \
  docker://registry.ocp4.example.com:8443/redhattraining/docker-nginx
 
skopeo inspect --config \
docker://registry.ocp4.example.com:8443/redhattraining/docker-nginx:1.23    # view the configuration for the image.  
  
oc run bitnami-mysql \
  --image registry.ocp4.example.com:8443/redhattraining/bitnami-mysql:8.0.31  
  
```
## OCP template manipulation and deployment
```
oc get templates -n openshift
oc get template mysql-persistent -n opeshift -o yaml
oc get templates -n openshift -o custom-columns=NAME:.metadata.name|grep -i ^postgres

# explore and get template parameters
oc describe template mysql-persistent -n openshift
oc process --parameters mysql-persistent -n openshift 

#generate list of resources for creating new application
oc process -f <filename>
# in yaml format
oc process -o yaml -f <filename>
oc process -o -f filename  > myapp.yaml
oc process -o yaml -f mysql.yaml \
 -p MYSQL_USER=dev -p MYSQL_PASSWORD =$P4SSD -p MYSQL_DATABASE=bank \
 -p VOLUME_CAPACITY=10Gi > mysqlProcessed.yaml
oc process -f mysqlProcessed.yaml
oc process -f mysql.yaml -p MYSQL_USER=dev \
  -p MYSQL_PASSWORD=$P4SSD -p MYSQL_DATABASE=bank \
  -p VOLUME_CAPACITY=10Gi | oc create -f -

#Full template sequence
#1. export template
oc get template mysql-persistence -o yaml -n openshift > mysql-persistence-template.yaml
#2. set parameters and create
oc process -f mysql-persistent-template.yaml \
 -p MYSQL_USER=dev -p MYSQL_PASSWORD=$P4SSD -p MYSQL_DATABASE=bank \
 -p VOLUME_CAPACITY=10Gi | oc create -f - 


```


## OCP multicontainer app deployment
```
oc login -u developer -p developer https://api.ocp4.example.com:6443
oc new-project deploy-newapp
oc get all
oc get serviceaccounts
oc get secrets
oc describe template mysql-persistent -n openshift # vew content of template

#create new instance with TEMPLATE and label "red"
oc new-app -l team=red --template mysql-persistent \
   -p MYSQL_USER=developer \
   -p MYSQL_PASSWORD=developer
oc get pods   # check te created resources. Can use -w option

#create new instance with IMAGE and label "blue"
oc new-app --name db-image -l team=blue \
 --image registry.ocp4.example.com:6443/rehel9/mysql-80:1 \
 -e MYSQL_USER=developer \
 -e MYSQL_PASSWORD=developer \ 
 -e MYSQL_ROOT_PASSWORD=redhat

#see resources by label (capital L)
oc get pods -L team

# check pod created by template and see readiness probe and limits
oc get pods -l deploymentconfig=mysql \
 -o jsonpath='{item[0].spec.containers[0].readinesProbe}' |jq

oc get pods -l deploymentconfig=mysql \
 -o jsonpath='{.item[0].spec.containers[0].resources.limits}' | jq   

# check pod created by image and see readiness probe and limits
oc get pods -l deploymentconfig=db-image \
 -o jsonpath='{itiems[0].spec.comtainers[0].readinessProbe}' | jq
 
oc get pods -l deploymentconfig=dm-image \
  -o jsonpath='{items[0].spec.containers[0].resources}' | jq
 
oc get secrets
oc get services
oc get services pods -l team=red

#cleanup by label
oc delete all -l team=red
oc delete secret, pvc -l team=red

#check deletion
oc get secret,pod,svc,pvs,dc -l team=red
oc get is,deployment,svc
  
```
## OCP long-lived and short-lived app deployment
```
oc login -u developer -p developer \
   https:\\api.ocp4.example.com:6443

oc new-project deploy-workloads
# deployment that runs an ephemeral MySQL server

oc create deployment my-db \
  --image registry.ocp4.example.com:8443/rhel9/mysql-80:1  

#check for errors
oc get deployments
oc get pods  
oc logs deploy/my-db 
# log shows environment variables missing

#set environment variables
oc set env deploymen/my-db \
 MYSQL_USER=developer \
 MYSQL_PASSWORD=developer \
 MYSQL_DATABASE=sampledb

#check status
oc get deployments
oc get pods -o wide
 
# test DB running using IP from the last command
oc run -it db-test --restart=Never \
  --image registry.ocp4.example.com:8443/rhel9/mysql-80:1 \
  -- mysql sampledb -h 10.8.0.91  -u developer --password=developer \
  -e "select 1"
 
#delete pod, not deployemnt
oc delete pod -l app=my-db
# its recreated
oc get pod -l app=my-deb   

#create job with printing date loop
oc create job date-loop \
 --image registry.ocp4.example.com:8443/ubi9/ubi \
 -- /bin/bash -c "for i in {1..30}; do date; done"

# check results
oc get job date-loop -o yaml
oc get jobs
oc logs job/date-loop

#cleanup
oc delete pod -l job-name=date-loop
oc get pod -l job-name=date-loop
oc get job -l job-name=date-loop  
```

## OCP app exposing and scaling
```
oc login -u developer  -p developer \
   https://api.ocp4.example.com:6443
oc new-project web-application
# create web app from image
oc create deployment satir-app --image registry.ocp4.example.com:8443/redhattraining/do180-httpd-app:v1
# check status of app
oc get pods 
oc status
oc create deployment sakila-app --image registry.ocp4.example.com:8443/redhattraing/do180-httpd-app:v01
oc get pods
oc status


#  creating services from deployments by expose command 
oc expose deployment satir-app --name satir-svc --port 8080 --target-port 8080
oc expose deployment sakila-app --name sakila-svc --port 8080 --target-port 8080
oc get services
oc get endpoints
oc get pods -o wide

# 1. exposing created services
oc expose service satir-svc --name satir
oc get routes

# 2. ALTERNATIVE: create ingress for the route
oc create ingress ingr-sakila --rule "ingr-sakila.apps.ocp4.example.com/*=sakila-svc:8080"


#only service expose port. ingress has <all> ports
oc get ingress
oc get routes

# test using curl and url from routes output
curl ingr-sakila.apps.ocp4.example.com
curl satir-web-application.apps.ocp4.example.com

#scale
oc scale deployment sakila-app --replicas 2
oc get pods
oc scale deployment satir-app --replicas 3
oc get pods -o wide
oc get endpoints

#configure cookie for INGRESS
oc annotate ingress ingr-sakila ingress.kubernetes.io/affinity=cookie

oc annotate ingress <controller-name> ingress.kubernetes.io/affinity=cookie

# test curl in loop
for i in {1..3}; do curl ingr-sakila.apps.ocp4.example.com; done
curl ingr-sakila.apps.ocp4.example.com -c /tmp/cookie_jar
#check cookies
cat /tmp/cookie_jar
# using the same pod via cookies
for i in {1..3}; do curl ingr-sakila.apps.ocp4.example.com -b /tmp/cookie_jar; done
# using different pods
for i in {1..3}; do curl ingr-sakila.apps.ocp4.example.com: done

#configure cookie for SERVICE
oc annotate route satir router.openshift.io/cookie_name="hello"
for i in {1..3}; do curl satir-web-application.apps.ocp4.example.com; done
curl satir-web-application.apps.ocp4.example.com -c /tmp/cookie_jar

for i in {1..3}; do curl satir-web-application.apps.ocp4.example.com -b /tmp/cookie_jat; done
for i in {1..3}; do curl satir-web-applicatiom.apps.ocp4.example.com; done 

THE END
``` 
## OCP app nerworking and network management
```
oc login -u developer -p developer https://api.ocp4.example.com:6443
oc project database-application

1. Create DB
oc create deployment mysql-app --image registry.ocp4.example.com:8443/redhattraing/mysql-app:v1

oc get pods
oc status
oc logs mysql-.....

oc set env deployment/mysql-app \
  MYSQL_USER=redhat
  MYSQL_PASSWORD=redhat123
  MYSQL_DATABASE=world_x

oc get pods

oc exec -ti mysql-app.... \
 -- /bin/bash -c "mysql -u redhat -p redhat123 </tmp/world_x.sql"  
   
oc rsh  mysql-app../   

mysql -u redhat -p redhat123 world_x

oc expose deployment mysql-app --name mysql-service --port 8080 --target-port 8080

oc get services
oc get endpoints

2. Create web-contained on db
oc create deployment php-app --image registry.ocp4.example.com:8443/redhattraining/php-webapp:v1
oc get pods
oc status

oc expose deployment php-app --name php-svc --port 8080 --target-port 8080
oc get services
oc get endpoints

oc expose service/php-svc --name --phpapp
oc get routes

THE END
``` 

## OCP container storage
``` 
FROM UI
1. From UI create Deployments from Workloads - deployment name: webconfig
2. Create service from demployment using Networking>Services : webconfig-svc
3. Create Route from Networking : webconfig-rt
4. Create Configmap from Workloads : webfiles

FROM CLI

oc login -u developer -p developer https://api.ocp4.example.com:6443

oc set volume dempoyment/webconfig \
 --add --type configmap  --configmap-name webfiles \
 --mount-path /var/www/html/

oc status
oc get pods

``` 

## OCP persistent data volumes
```
In UI 
1. check nfs-stotage in Administrator>Stirage>StorageClasses
2. Workloads>Deployment Create Deployment , use image mysql-80
3. Advanced option > Scaling set scaling
4. Networking > Services > Create Service
5. Workloads > Deployments > Add storage > Create new claim
6. Check status Workloads > Deployment >dp-pod > Yaml

In CLI
oc login -u developer -p developer https://api.ocp4.example.com:6443
oc project storage-volumes

oc create configmap init-db-cm \
 --from-file=//////////

oc set volumes deployment/db-pod \
 --add --name init-db-volume --type configmap --configmap-name init-db-cm \
 --mount-path /var/db/config

oc rsh deployment/db-pod
mysql --uuser1 -pmypa55w0rd itmes </var/db/configinit-db/sql
mysql --uuser1 -pmypa55w0rd items -e 'select * from Item;'
exit 

#delete pod, recreate it and check that data is still threre
oc delete delete deployment/db-pod
oc get pods
oc create deployment db-pod --port 3306 --image=registry.ocp4.example.com:8443/rhel8/mysal-80
oc set env deployment/db-pod \
MYSQL_USER=user1 \
MYSQL_PASSWORD=mypa55w0rd \
MYSQL_DATABASE=Items
#set pvc back
oc set volumes deployment/db-pod \
 --add --type pvc \
 --mount-path /var/lib/mysql \
 --claim-name db-pod-pvc

oc run query-db -it --rm \
 --image registry.ocp4///
 --resyart Never \
  --/bin/bash -c "mysql -uuser -ppassword --protocol tcp \
  -h db-pod -P3306 items -e 'select * from Item;'"

oc delete deployments/db-pod
oc delete pvc/db-pod-pvc
 
```
## OCP select storage class for application
```
oc login -u developer -p developer https://api/ocp4.example.com:6443
oc  project storage classes 
oc get sc
oc describe sc lvms-vg1

oc create deployment db-pod --port 3306 \
 --image registry.ocp4.example,com:8443/rhel8/mysql-80
 
oc set env deployment/db-pod
 MYSQL_USER=user1 \
 MYSQL_PASSWORD=password \
 MYSQL_DATABASE=items
 
oc get pods
# create service from deployment
oc expose deployment/db-pod
#set volume for deployment
oc set volumes deployment/db-pod \
 --add --name  odf-lvm-storage --type pvc \
 --claim-mode rwo --claim-size 1Gi --mount-path /var/lib/mysql \
 --claim-class lcm-vg1 \
 --claim-name db-pod-odf-pvc 
 
oc get pvc
oc describe pvc db-pod-odf-pvc

oc run db-pod -ti --rm \
 --image registry.opc4.example.com:8443/rhel8/mysql-80 \
 --restart Never --command \
 --/bin/bash -c "mysql -u... -p.... P3306 0h db-pod --protocol tcp items -e 'show databases;'"
 
pc delete all -l app=db-pod
oc get pvc
oc delete pvc db-pod-odf-pvc         

#create pvc YAML for NFS class
oc create -f nfs-pvc.yaml
oc describe pvc nfs-pvc

oc create deployment web-pod --port 8080 \
 --image registry.ocp4.example.com:8443/uni8/httpd-24:1-215
oc expose deployment web-pod
oc expose svc web-pod --hostname web-pod.apps.example.com
oc get routes
curl http://web-pod.apps.ocp4.example.com/

oc set volumes deployment /web-pod \
 --add --name nfs-volume \
 --claim-name nfs-pvc \
 --mount-path /var/www/html

oc create deployment app-pod --port 9090 \
--image registry.example.com:8443/redhattraining/do180-roster

oc expose deployment app-pod
oc expose svc app-pod --hostname app-pod.apps.ocp4.example.com
oc set volumes deployment app-pod --add --name nfs-volume  --claim-name nfs-pvc --mount-path /var/tmp

```

## OCP storage with stateful sets
```
oc login -u developer -p developer  https://api.ocp4.example.com:6443

oc create deployment web-server --image registry.ocp4.example.com:8443/redhattraing/hello-world-nging:latest
oc get pods -l app=web-server

oc set volumes deployment/web-server \
--add --name web-pv --type persistentVolumeClaim --claim-mode rwo
--claim-size 1Gi --mount-path /usr/share/nginx/html --claim-name web-pv-claim

oc get pods -l app=web-server 
oc get pvc
#get pod running <pod-name>
oc get pods
oc exec -it pod/<pod-name> \
-- /bin/bash -c 'echo "Hello , World from ${HOSTNAME}" > /urs/share/nginx/html/index.html'
oc exec -it pod/webserver-pod /bin/bash -c 'echo "Hello, World from ${HOSTNAME}" > /usr/share/nginx/html/index.html'

oc scale deployment web-server --replicas 2
oc get pods 
oc exec -ti pod/web-server-pod cat /usr/share/nginx/html/index.html

#Create statefullset from YAML for splitting DBs per pv
oc create -f /..../..../statefulset-db.yaml 
oc get statefulset
oc get pods -l app=database
#get pods dbserver-0,1
oc exec -ti pod/dbserver-0 /bin/bash -c "mysql -uredhat -predhat123 sakila -e 'create table items (count INT);'"
oc exec -ti pod/dbserver-1 /bin/bash -c "mysql -uredhat -predhat123 sakila -e 'create table inventory (count INT);'" 

oc get pod dbserver-0 -o json | jq .spec.volumes[0].persistentVolumeClaim.claimName
oc get pod dbserver-1 -o json | jq .spec.volumes[0].persistentVolumeClaim.claimName

oc exec  -it pod/dbserver-1 /bin/bash -c "mysql -u... -p... sakila -e 'show tables;'"
```

# OC manage cluster data
```
oc login -u developer -p password htts://api.ocp4.example.com:6443
oc project storage-review
oc create secret generic world-cred \
 --from-literal user=redhat \
 --from-literal password=redhat123 \
 --from-literal database=world_x
oc get secres world-cred
oc create configmap dbfiles --from-file insertdata.sql
oc get configmaps

# creade deployment pod
oc create deployment dbserver --image registry.ocp4.example.com:8443/redhattraingng/mysql-app:v1
oc set env deployment/dbserver --from secret/world-cred --prefix MYSQL_
oc get pods

#set volumes from existing lvm
oc set volume deployment/dbserver  \
--add --name dbserver-lvm --type persistentVolumeClaim \
--claim-mode  rwo --claim-size iGi --mount-path /var/lib/mysql \
--claim-class lvms-vg1 --claim-name dbserver-lvm-pvc 
oc get pods
oc get pvc

#exose pod as a server
oc expose deployment dbserver --name mysql-service --port 3306 --target-port 3306
oc get services
oc get endpoints

oc create deployment file0sharinf --image registry.ocp4.example.com:8443/redhattraining/php-webapp-mysql:v1
oc get pods

oc scale deployment file-sharing --replicas 2
oc get pods 
oc expose deployment file-sharing --name file-sharing --port 8080 --target-port 8080
oc get services
oc get endpoints
oc expose service/file0sharing
oc get routes
# check in browser http://file-sharing-storage-review.apps.ocp4.example.com

#set volume from configmap dbfiles
oc set volume deployment/file-sharing \
--add --name config-map-pvc --type configmap \
--configmap-name db-files \
--mount-path /home/database-files

oc get pods and verify content
oc  exec -ti pod/<file-sharing> -- head /home/database-files/insertdata.sql

oc set volume deployment/file-sharing \
--add --name shared0volume  --type persistentVolumeClaim \
--claim-mode rwo --claim-size 1Gi --mount-path /homesharedfiles \
--claim-class nfs-storage --claim-name shared-pvc

oc get pods
oc get pvc
oc exet -ti pod/<file-sharing> -- cp /home/database-files/insertdata.sql /home/sharedfiles/

#remove pvc after sharing
oc set volume deployment/file-sharing --remove --name=config-map-pvc
# and add it to dbserver

oc set volume deployment/dbserver \
--add --name shared-volume \
--claim-name shared=pvc \
--mount-path /home/sharedfiles

oc get pods
oc rsh dbserver-<pod>
mysql -u$MYSQL_USER -p$MYSQL_PASSWORD world_x < /home/sharedfiles/insertdata.sql

#Last - check  http://file-sharing-storage-review.apps.ocp4.example.com
```

# OCP application HA

## HA application on K8S
```
oc login -u developer -p developer  https://api.ocp4.example.com:6443
oc project reliability-ha

#check restartPolicy as Always

cat long-load.yaml
oc apply -f long-load.yaml
oc exec long-load -- curl -s localhost:3000/health
oc get pods
oc exec long-load -- curl -s localhost:3000/destruct
oc get pods
oc delete pod long-load

# set restartPolicy in yaml to Never

oc apply -f long-loads.yaml  
oc exec long-load -- curl -s localhost:3000/health
oc exec long-load -- curl -s localhost:3000/destruct
oc get pods
oc delete pod long-load

#set delay on startup in yaml as env: -name :START_DELAY value: "6000" restartPolycy : Always

oc apply -f long-load.yaml
oc get pods 
oc exec long-load -- curl -s localhost:3000/health 
```

## Application health probes
```
oc login -u developer p developer https://api.ocp4.example.com:6443
oc project reliability-probes
.....
containers:
- image: registry.ocp4.example.com:8443/redhattraining/long-load:v1
imagePullPolicy: Always
name: long-load
startupProbe:
  failureThreshold: 30
  periodSeconds: 3
  httpGet:
    path: /health
    port: 3000

oc exec deploy/long-load --curl -s localhost:3000/togglesick

oc set probe deploy/long-load  \
--readiness --lailure-treshold 1 --period-seconds 3 \
--get-url http://3000/health

#simulate error on the pod for 5 seconds
oc exec deploy/long-load -- curl -s localhost:3000/hiccup?time=5

```
## Reserve Application compute capacity
```
oc login -u admin -p admin https://api.ocp4.example.com:6443
oc describe node master01
oc project reliability-request
oc apply -f long-load-deploy.yaml

#yaml
resources:
  requests:
    memory: 1Gi
    
oc get events --feild-selector reason="FailedScheduling"
oc describe pod/long-load-......
oc describe node master01

#fix it by increasing memory
oc set resources deploy/lond-load --requests memory=250Mi
oc delete pod -l app=long-load
oc get pods
oc describe node master01    

```

## Limit Application compute capacity

```
oc login -u developer -p developer https:api.ocp4.example.com:6443
oc project reliability-limits

#yaml 
resources:
  requests:
    memory: 20Mi
  limits:
    memory: 35Mi

oc apply -f leakapp.yml
oc get pods
watch oc get pods

oc get pods leakapp-<...> -o jsonpath='{.status.containerStatuses[0].lastState}' | jq .
oc set resources deployment/leakapp --limits memory=600Mi
```

## Application autoscaling
```
```

# Deploy web-app End-2-End
```
oc login -u developer -p - developer https://api.cop4.example.com:6443
oc new-project review
oc create istag mysql8:1 --from-image registry.ocp4.example.com:8443/rhel8/mysql-80:1-228
oc set image-lookup mysql8
#verify image streem lookup established 
oc set image-lookup mysql8:1

oc create secret generic dbparams \
--from-literal user=operator1 --from-literal password=redhat123 \
--from-literal database=quotesdb

#create deploument from image stream tag with no pods
oc create deployment quotesdb --image mysql80:1 --replicas 0

#all available and ready pods -0 , get the container name for triggers further
oc get deployment qoptesdb -o wide

#set triggers for image stream
oc set triggers deployment/quotesdb --from-image mysql8:1 --containers mysql8

#use secret for creating environments in deployment
oc set env deployment/quotesdb --from secret/dbparames --prefix MYSQL_

#check aivailable storage classes  
oc get sc 
 
#set volume based on available sc
oc set volumes deployment/quotesdb  --add --claim-class lvms-vg1 --claim-size 2Gi \
mount-path /var/lib/mysql

#make it available , from 0 to 1
oc scale deployment/quotesdb --replicas 1

oc get pods

#create service via expose
oc expose deployment/quotesdb  --port 3306

oc describe service quotesdb

#create frontend for db
oc create deployment frontend --iamge registry.ocp4.example.com:8443/..... --replicas 0

oc set env deployment/frontend --from secret/dbparams --prefix QUOTES_
oc set env deployment/frontend QUOTES_HOSTNAME=quetasdb

oc scale deployment/frontend --replicas 1
oc get pods
  
#expose frontend standard port http
oc expose deployment/frontend --port 8080
oc expose service frontend
oc get route

# finally, curl to route //

curl http://frontend.apps..... 

```

# Troubleshoot and scale application
```
In UI
1. Adnministrator view > Observe > Dashboards
2. Cluster in Dashboards > Inspect
3. Zoom in 5m > select grath with most consuming namespace
4. In grapphs select workloads and set name of Namespace in last 5 minutes 
5. select deployment consuming most CPU
6. From Workloads > delete faulty deployment

From CLI

oc login -u development -p development  https://api.cop4.example.com:6443
oc project compreview-scale
#check CrashLoopBackOff and pods name
get pods
oc logs quotes-pod-/////
# set missing encironment variables
oc get deployments/quotesdb -o wide

oc set image deployment/quiotesdb mysql-80=registry.ocp4.example.com:8443/rhel9/mysql-80
oc get deployment/quotesdb -o wide
oc get pods 

oc set probe deployment/quotesdb --readiness -- mysqladmin ping
oc set probe deployment/quotesdb --liveness  -- mysqladmin ping 

oc set resources deployment/quotesdb --requests cpu=200m, memory=256Mi --limits cpu=500m,memory=1Gi

oc set probe deployment/quotesdb --readiness --get-url http://:8000/status
oc set probe deployment/quotesdb --liveness  --get-url http://:8000/env

oc set resources deoloyment/frontend --requests cpu=200m , memory=256Mi --limits cpu=500m,memory=512Mi

oc scale deployment/frontend --replicas 3

oc get pods
oc get routes
# curl to routes to complete

```