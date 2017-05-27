# First Step of DEIS Application Deploy Management

## Determine IP Address of DEIS router

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



## Register an Admin User for DEIS

The first user to register against Deis Workflow will automatically be given administrative privileges. It means that in this example, "yosshi" is administrator.


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

## Prepare the deployment of the Application

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

$ ssh yosshi@yosshi-k8smgmt.japanwest.cloudapp.azure.com -A -i ~/.ssh/yosshi-k8s-acs
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

```
yosshi@k8s-master-27AF23F9-0:~$  deis whoami
You are yosshi at http://deis.000.111.222.333.nip.io
```

## Deploy Application from Azure Container Registry

### Deploy mynginx

```
yosshi@k8s-master-27AF23F9-0:~$ deis create mynginx --no-remote
Creating Application... done, created mynginx
If you want to add a git remote for this app later, use `deis git:remote -a mynginx`
```
```
yosshi@k8s-master-27AF23F9-0:~$ deis apps:list
=== Apps
mynginx
```

```
yosshi@k8s-master-27AF23F9-0:~$ deis config:set PORT=80 -a mynginx
Creating config... done

=== mynginx Config
PORT      80
```

```
yosshi@k8s-master-27AF23F9-0:~$ deis registry:set username=yosshi password="==***+*******/**************/***" -a mynginx
Applying registry information... done

=== mynginx Registry
password     ==***+*******/**************/***
username     yosshi
```

```
yosshi@k8s-master-27AF23F9-0:~$ deis pull yosshi.azurecr.io/tyoshio2002/mynginx:1.0 -a mynginx
Creating build... done

```

```
yosshi@k8s-master-27AF23F9-0:~$ deis ps -a mynginx
=== mynginx Processes
--- cmd:
mynginx-cmd-4107848816-s47sm up (v4)
```

Pleaes access to the web site URL?  
http://mynginx.000.111.222.333.nip.io/

### Deploy HelloWorld Java Application (JAR)

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

## Scale out the Application

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

## Configure the AutoScale Application

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

## Configure the HealthCheck of Application

```
yosshi@k8s-master-27AF23F9-0:~$ deis healthchecks:set liveness httpGet 80 --type=cmd -a mynginx
Applying livenessProbe healthcheck... done

=== mynginx Healthchecks

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

## Maintenance Mode of Application

```
yosshi@k8s-master-27AF23F9-0:~$ deis maintenance:info -a helloworld
Maintenance mode is off.
```

```
yosshi@k8s-master-27AF23F9-0:~$ deis maintenance:info -a helloworld
Maintenance mode is off.
```

```
yosshi@k8s-master-27AF23F9-0:~$ deis maintenance:on -a helloworld
Enabling maintenance mode for helloworld... done
```

```
yosshi@k8s-master-27AF23F9-0:~$ deis maintenance:info -a helloworld
Maintenance mode is on.
```

```
yosshi@k8s-master-27AF23F9-0:~$ deis maintenance:off -a helloworld
Disabling maintenance mode for helloworld... done
```

```
yosshi@k8s-master-27AF23F9-0:~$ deis maintenance:info -a helloworld
Maintenance mode is off.
```

## Release Management of Application

```
yosshi@k8s-master-27AF23F9-0:~$  deis releases -a helloworld
=== helloworld Releases
v5	2017-05-27T15:34:39Z	yosshi added healthcheck info for proc type cmd
v4	2017-05-27T15:22:07Z	yosshi deployed yosshi.azurecr.io/tyoshio2002/helloworld:1.0
v3	2017-05-27T15:21:46Z	yosshi added registry info password, username
v2	2017-05-27T15:21:33Z	yosshi added PORT
v1	2017-05-27T15:21:08Z	yosshi created initial release
```

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


## Role Back the Application

```
yosshi@k8s-master-27AF23F9-0:~$  deis releases -a helloworld
=== helloworld Releases
v5	2017-05-27T15:34:39Z	yosshi added healthcheck info for proc type cmd
v4	2017-05-27T15:22:07Z	yosshi deployed yosshi.azurecr.io/tyoshio2002/helloworld:1.0
v3	2017-05-27T15:21:46Z	yosshi added registry info password, username
v2	2017-05-27T15:21:33Z	yosshi added PORT
v1	2017-05-27T15:21:08Z	yosshi created initial release
```

```
yosshi@k8s-master-27AF23F9-0:~$ deis rollback v4 -a helloworld
Rolling back to v4... done, v6
```

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

## Configure the Environment Value for Application

```
yosshi@k8s-master-27AF23F9-0:~$ deis config -a helloworld
=== helloworld Config
PORT      8080
```


```
yosshi@k8s-master-27AF23F9-0:~$ deis config:set KEY=VALUE -a helloworld
Creating config... done

=== helloworld Config
KEY       VALUE
PORT      8080
```

```
yosshi@k8s-master-27AF23F9-0:~$ deis config -a helloworld
=== helloworld Config
KEY       VALUE
PORT      8080
```