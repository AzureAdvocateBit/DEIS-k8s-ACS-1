# Install Helm and DEIS Workflow on ACS


## 1. Preparation: Install Client CLI

### Install deis command

After login to the master node, please execute following  command?

```
yosshi@k8s-master-27AF23F9-0:~$ curl -sSL http://deis.io/deis-cli/install-v2.sh | bash
Downloading deis-stable-linux-amd64 From Google Cloud Storage...

The Deis Workflow CLI (deis) is now available in your current directory.

To learn more about Deis Workflow, execute:

    $ ./deis --help

yosshi@k8s-master-27AF23F9-0:~$ ls
deis
yosshi@k8s-master-27AF23F9-0:~$ sudo mv deis /usr/local/bin/
```

If you need to get more detail information, please refer to [deis command install](https://deis.com/docs/workflow/quickstart/install-cli-tools/)?  


### Install helm command

 please execute following command?

```
yosshi@k8s-master-27AF23F9-0:~$ wget https://kubernetes-helm.storage.googleapis.com/helm-v2.4.1-linux-amd64.tar.gz
--2017-05-27 12:21:49--  https://kubernetes-helm.storage.googleapis.com/helm-v2.4.1-linux-amd64.tar.gz
Resolving kubernetes-helm.storage.googleapis.com (kubernetes-helm.storage.googleapis.com)... 172.217.24.144, 2404:6800:4004:806::2010
Connecting to kubernetes-helm.storage.googleapis.com (kubernetes-helm.storage.googleapis.com)|172.217.24.144|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 14905955 (14M) [application/x-tar]
Saving to: ‘helm-v2.4.1-linux-amd64.tar.gz’

helm-v2.4.1-linux-amd64.tar.gz      100%[===================================================================>]  14.21M  29.4MB/s    in 0.5s    

2017-05-27 12:21:51 (29.4 MB/s) - ‘helm-v2.4.1-linux-amd64.tar.gz’ saved [14905955/14905955]

yosshi@k8s-master-27AF23F9-0:~$ ls
helm-v2.4.1-linux-amd64.tar.gz
yosshi@k8s-master-27AF23F9-0:~$ tar xvfz helm-v2.4.1-linux-amd64.tar.gz 
linux-amd64/
linux-amd64/helm
linux-amd64/LICENSE
linux-amd64/README.md
yosshi@k8s-master-27AF23F9-0:~$ sudo mv linux-amd64/helm /usr/local/bin/
```

If you need to get more detail information, please refer to [helm command install](https://github.com/kubernetes/helm#install)?

After installed both deis and helm to /usr/local/bin, you can get the list as follows.

```
yosshi@k8s-master-27AF23F9-0:~$ ls /usr/local/bin/
deis  helm  kubectl
```

## 2. Preparation: Create Azure Storage Account

Defaul storage of DEIS is caled minio. And it is not enough. So instead of the minio, I recommend to use the dedicated storage for DEIS. In order to use the Azure Storege for DEIS, please create new Stroage Account?  

On the Azure Portal, please push the "+ New" button? Then you can see the followign screen. After that, please select the **"Storage"** on the menu? 

![](https://c1.staticflickr.com/5/4200/34113075583_3829ab6e76_z.jpg)

Then you can see the additional menu. Pleaese push the "Storage account -blob, file, table, queue" on the "FEATURED APPS"? 

![](https://c1.staticflickr.com/5/4251/34113075673_22e97af0ec_z.jpg)

Then you can see the following screen. Please fill out all field for your environment?

![](https://c1.staticflickr.com/5/4269/34082666134_a96bc467a9_z.jpg)

For example, input following on this environment.

|Field name|Value|
|---|---|
|Name|storage4deis|
|Deployment model|Resource Manager|
|Account kind|General Purpose|
|Performance|Standard|
|Storage service encryption|Disable|
|Subscription|*********|
|Resource Group|k8s-ACS|
|Location|Japan West|

Finally, please push the "Create" button? After for a while, you can see like following screen.

![](https://c1.staticflickr.com/5/4225/34883848556_4f009093cb_z.jpg)


## 3. Install Helm and DEIS WorkFlow

For detail installation, please refer to the [Install Deis Workflow on Azure Container Service](https://deis.com/docs/workflow/quickstart/provider/azure-acs/install-azure-acs/)?

### 3.1 Initialize of helm
At first, please initialize the helm by usin **helm init** command?

```
yosshi@k8s-master-27AF23F9-0:~$ helm init --upgrade
Creating /home/yosshi/.helm 
Creating /home/yosshi/.helm/repository 
Creating /home/yosshi/.helm/repository/cache 
Creating /home/yosshi/.helm/repository/local 
Creating /home/yosshi/.helm/plugins 
Creating /home/yosshi/.helm/starters 
Creating /home/yosshi/.helm/repository/repositories.yaml 
$HELM_HOME has been configured at /home/yosshi/.helm.

Tiller (the helm server side component) has been installed into your Kubernetes Cluster.
Happy Helming!
```

### 3.2 Add the Deis Chart Repository
The Deis Chart Repository contains everything needed to install Deis Workflow.  

```
yosshi@k8s-master-27AF23F9-0:~$ helm repo add deis https://charts.deis.com/workflow
"deis" has been added to your repositories
```

### 3.3 Set the Strage Account Info to Environment Value


**Note (Local Machine): Please execute following 3 command**  
On your local machine which installed Azure CLI (az) command, please execute the following command?

```
# Please specify the Storage Account name and 
# Resource Group name?

$ export AZURE_SA_NAME=storage4deis
$ export AZURE_RG_NAME=k8s-acs

# Please get the Storage Account keys

$ az storage account keys list -n $AZURE_SA_NAME -g $AZURE_RG_NAME --query [0].value --output tsv
***/*******************/**************************************************************==
```

**Note (Master node): Please execute following 3 command**  
On the master node, please execute the following command?  

```
# For Environment Value "AZURE_SA_NAME" and "AZURE_RG_NAME" 
# is same as the above

yosshi@k8s-master-27AF23F9-0:~$ export AZURE_SA_NAME=storage4deis
yosshi@k8s-master-27AF23F9-0:~$ export AZURE_RG_NAME=k8s-acs

# Copy & Paste from the result of above 
# "az storage account keys"

yosshi@k8s-master-27AF23F9-0:~$ export AZURE_SA_KEY="***/*******************/**************************************************************=="
```

### 3.4 Install DEIS WorkFlow
In order to install the DEIS workflow, please execute the following command?  

```
yosshi@k8s-master-27AF23F9-0:~$ helm install deis/workflow --namespace=deis --set global.storage=azure,azure.accountname=$AZURE_SA_NAME,azure.accountkey=$AZURE_SA_KEY,azure.registry_container=registry,azure.database_container=database,azure.builder_container=builder
Error: secrets "builder-key-auth" already exists
yosshi@k8s-master-27AF23F9-0:~$ helm install deis/workflow --namespace=deis --set global.storage=azure,azure.accountname=$AZURE_SA_NAME,azure.accountkey=$AZURE_SA_KEY,azure.registry_container=registry,azure.database_container=database,azure.builder_container=builder
NAME:   sad-maltese
LAST DEPLOYED: Sat May 27 14:54:07 2017
NAMESPACE: deis
STATUS: DEPLOYED

RESOURCES:
==> v1/Secret
NAME                   TYPE    DATA  AGE
deis-router-dhparam    Opaque  1     8s
minio-user             Opaque  2     8s
objectstorage-keyfile  Opaque  5     8s

==> v1/ConfigMap
NAME                  DATA  AGE
slugbuilder-config    2     8s
slugrunner-config     1     8s
dockerbuilder-config  2     8s

==> v1/ServiceAccount
NAME                   SECRETS  AGE
deis-logger            1        8s
deis-monitor-telegraf  1        8s
deis-nsqd              1        8s
deis-builder           1        8s
deis-workflow-manager  1        8s
deis-controller        1        8s
deis-database          1        7s
deis-router            1        7s
deis-registry          1        7s
deis-logger-fluentd    1        7s

==> v1/Service
NAME                    CLUSTER-IP    EXTERNAL-IP  PORT(S)                                                   AGE
deis-monitor-influxui   10.0.87.175   <none>       80/TCP                                                    7s
deis-monitor-grafana    10.0.169.98   <none>       80/TCP                                                    7s
deis-builder            10.0.167.169  <none>       2222/TCP                                                  7s
deis-nsqd               10.0.67.111   <none>       4151/TCP,4150/TCP                                         7s
deis-logger-redis       10.0.114.221  <none>       6379/TCP                                                  7s
deis-router             10.0.162.135  <pending>    80:30511/TCP,443:31724/TCP,2222:31665/TCP,9090:32202/TCP  7s
deis-monitor-influxapi  10.0.79.154   <none>       80/TCP                                                    7s
deis-controller         10.0.147.242  <none>       80/TCP                                                    7s
deis-database           10.0.194.94   <none>       5432/TCP                                                  6s
deis-logger             10.0.80.244   <none>       80/TCP                                                    6s
deis-registry           10.0.51.81    <none>       80/TCP                                                    6s
deis-workflow-manager   10.0.117.222  <none>       80/TCP                                                    6s

==> v1beta1/DaemonSet
NAME                   DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE-SELECTOR  AGE
deis-registry-proxy    6        6        2      0           0          <none>         6s
deis-logger-fluentd    6        6        4      0           0          <none>         6s
deis-monitor-telegraf  6        6        3      0           0          <none>         6s

==> v1beta1/Deployment
NAME                   DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
deis-controller        1        1        1           0          6s
deis-monitor-influxdb  1        1        1           0          6s
deis-logger-redis      1        1        1           0          6s
deis-logger            1        1        1           0          6s
deis-database          1        1        1           0          6s
deis-registry          1        1        1           0          6s
deis-nsqd              1        1        1           0          6s
deis-workflow-manager  1        1        1           0          6s
deis-builder           1        1        1           0          5s
deis-monitor-grafana   1        1        1           0          5s
deis-router            1        1        1           0          5s

```


### 3.5 Confirm the status of DEIS from Helm and k8s

You can get the name of DEIS by using **"helm ls"** command. In this example, the name of DEIS is "prodding-olm".

```
yosshi@k8s-master-27AF23F9-0:~$ helm ls
NAME       	REVISION	UPDATED                 	STATUS  	CHART           	NAMESPACE
sad-maltese	1       	Sat May 27 14:54:07 2017	DEPLOYED	workflow-v2.14.0	deis     
```

In order to confirm the status of DEIS, you can execute **"helm status"** command.

```
yosshi@k8s-master-27AF23F9-0:~$ helm status sad-maltese
LAST DEPLOYED: Sat May 27 14:54:07 2017
NAMESPACE: deis
STATUS: DEPLOYED

RESOURCES:
==> v1/Secret
NAME                   TYPE    DATA  AGE
deis-router-dhparam    Opaque  1     4m
minio-user             Opaque  2     4m
objectstorage-keyfile  Opaque  5     4m

==> v1/ConfigMap
NAME                  DATA  AGE
slugbuilder-config    2     4m
slugrunner-config     1     4m
dockerbuilder-config  2     4m

==> v1/ServiceAccount
NAME                   SECRETS  AGE
deis-logger            1        4m
deis-monitor-telegraf  1        4m
deis-nsqd              1        4m
deis-builder           1        4m
deis-workflow-manager  1        4m
deis-controller        1        4m
deis-database          1        4m
deis-router            1        4m
deis-registry          1        4m
deis-logger-fluentd    1        4m

==> v1/Service
NAME                    CLUSTER-IP    EXTERNAL-IP   PORT(S)                                                   AGE
deis-monitor-influxui   10.0.87.175   <none>        80/TCP                                                    4m
deis-monitor-grafana    10.0.169.98   <none>        80/TCP                                                    4m
deis-builder            10.0.167.169  <none>        2222/TCP                                                  4m
deis-nsqd               10.0.67.111   <none>        4151/TCP,4150/TCP                                         4m
deis-logger-redis       10.0.114.221  <none>        6379/TCP                                                  4m
deis-router             10.0.162.135  40.74.137.24  80:30511/TCP,443:31724/TCP,2222:31665/TCP,9090:32202/TCP  4m
deis-monitor-influxapi  10.0.79.154   <none>        80/TCP                                                    4m
deis-controller         10.0.147.242  <none>        80/TCP                                                    4m
deis-database           10.0.194.94   <none>        5432/TCP                                                  4m
deis-logger             10.0.80.244   <none>        80/TCP                                                    4m
deis-registry           10.0.51.81    <none>        80/TCP                                                    4m
deis-workflow-manager   10.0.117.222  <none>        80/TCP                                                    4m

==> v1beta1/DaemonSet
NAME                   DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE-SELECTOR  AGE
deis-registry-proxy    6        6        6      0           0          <none>         4m
deis-logger-fluentd    6        6        6      0           0          <none>         4m
deis-monitor-telegraf  6        6        6      0           0          <none>         4m

==> v1beta1/Deployment
NAME                   DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
deis-controller        1        1        1           1          4m
deis-monitor-influxdb  1        1        1           1          4m
deis-logger-redis      1        1        1           1          4m
deis-logger            1        1        1           1          4m
deis-database          1        1        1           1          4m
deis-registry          1        1        1           1          4m
deis-nsqd              1        1        1           1          4m
deis-workflow-manager  1        1        1           1          4m
deis-builder           1        1        1           1          4m
deis-monitor-grafana   1        1        1           1          4m
deis-router            1        1        1           1          4m
```

DEIS and Helm is running on top of k8s, so you can of course confirm the status by using **"kubectl"** command.

```
yosshi@k8s-master-27AF23F9-0:~$ kubectl --namespace=deis get pods
NAME                                     READY     STATUS    RESTARTS   AGE
deis-builder-2791435605-jmt6x            1/1       Running   0          4m
deis-controller-2625979609-cdx7q         1/1       Running   1          4m
deis-database-1571314948-g4zh8           1/1       Running   0          4m
deis-logger-343314728-whbht              1/1       Running   1          4m
deis-logger-fluentd-cf58n                1/1       Running   0          4m
deis-logger-fluentd-p165v                1/1       Running   0          4m
deis-logger-fluentd-r950l                1/1       Running   0          4m
deis-logger-fluentd-wp18v                1/1       Running   0          4m
deis-logger-fluentd-xfqfq                1/1       Running   0          4m
deis-logger-fluentd-zgpmr                1/1       Running   0          4m
deis-logger-redis-394109792-2fmvn        1/1       Running   0          4m
deis-monitor-grafana-740719322-4c444     1/1       Running   0          4m
deis-monitor-influxdb-2881832136-gbfhd   1/1       Running   0          4m
deis-monitor-telegraf-613m4              1/1       Running   1          4m
deis-monitor-telegraf-fzrpf              1/1       Running   1          4m
deis-monitor-telegraf-mk8vg              1/1       Running   1          4m
deis-monitor-telegraf-r45bs              1/1       Running   1          4m
deis-monitor-telegraf-rkb9d              1/1       Running   1          4m
deis-monitor-telegraf-vs059              1/1       Running   1          4m
deis-nsqd-3764030276-52qpw               1/1       Running   0          4m
deis-registry-1792206801-lm0fx           1/1       Running   0          4m
deis-registry-proxy-1qv2t                1/1       Running   0          4m
deis-registry-proxy-3v08h                1/1       Running   0          4m
deis-registry-proxy-7mskb                1/1       Running   0          4m
deis-registry-proxy-bwbqg                1/1       Running   0          4m
deis-registry-proxy-h8nqn                1/1       Running   0          4m
deis-registry-proxy-h97vs                1/1       Running   0          4m
deis-router-2483473170-j1hc0             1/1       Running   0          4m
deis-workflow-manager-1893365363-5lcqm   1/1       Running   0          4m
```

