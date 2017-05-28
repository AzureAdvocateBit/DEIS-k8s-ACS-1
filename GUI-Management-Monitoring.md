# GUI Management and Monitoring

## Kubernetes Web UI (Dash board)

On master node, please execute following?

```
yosshi@k8s-master-27AF23F9-0:~$ kubectl proxy &
[1] 55658
```

After executed the above, please execute following on your local machine?

```
$ ssh -fNL 8001:localhost:8001 yosshi@yosshi-k8smgmt.japanwest.cloudapp.azure.com -i ~/.ssh/yosshi-k8s-acs
```

After executed both of the above, please open your browser and access to the following URL?

http://localhost:8001/ui

Then you can see following screen on your browser.

![Kubernetes Dash Board](https://c1.staticflickr.com/5/4267/34946140565_3923344b49_z.jpg)

## Kubernetes Operational View

After executed the above, please install the "Kubernetes Operational View"? It is very cool GUI to understand the behavior of individual images and nodes on Azure.

At first, please execute the **helm search** command? If you installed successfully, you can get the result as follows.

```
yosshi@k8s-master-27AF23F9-0:~$ helm search|grep ops-view
stable/kube-ops-view         	0.3.0  	Kubernetes Operational View - read-only system ...
```

If you installed successfully, please install the kube-ops-view ?

```
yosshi@k8s-master-27AF23F9-0:~$ helm install stable/kube-ops-view --namespace common
NAME:   queenly-ibex
LAST DEPLOYED: Sun May 28 14:52:02 2017
NAMESPACE: common
STATUS: DEPLOYED

RESOURCES:
==> v1/Service
NAME                        CLUSTER-IP   EXTERNAL-IP  PORT(S)  AGE
queenly-ibex-kube-ops-view  10.0.228.17  <none>       80/TCP   0s

==> v1beta1/Deployment
NAME                        DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
queenly-ibex-kube-ops-view  1        0        0           0          0s


NOTES:
To access the Kubernetes Operational View UI:

1. First start the kubectl proxy:

   kubectl proxy

2. Now open the following URL in your browser:

   http://localhost:8001/api/v1/proxy/namespaces/common/services/queenly-ibex-kube-ops-view/

Please try reloading the page if you see "ServiceUnavailable / no endpoints available for service", pod creation might take a moment.
```

After executed above the command, please open your browser and access to the following URL?

http://localhost:8001/api/v1/proxy/namespaces/common/services/queenly-ibex-kube-ops-view/

Then you can see following screen on your browser.  

![](https://c1.staticflickr.com/5/4275/34814452211_1d9ae299ca_z.jpg)

If you installed or scale out the application, you can see the following screen.  

![](https://c1.staticflickr.com/5/4271/34814452111_0b3463765a_z.jpg)

It's so cool!!

## Grafana
If you installed DEIS succesfully, already you can access to the Grafana too.

```
User : admin
Password : admin
```

please open your browser and access to the following URL?

http://grafana.000.111.222.333.nip.io

![](https://c1.staticflickr.com/5/4221/34906226886_0be9312ff4_z.jpg)

After login to the Grafana, you can see the usage status on graph like follows.

![](https://c1.staticflickr.com/5/4276/34103643914_25f1499939_z.jpg)


## Deis Dash 

Deis Dash is a web based UI for the DEIS and provided as OSS.

https://github.com/olalonde/deisdash

[In the previous entry](https://github.com/yoshioterada/DEIS-k8s-ACS/blob/master/FirstStepOfDEIS.md), you can see so many deis command. If you install this GUI tools, you can execute some command on GUI screen.

### Get Deis Dash

```
yosshi@k8s-master-27AF23F9-0:~$ mkdir deisdash

yosshi@k8s-master-27AF23F9-0:~$ cd deisdash/

yosshi@k8s-master-27AF23F9-0:~/deisdash$ git clone https://github.com/olalonde/deisdash.git
Cloning into 'deisdash'...
remote: Counting objects: 218, done.
remote: Total 218 (delta 0), reused 0 (delta 0), pack-reused 218
Receiving objects: 100% (218/218), 1009.89 KiB | 0 bytes/s, done.
Resolving deltas: 100% (74/74), done.
Checking connectivity... done.

yosshi@k8s-master-27AF23F9-0:~/deisdash$ ls
deisdash

yosshi@k8s-master-27AF23F9-0:~/deisdash$ cd deisdash/

yosshi@k8s-master-27AF23F9-0:~/deisdash/deisdash$ ls
actions     containers  index.html  LICENSE     package.json  reducers   server.js  store   utils                    webpack.config.js
components  electron    index.js    middleware  README.md     routes.js  static     styles  webpack.build.config.js  webpack.electron.config.js
```

### Create Deis Dash Application on DEIS

```
yosshi@k8s-master-27AF23F9-0:~/deisdash/deisdash$ deis create dash
Creating Application... done, created dash
Git remote deis successfully created for app dash.
yosshi@k8s-master-27AF23F9-0:~/deisdash/deisdash$ deis config:set NPM_CONFIG_PRODUCTION=false
Creating config... done

=== dash Config
NPM_CONFIG_PRODUCTION      false
```

### Build & Run Deis Dash Application

```
yosshi@k8s-master-27AF23F9-0:~/deisdash/deisdash$ git push deis The authenticity of host '[deis-builder.000.111.222.333.nip.io]:2222 ([000.111.222.333]:2222)' can't be established.
ECDSA key fingerprint is SHA256:G228itP0HklCmU+/qFUAPHmnHUzOpv7njTuGAoo1Vzs.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[deis-builder.000.111.222.333.nip.io]:2222,[000.111.222.333]:2222' (ECDSA) to the list of known hosts.
Counting objects: 217, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (135/135), done.
Writing objects: 100% (217/217), 1009.75 KiB | 0 bytes/s, done.
Total 217 (delta 74), reused 217 (delta 74)
remote: Resolving deltas: 100% (74/74), done.
Starting build... but first, coffee!
...
...
...
...
...
...
...
...
...
...
...
...
-----> Restoring cache...
       No cache file found. If this is the first deploy, it will be created now.
-----> Node.js app detected
       
-----> Creating runtime environment
       
       NPM_CONFIG_LOGLEVEL=error
       NPM_CONFIG_PRODUCTION=false
       NODE_VERBOSE=false
       NODE_ENV=production
       NODE_MODULES_CACHE=true
       
-----> Installing binaries
       engines.node (package.json):  5.5.0
       engines.npm (package.json):   3.3.12
       
       Downloading and installing node 5.5.0...
       npm 3.3.12 already installed with node
       
-----> Restoring cache
       Skipping cache restore (new runtime signature)
       
-----> Building dependencies
       Installing node modules (package.json)
       
       > node-sass@3.13.1 install /tmp/build/node_modules/node-sass
       > node scripts/install.js
       
       Downloading binary from https://github.com/sass/node-sass/releases/download/v3.13.1/linux-x64-47_binding.node
……
……
……

-----> Discovering process types
       Default process types for Node.js -> web
-----> Checking for changes inside the cache directory...
       Files inside cache folder changed, uploading new cache...
       Done: Uploaded cache (26M)
-----> Compiled slug size is 40M
Build complete.
Launching App...
...
...
...
...
...
...
Done, dash:v4 deployed to Workflow

Use 'deis open' to view this application in your browser

To learn more, use 'deis help' or visit https://deis.com/

To ssh://git@deis-builder.000.111.222.333.nip.io:2222/dash.git
 * [new branch]      master -> master
```

### Deis GUI dash board Management

After installed the dash application, please access to the following URL?

http://dash.000.111.222.333.nip.io

If you acces to the above URL, you can see following screen. In this screen, please login with the account which created the **deis register** in the previous entry.

![](https://c1.staticflickr.com/5/4267/34947352065_79f4fec6a2_z.jpg)

After login the dash, you can see the following screen. In this screen, you can confirm all of applications which you deployed.

![](https://c1.staticflickr.com/5/4195/34136824063_19177d3e8c_z.jpg)

If you push the link of specific application, you can see the following screen. In this screen, you can scale out with the Scale bar.

![](https://c1.staticflickr.com/5/4227/34560152070_53709aaf29_z.jpg)

If you push the Configuration tab, you can see the following screen. In this screen, you can confirm exsiting Environment Value. And also you can add additional Environment Value.

![](https://c1.staticflickr.com/5/4225/34136823563_29c6b8b655_z.jpg)

If you push the link of the Releasese, then you can see the following screen.

![](https://c1.staticflickr.com/5/4243/34560151580_d272e02dd2_z.jpg)

If you push the link of Logs, you can see the Applicatoin log on the screen.

![](https://c1.staticflickr.com/5/4196/34560151120_48f10cd47a_z.jpg)

## Prometheus Log Monitoring

[Prometheus](https://prometheus.io/) is open source systems monitoring and alerting tool.

Prometheus's main features are:  

- a multi-dimensional [data model](https://prometheus.io/docs/concepts/data_model/) (time series identified by metric name and key/value pairs)
- a [flexible query language](https://prometheus.io/docs/querying/basics/) to leverage this dimensionality
- no reliance on distributed storage; single server nodes are autonomous
- time series collection happens via a pull model over HTTP
- [pushing time series](https://prometheus.io/docs/instrumenting/pushing/) is supported via an intermediary gateway
- targets are discovered via service discovery or static configuration
- multiple modes of graphing and dashboarding support

To install the Prometheus, please execute the following command?

```
yosshi@k8s-master-27AF23F9-0:~$ helm install stable/prometheus --namespace common
NAME:   kind-crocodile
LAST DEPLOYED: Sun May 28 16:09:45 2017
NAMESPACE: common
STATUS: DEPLOYED

RESOURCES:
==> v1/ConfigMap
NAME                                    DATA  AGE
kind-crocodile-prometheus-server        3     2s
kind-crocodile-prometheus-alertmanager  1     2s

==> v1/PersistentVolumeClaim
NAME                                    STATUS   VOLUME  CAPACITY  ACCESSMODES  STORAGECLASS  AGE
kind-crocodile-prometheus-server        Pending  1s
kind-crocodile-prometheus-alertmanager  Pending  1s

==> v1/Service
NAME                                          CLUSTER-IP   EXTERNAL-IP  PORT(S)   AGE
kind-crocodile-prometheus-server              10.0.251.24  <none>       80/TCP    1s
kind-crocodile-prometheus-kube-state-metrics  None         <none>       80/TCP    1s
kind-crocodile-prometheus-alertmanager        10.0.22.54   <none>       80/TCP    1s
kind-crocodile-prometheus-node-exporter       None         <none>       9100/TCP  1s

==> v1beta1/DaemonSet
NAME                                     DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE-SELECTOR  AGE
kind-crocodile-prometheus-node-exporter  6        6        0      0           0          <none>         1s

==> v1beta1/Deployment
NAME                                          DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
kind-crocodile-prometheus-alertmanager        1        1        1           0          1s
kind-crocodile-prometheus-kube-state-metrics  1        1        1           0          1s
kind-crocodile-prometheus-server              1        1        1           0          1s


NOTES:
The Prometheus server can be accessed via port 80 on the following DNS name from within your cluster:
kind-crocodile-prometheus-server.common.svc.cluster.local


Get the Prometheus server URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace common -l "app=prometheus,component=server" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace common port-forward $POD_NAME 9090


The Prometheus alertmanager can be accessed via port 80 on the following DNS name from within your cluster:
kind-crocodile-prometheus-alertmanager.common.svc.cluster.local


Get the Alertmanager URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace common -l "app=prometheus,component=alertmanager" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace common port-forward $POD_NAME 9093

For more information on running Prometheus, visit:
https://prometheus.io/
```

After installed the Prometheus, please execute following on **Master node**?

```
yosshi@k8s-master-27AF23F9-0:~$ export POD_NAME=$(kubectl get pods --namespace common -l "app=prometheus,component=server" -o jsonpath="{.items[0].metadata.name}")
yosshi@k8s-master-27AF23F9-0:~$  kubectl --namespace common port-forward $POD_NAME 9090
Forwarding from 127.0.0.1:9090 -> 9090
Forwarding from [::1]:9090 -> 9090
```

After executed the above, please executed following on **Local Machine**?

```
$ ssh -fNL 9090:localhost:9090 yosshi@yosshi-k8smgmt.japanwest.cloudapp.azure.com -i ~/.ssh/yosshi-k8s-acs
```

### Prometheus GUI Monitoring

After executed both of the above, please access to the following URL?

http://localhost:9090

Then you can see the following screen.

![](https://c1.staticflickr.com/5/4243/34784467762_8b9e07bfac_z.jpg)

If you select the "-insert metric at cursor-", then you noticed that you already can get so many metrics as default.

![](https://c1.staticflickr.com/5/4204/34784468062_7cfe1d5d33_z.jpg)

If you select your favorite metrics, please push the button of "Execute"? Then you can see the raw data on the screen.

![](https://c1.staticflickr.com/5/4268/34784468862_40893f167b_z.jpg)

After you confirm the raw data, please push the "Graph" tab? Then you can see it as Graph on the screen like follows.

![](https://c1.staticflickr.com/5/4273/34560922890_943e71bd34_z.jpg)

