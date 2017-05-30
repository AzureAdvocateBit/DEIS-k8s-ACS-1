# First Step of DEIS Application Deploy Management

## 1. Determine IP Address of DEIS router

At first, you need to confirm the IP address of the Router.  
On Azure, Deis Workflow will automatically provision and attach a  Router. The Router is responsible for routing HTTP and HTTPS requests from the public internet to applications that are deployed and managed by Deis Workflow, as well as streaming TCP requests to the Builder.

By describing the deis-router service, you can see what IP address is allocated by Azure for your Deis Workflow cluster:

**Note:**  
In the following example, IP address was wrote as "000.111.222.333". In fact it is not actual IP address but I created a dummy IP address while writing this document. Please change your IP address instead of the above "000.111.222.333"?


```
yosshi@k8s-master-27AF23F9-0:~$ kubectl --namespace=deis describe svc deis-router
Name:			deis-router
Namespace:		deis
Labels:			heritage=deis
Selector:		app=deis-router
Type:			LoadBalancer
IP:			10.0.162.135
LoadBalancer Ingress:	000.111.222.333 <--- THIS IS IMPORTANT !!
Port:			http	80/TCP
NodePort:		http	30511/TCP
Endpoints:		10.244.6.16:8080
Port:			https	443/TCP
NodePort:		https	31724/TCP
Endpoints:		10.244.6.16:6443
Port:			builder	2222/TCP
NodePort:		builder	31665/TCP
Endpoints:		10.244.6.16:2222
Port:			healthz	9090/TCP
NodePort:		healthz	32202/TCP
Endpoints:		10.244.6.16:9090
Session Affinity:	None
Events:
  FirstSeen	LastSeen	Count	From			SubObjectPath	Type		Reason			Message
  ---------	--------	-----	----			-------------	--------	------			-------
  5m		5m		1	{service-controller }			Normal		CreatingLoadBalancer	Creating load balancer
  2m		2m		1	{service-controller }			Normal		CreatedLoadBalancer	Created load balancer
```



## 2. Register an Admin User for DEIS

At first, you need to create and register a login account. Deis Workflow will automatically be given administrative privileges for first created users. It means that in this example, "yosshi" is administrator.  

 **Note:**  
Please change your actual IP address from "000.111.222.333"? For example, if your actual IP address was "40.74.137.24", please specify following?  

http://deis.40.74.137.24.nip.io


```
yosshi@k8s-master-27AF23F9-0:~$ deis register http://deis.000.111.222.333.nip.io
username: yosshi
password: 
password (confirm): 
email: foo.bar@microsoft.com
Registered yosshi
Logged in as yosshi
Configuration file written to /home/yosshi/.deis/client.json
```

## 3. Prepare the deployment of the Application

For Dockerfile and Buildpack based application deploys via git push, Deis Workflow identifies users via SSH keys. SSH keys are pushed to the platform and must be unique to each user. Users may have multiple SSH keys as needed. Please refer to the [Users and SSH Keys](https://deis.com/docs/workflow/users/ssh-keys/)?

Deployment in this Hands On Lab, we used Docker Image from Azure Container Registry. So there is no need to create ssh keys. However you may need to deploy via Dockerfile or Buildpack in the future. So please create the following ?

```
yosshi@k8s-master-27AF23F9-0:~$ cd ~/.ssh/

yosshi@k8s-master-27AF23F9-0:~/.ssh$ ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/yosshi/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/yosshi/.ssh/id_rsa.
Your public key has been saved in /home/yosshi/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:KrH1WUeWCqIKV3p6JhLjY399xmpA7Y3RlHjr93Yspp0 yosshi@k8s-master-27AF23F9-0
The key's randomart image is:
+---[RSA 2048]----+
|        . .      |
|       . +   .   |
|    . o = . +    |
|   o o + + +     |
|+ o = o S o .    |
|.= + = = = o     |
|.++ = + + . . .  |
|..o+ o o +  .=.o |
|   .. ..+  .+Eo  |
+----[SHA256]-----+

yosshi@k8s-master-27AF23F9-0:~/.ssh$ deis keys:add ~/.ssh/id_rsa.pub 
Uploading id_rsa.pub to deis... done

yosshi@k8s-master-27AF23F9-0:~/.ssh$ deis keys:list
=== yosshi Keys
yosshi@k8s-master-27AF23F9-0 ssh-rsa AAAAB3Nz...27AF23F9-0
```

After created the SSH key, please exit at once and login to the system again.

```
yosshi@k8s-master-27AF23F9-0:~/.ssh$ exit
logout
Connection to yosshi-k8smgmt.japanwest.cloudapp.azure.com closed.

$ ssh yosshi@yosshi-k8smgmt.japanwest.cloudapp.azure.com -i ~/.ssh/yosshi-k8s-acs
Welcome to Ubuntu 16.04 LTS (GNU/Linux 4.4.0-28-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

193 packages can be updated.
105 updates are security updates.

Last login: Sat May 27 12:14:31 2017 from 61.213.65.187
```

After login to the system, please login to the DEIS again as follows?

```
yosshi@k8s-master-27AF23F9-0:~$ deis login http://deis.000.111.222.333.nip.io
username: yosshi
password: 
Logged in as yosshi
Configuration file written to /home/yosshi/.deis/client.json
```

After login to the DEIS, you can confirm the current logged in user as follows.

```
yosshi@k8s-master-27AF23F9-0:~$  deis whoami
You are yosshi at http://deis.000.111.222.333.nip.io
```

## 4. Deploy Application from Azure Container Registry

In the DEIS WorkFlow, you can choose three kind of way to deploy the application as follows.

|Deploy Type|Description|
|---|---|
|Buildpacks|Heroku buildpacks are useful if you want to follow Heroku's best practices for building applications or if you are porting an application from Heroku. [Learn how to deploy applications using Buildpacks.](https://deis.com/docs/workflow/applications/using-buildpacks/)|
|Dockerfiles|Dockerfiles are a powerful way to define a portable execution environment built on a base OS of your choosing. [Learn how to deploy applications using Dockerfiles.](https://deis.com/docs/workflow/applications/using-dockerfiles/)|
|Docker Image|Deploying a Docker image onto Deis allows you to take a Docker image from either a public or a private registry and copy it over bit-for-bit, ensuring that you are running the same image in development or in your CI pipeline as you are in production.[Learn how to deploy applications using Docker images.](https://deis.com/docs/workflow/applications/using-docker-images/)|


In this Hands on Lab, we will use the "Docker Image" type deployment. Because already you upload two Docker images to Azure Container Registry in the [Create Azure Container Registry (Private Docker Registry)](./CreateAzureContainerRegistry.md) section. If you didn't do it, please do it before following step?  


### 4.1 Deploy mynginx  

You can create the deis application by using **"deis create"** command.  

```
yosshi@k8s-master-27AF23F9-0:~$ deis create --help
Creates a new application.

- if no <id> is provided, one will be generated automatically.

Usage: deis apps:create [<id>] [options]

Arguments:
  <id>
    a uniquely identifiable name for the application. No other app can already
    exist with this name.

Options:
  --no-remote
    do not create a 'deis' git remote.
  -b --buildpack BUILDPACK
    a buildpack url to use for this app
  -r --remote REMOTE
    name of remote to create. [default: deis]
```

If you choose the deployment type as **BuildPack or Dockerfile**, you don't need to specify the **"--no-remote"** options. However if you choose the Docker Image, please specify the **"--no-remote"** options?   

I also recommend you to specify the unique ID (ex. mynginx), because if you don't specify the ID, the DEIS will automatically create a unique ID for Application. It may course the confusion of the management.   

```
yosshi@k8s-master-27AF23F9-0:~$ deis create mynginx --no-remote
Creating Application... done, created mynginx
If you want to add a git remote for this app later, use `deis git:remote -a mynginx`
```

After created the Application, please confirm the list of Applicaitons? Now you can see only one deployment.

```
yosshi@k8s-master-27AF23F9-0:~$ deis apps:list
=== Apps
mynginx
```

After created the Application, please configure the PORT number for the Application (It is the port number which you specify the EXPOSE on Dockerfile)?

```
yosshi@k8s-master-27AF23F9-0:~$ deis config:set PORT=80 -a mynginx
Creating config... done

=== mynginx Config
PORT      80
```

This step is only needed for "Docker Image" deployment type. In the [Private Registry](https://deis.com/docs/workflow/applications/using-docker-images/) document in DEIS, following was explained.

```
When using a private registry the docker images 
are no longer pulled into the Deis Internal Reg
istry via the Deis Workflow Controller but rath
er is managed by Kubernetes. This will increase
security and overall speed, however the applica
tion port information can no longer be discover
ed. Instead the application port information ca
n be set via deis config:set PORT=80 prior to s
etting the registry information.
```

After that, in order to access to the Azure Container Registry (Private Registry), please set the username and password for your application?

```
yosshi@k8s-master-27AF23F9-0:~$ deis registry:set username=yosshi password="==***+*******/**************/***" -a mynginx
Applying registry information... done

=== mynginx Registry
password     ==***+*******/**************/***
username     yosshi
```

Finally, you can pull the image fron the Private Docker Registry as follows.

```
yosshi@k8s-master-27AF23F9-0:~$ deis pull yosshi.azurecr.io/tyoshio2002/mynginx:1.0 -a mynginx
Creating build... done

```

Please confirm whether your application is running or not?

```
yosshi@k8s-master-27AF23F9-0:~$ deis ps -a mynginx
=== mynginx Processes
--- cmd:
mynginx-cmd-4107848816-s47sm up (v4)
```

After deployed the application, you can access to your application with  
**http://[your application id].[IP address].nip.io**.

For example, you can access to the application like follows.  
http://mynginx.000.111.222.333.nip.io/

### 4.2 Deploy HelloWorld Java Application (JAR)

All the deployment process is same as nginx except fot the PORT number configuration. Any kind of the applcation will be able to deploy the same procedure.

Please refer to the following?

```
yosshi@k8s-master-27AF23F9-0:~$ deis create helloworld --no-remote
Creating Application... done, created helloworld
If you want to add a git remote for this app later, use `deis git:remote -a helloworld`
```

```
yosshi@k8s-master-27AF23F9-0:~$ deis apps:list
=== Apps
helloworld
mynginx
```

```
yosshi@k8s-master-27AF23F9-0:~$ deis config:set PORT=8080 -a helloworld
Creating config... done

=== helloworld Config
PORT      8080
```

```
yosshi@k8s-master-27AF23F9-0:~$ deis registry:set username=yosshi password="==***+*******/**************/***" -a helloworld
Applying registry information... done

=== helloworld Registry
password     ==***+*******/**************/***
username     yosshi
```

```
yosshi@k8s-master-27AF23F9-0:~$ deis pull yosshi.azurecr.io/tyoshio2002/helloworld:1.0 -a helloworld
Creating build... done
```

```
yosshi@k8s-master-27AF23F9-0:~$ deis ps -a helloworld
=== helloworld Processes
--- cmd:
helloworld-cmd-3351959756-fnklr up (v4)
```

## 5. Scale out the Application

In order to scale out the Application, you can execute **"deis scale"** command.

```
yosshi@k8s-master-27AF23F9-0:~$ deis scale cmd=4 -a mynginx
Scaling processes... but first, coffee!
done in 21s
=== mynginx Processes
--- cmd:
mynginx-cmd-4107848816-5g7mm up (v4)
mynginx-cmd-4107848816-s47sm up (v4)
mynginx-cmd-4107848816-tb5kb up (v4)
mynginx-cmd-4107848816-z7grz up (v4)
```

```
yosshi@k8s-master-27AF23F9-0:~$ deis scale cmd=4 -a helloworld
Scaling processes... but first, coffee!
done in 46s
=== helloworld Processes
--- cmd:
helloworld-cmd-3351959756-8l37s up (v4)
helloworld-cmd-3351959756-fnklr up (v4)
helloworld-cmd-3351959756-gcc8s up (v4)
helloworld-cmd-3351959756-zkqg5 up (v4)
```

**Note:**  
You can specify the command argument as "cmd=number" or "web=number". If you deploy the appliaction by using BuildPack, please specify the "web". And if you deploy the application, by using Dockerfile or Docker Image, please specify the "cmd"?

For more detail, please refer to the following original explantion?

|[web vs cmd Process Types](https://deis.com/docs/workflow/applications/managing-app-processes/)|
|---|
|When deploying to Deis Workflow using a **Heroku Buildpack, Workflow boots the web process type** to boot the application server. When you deploy an application that has a **Dockerfile or uses Docker images, Workflow boots the cmd process type**. Both act similarly in that they are exposed to the router as web applications. However, the cmd process type is special because, if left undefined, it is equivalent to running the container without any additional arguments. (i.e. The process specified by the Dockerfile or Docker image's CMD directive will be used.) **If migrating an application from Heroku Buildpacks to a Docker-based deployment, Workflow will not automatically convert the web process type to cmd**. To do this, you'll have to manually scale down the old process type and scale the new process type up.|



## 6. Configure the AutoScale Application

Autoscale allows adding a minimum and maximum number of pods on a per process type basis. This is accomplished by specifying a target CPU usage across all available pods.

```
yosshi@k8s-master-27AF23F9-0:~$ deis autoscale:set cmd --min=3 --max=8 --cpu-percent=75 -a helloworld
Applying autoscale settings for process type cmd on helloworld... done

yosshi@k8s-master-27AF23F9-0:~$ deis autoscale:list -a helloworld
=== helloworld Autoscale

--- cmd:
Min Replicas: 3
Max Replicas: 8
CPU: 75%
```

## 7. Configure the HealthCheck of Application

By default, Workflow only checks that the application starts in their Container. If it is preferred to have Kubernetes respond to application health, a health check may be added by configuring a health check probe for the application.


```
yosshi@k8s-master-27AF23F9-0:~$ deis healthchecks:set liveness httpGet 80 --type=cmd -a helloworld
Applying livenessProbe healthcheck... done

=== helloworld Healthchecks

cmd:
--- Liveness
Initial Delay (seconds): 50
Timeout (seconds): 50
Period (seconds): 10
Success Threshold: 1
Failure Threshold: 3
Exec Probe: N/A
HTTP GET Probe: Path="/" Port=80 HTTPHeaders=[]
TCP Socket Probe: N/A

--- Readiness
No readiness probe configured.
```

The health checks are implemented as [Kubernetes container probes](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes). A liveness and a readiness probe can be configured, and each probe can be of type httpGet, exec, or tcpSocket depending on the type of probe the container requires.

A liveness probe is useful for applications running for long periods of time, eventually transitioning to broken states and cannot recover except by restarting them.

For more detail, please refere to this [document](https://deis.com/docs/workflow/applications/managing-app-configuration/)?



## 8. Maintenance Mode of Application

Maintenance mode for applications is useful to perform certain migrations or upgrades during which we don't want to serve client requests. Deis Workflow supports maintenance mode for an app during which the access to the app is blocked.

### 8.1 Confirm Current mode
You can confirm the current maintenance mode as follows.

```
yosshi@k8s-master-27AF23F9-0:~$ deis maintenance:info -a helloworld
Maintenance mode is off.
```

**Off** : means accessable.  
**On**  : means not accessable.  

### 8.2 Maintenance mode ON

If you set the maintenance mode to on, any user can't access to the service.

```
yosshi@k8s-master-27AF23F9-0:~$ deis maintenance:on -a helloworld
Enabling maintenance mode for helloworld... done
```

```
yosshi@k8s-master-27AF23F9-0:~$ deis maintenance:info -a helloworld
Maintenance mode is on.
```

### 8.3 Maintenance mode OFF

If you set the maintenance mode to off, any user can access to the service.

```
yosshi@k8s-master-27AF23F9-0:~$ deis maintenance:off -a helloworld
Disabling maintenance mode for helloworld... done
```

```
yosshi@k8s-master-27AF23F9-0:~$ deis maintenance:info -a helloworld
Maintenance mode is off.
```

## 9. Release Management of Application

Deis Workflow tracks all changes to your application. Each time a build or config change is made to your application a new release is created. These release numbers increase monotonically. You can see a record of changes to your application using **deis releases**.

```
yosshi@k8s-master-27AF23F9-0:~$ deis releases -a helloworld
=== helloworld Releases
v5	2017-05-27T15:34:39Z	yosshi added healthcheck info for proc type cmd
v4	2017-05-27T15:22:07Z	yosshi deployed yosshi.azurecr.io/tyoshio2002/helloworld:1.0
v3	2017-05-27T15:21:46Z	yosshi added registry info password, username
v2	2017-05-27T15:21:33Z	yosshi added PORT
v1	2017-05-27T15:21:08Z	yosshi created initial release
```

In order to get more detail of specific version like v2, you can get the information by using **deis releases:info** command.

```
yosshi@k8s-master-27AF23F9-0:~$ deis releases:info -a helloworld v2
=== helloworld Release v2
config:   ce32ed2c-bb23-4aff-b681-157d5c8b7534
owner:    yosshi
created:  2017-05-27T15:21:33Z
summary:  yosshi added PORT
updated:  2017-05-27T15:21:33Z
uuid:     f8a9ea6b-59a4-42e4-973e-67c0792a3762
```


## 10. Role Back the Application

Deis Workflow also supports rolling back go previous releases. If  your new version of the application failed somethings, you can  rollback to a previous version.

### 10.1 Current Application Release Info  
In this application, there is five version until now.  

```
yosshi@k8s-master-27AF23F9-0:~$  deis releases -a helloworld
=== helloworld Releases
v5	2017-05-27T15:34:39Z	yosshi added healthcheck info for proc type cmd
v4	2017-05-27T15:22:07Z	yosshi deployed yosshi.azurecr.io/tyoshio2002/helloworld:1.0
v3	2017-05-27T15:21:46Z	yosshi added registry info password, username
v2	2017-05-27T15:21:33Z	yosshi added PORT
v1	2017-05-27T15:21:08Z	yosshi created initial release
```

### 10.2 Role Back the Application to v4  

In this example, rollback to v4.
  
```
yosshi@k8s-master-27AF23F9-0:~$ deis rollback v4 -a helloworld
Rolling back to v4... done, v6
```

After roll backed the application, please confirm whether it was rolled back or not?

```
yosshi@k8s-master-27AF23F9-0:~$ deis releases -a helloworld
=== helloworld Releases
v6	2017-05-27T15:41:36Z	yosshi rolled back to v4
v5	2017-05-27T15:34:39Z	yosshi added healthcheck info for proc type cmd
v4	2017-05-27T15:22:07Z	yosshi deployed yosshi.azurecr.io/tyoshio2002/helloworld:1.0
v3	2017-05-27T15:21:46Z	yosshi added registry info password, username
v2	2017-05-27T15:21:33Z	yosshi added PORT
v1	2017-05-27T15:21:08Z	yosshi created initial release
```

## 11. Configure the Environment Value for Application

Deis treats backing services as attached resources (like databases, messaging, caches and queues). Attachments are performed using environment variables.  
Please refer to [IV. Backing Service of 12 Factor App](https://12factor.net/backing-services)

### 11.1 Current Configuration

If you deployed application from Docker image, already you configured the PORT number like follows.

```
yosshi@k8s-master-27AF23F9-0:~$ deis config -a helloworld
=== helloworld Config
PORT      8080
```

### 11.2 Configure the new Environment Value

In order to configure the Environment Value for the Applicaiton, you can execute **deis config:set** command like follows.

```
yosshi@k8s-master-27AF23F9-0:~$ deis config:set KEY=VALUE -a helloworld
Creating config... done

=== helloworld Config
KEY       VALUE
PORT      8080
```

### 11.3 DB Attach configuration
For example, use deis config to set a DAHOST, DBNAME, JDBCUSER, JDBCPASSWORD that attaches the application to an external MySQL database.  

```
yosshi@k8s-master-27AF23F9-0:~$ deis config -a helloworld
=== helloworld Config
DB_HOST            looping-angelfish-mysql.default.svc.cluster.local
DB_NAME            artist
JDBC_PASSWORD      **********
JDBC_USER          root
```

## 12. Restrict the Access (Whitelist) to the services


### 12.1 List the current availabel IP address

If you execute the **"deis whitelist:list"** command, you can confirm the current configuration as follows.

```
yosshi@k8s-master-27AF23F9-0:~$ deis whitelist:list -a helloworld
=== helloworld Whitelisted Addresses
```

### 12.2 Add whitelist IP address to the service

If you execute the **"deis whitelist:add"** command, you can add the whitelist IP address.

```
yosshi@k8s-master-27AF23F9-0:~$ deis whitelist:add 10.0.1.0/24,121.212.121.212 -a helloworld
Adding 10.0.1.0/24,121.212.121.212 to helloworld whitelist...
done
yosshi@k8s-master-27AF23F9-0:~$ deis whitelist:list  -a helloworld
=== helloworld Whitelisted Addresses
10.0.1.0/24
121.212.121.212
```

### 12.3 Remove whitelist IP address from the service

If you execute the **"deis whitelist:remove"** command, you can remove the whitelist IP address from the service.

```
yosshi@k8s-master-27AF23F9-0:~$ deis whitelist:remove 121.212.121.212  -a helloworld
Removing 121.212.121.212 from helloworld whitelist...
done
yosshi@k8s-master-27AF23F9-0:~$ deis whitelist:list  -a helloworld
=== helloworld Whitelisted Addresses
10.0.1.0/24

```

## 13. Redundant Router instance

As default, deis route(nginx instance) is running on only one node. So if the node failed or down, all of service become unavailable.

### 13.1 Confirm the node which is running the router.

You can confirm the current node by using **"kubectl describe pod"** command.

```
yosshi@k8s-master-27AF23F9-0:~$ kubectl --namespace=deis describe pod deis-router |grep Node
Node:		k8s-agent-27af23f9-2/10.240.0.6
```

### 13.2 Increase the number of node which is running the router.

In production environment, you need to configure the redundant structure for the router instance. In order to increase the number of route, please execute **"kubectl scale"** command? 

```
yosshi@k8s-master-27AF23F9-0:~$ kubectl --namespace=deis scale --replicas=2 deployment/deis-router
deployment "deis-router" scaled

yosshi@k8s-master-27AF23F9-0:~$ kubectl --namespace=deis describe pod deis-router |grep Node
Node:		k8s-agent-27af23f9-2/10.240.0.6
Node:		k8s-agent-27af23f9-1/10.240.0.4
```


## 14. Segregation the Runtime Environment (Change the runtime node) 

In some situation, you would like to segregate the runtime environment like staging, production. If you use the k8s with deis, you can segregate the runtime environment very easily.

### 14.1 Confirm the number of nodes on ACS

At first, please confirm the number of the nodes on your Azure Container Services as follows?

```
yosshi@k8s-master-27AF23F9-0:~$ kubectl get nodes
NAME                    STATUS                     AGE
k8s-agent-27af23f9-0    Ready                      2d
k8s-agent-27af23f9-1    Ready                      2d
k8s-agent-27af23f9-2    Ready                      2d
k8s-agent-27af23f9-3    Ready                      2d
k8s-agent-27af23f9-4    Ready                      2d
k8s-agent-27af23f9-5    NotReady                   28s
k8s-master-27af23f9-0   Ready,SchedulingDisabled   2d
```

### 14.2 Confirm which node the service is running

Could you confirm which of the nodes your application is running? 

```
yosshi@k8s-master-27AF23F9-0:~$ kubectl describe pods --namespace helloworld |grep Node
Node:		k8s-agent-27af23f9-1/10.240.0.4
Node:		k8s-agent-27af23f9-2/10.240.0.6
Node:		k8s-agent-27af23f9-4/10.240.0.7
```

In this example the applicaiton is running on 3 nodes as   

- "k8s-agent-27af23f9-1"  
- "k8s-agent-27af23f9-2"  
- "k8s-agent-27af23f9-4"  


### 14.3 Grouping the node to segregate

In order to segregate the runtime environment, please create the group by using **"kubectl label nodes"** command? 
 
In following example, we create two group as node-group1 and node-group2.

**Group : node-group1**

```
yosshi@k8s-master-27AF23F9-0:~$ kubectl label nodes k8s-agent-27af23f9-0 environment=node-group1
node "k8s-agent-27af23f9-0" labeled
yosshi@k8s-master-27AF23F9-0:~$ kubectl label nodes k8s-agent-27af23f9-1 environment=node-group1
node "k8s-agent-27af23f9-1" labeled
yosshi@k8s-master-27AF23F9-0:~$ kubectl label nodes k8s-agent-27af23f9-2 environment=node-group1
node "k8s-agent-27af23f9-2" labeled
```

- "k8s-agent-27af23f9-0"  
- "k8s-agent-27af23f9-1"  
- "k8s-agent-27af23f9-2"  


**Group : node-group2**

```
yosshi@k8s-master-27AF23F9-0:~$ kubectl label nodes k8s-agent-27af23f9-3 environment=node-group2
node "k8s-agent-27af23f9-3" labeled
yosshi@k8s-master-27AF23F9-0:~$ kubectl label nodes k8s-agent-27af23f9-4 environment=node-group2
node "k8s-agent-27af23f9-4" labeled
yosshi@k8s-master-27AF23F9-0:~$ kubectl label nodes k8s-agent-27af23f9-5 environment=node-group2
node "k8s-agent-27af23f9-5" labeled
```

- "k8s-agent-27af23f9-3"  
- "k8s-agent-27af23f9-4"  
- "k8s-agent-27af23f9-5"  


### 14.4 Move the instances to node-group1

After created the group, you can move the instances to the specific group as follows. Please execute the **"deis tags:set"** command?

```
yosshi@k8s-master-27AF23F9-0:~$ deis tags:set environment=node-group1 -a helloworld
Applying tags... done

=== helloworld Tags
environment     node-group1

yosshi@k8s-master-27AF23F9-0:~$ kubectl describe pods --namespace helloworld |grep Node
Node:		k8s-agent-27af23f9-0/10.240.0.5
Node:		k8s-agent-27af23f9-2/10.240.0.6
Node:		k8s-agent-27af23f9-1/10.240.0.4
```

Of course, You can also move the instances to "node-group2" as follows.

```
yosshi@k8s-master-27AF23F9-0:~$ deis tags:set environment=node-group2 -a helloworld
Applying tags... done

=== helloworld Tags
environment     node-group2

yosshi@k8s-master-27AF23F9-0:~$ kubectl describe pods --namespace helloworld |grep Node
Node:		k8s-agent-27af23f9-5/10.240.0.9
Node:		k8s-agent-27af23f9-3/10.240.0.8
Node:		k8s-agent-27af23f9-4/10.240.0.7
```

## 15. Restart specific an process

If you would like to stop the specific process, you can do it by using **"deis ps:restart"** command.

### 15.1 List application processes

At first, please confirm how many process is running for the applicaiton?

```
yosshi@k8s-master-27AF23F9-0:~$ deis ps:list -a helloworld
=== helloworld Processes
--- cmd:
helloworld-cmd-720068219-gtzw5 up (v9)
helloworld-cmd-720068219-hlmhr up (v9)
helloworld-cmd-720068219-xqmrs up (v9)
```

### 15.2 Restart the appilcation process

For example, if you would like to restart the above process of "helloworld-cmd-720068219-xqmrs", please execute following?

```
yosshi@k8s-master-27AF23F9-0:~$ deis ps:restart helloworld-cmd-720068219-xqmrs -a helloworld
Restarting processes... but first, coffee!
done in 29s
=== helloworld Processes
--- cmd:
helloworld-cmd-720068219-ppch5 up (v9)
```
After executed the above, you can confirm the process name had changed like follows.

```
yosshi@k8s-master-27AF23F9-0:~$ deis ps:list -a helloworld
=== helloworld Processes
--- cmd:
helloworld-cmd-720068219-gtzw5 up (v9)
helloworld-cmd-720068219-hlmhr up (v9)
helloworld-cmd-720068219-ppch5 up (v9)
```