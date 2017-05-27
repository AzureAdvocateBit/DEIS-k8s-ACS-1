# Create Kubernetes on Azure Container Services (ACS) 

## Preparation to access to the ACS

In order to create the ACS on Azure, you need to get Principan information by using Azure CLI v2 (az command) on your local environment. 

## Install Azure CLI 2.0 

In order to install the Azure CLI 2.0, please refer to the 
"[Install Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)" documents?

If you are using Linux or Mac, simply please execute following command to install the cli on your machine?

```
$ curl -L https://aka.ms/InstallAzureCli | bash
```

**Note for Linux or Mac Users :**  
If you are using "Anaconda" on your system, you may fail to install the Azure CLI 2.0. If you uninstall it, you may be able to install the Azure CLI 2.0.

**Note for Mac Users :**  
If you execute the above "curl" and faced "openssl file not found" error, you may need to execute "xcode-select --install" command.

```
$ curl -L https://aka.ms/InstallAzureCli | bash
...

    cc -fno-strict-aliasing -fno-common -dynamic -arch x86_64 -arch i386 -g -Os -pipe -fno-common -fno-strict-aliasing -fwrapv -DENABLE_DTRACE -DMACOSX -DNDEBUG -Wall -Wstrict-prototypes -Wshorten-64-to-32 -DNDEBUG -g -fwrapv -Os -Wall -Wstrict-prototypes -DENABLE_DTRACE -arch x86_64 -arch i386 -pipe -I/System/Library/Frameworks/Python.framework/Versions/2.7/include/python2.7 -c build/temp.macosx-10.12-intel-2.7/_openssl.c -o build/temp.macosx-10.12-intel-2.7/build/temp.macosx-10.12-intel-2.7/_openssl.o
    build/temp.macosx-10.12-intel-2.7/_openssl.c:434:10: fatal error: 'openssl/opensslv.h' file not found
    #include <openssl/opensslv.h>
             ^
    1 error generated.
    error: command 'cc' failed with exit status 1
...    
    
$ xcode-select --install
```

## Login Azure from Azure CLI 2.0 

In order to login to the Azure via the CLI, please execute **"az login"** command ? Then you see the following message.  

```
$ az login
To sign in, use a web browser to open the page 
https://aka.ms/devicelogin 
and enter the code GG9XHVJRB to authenticate.
```

In order to login, please open the browser and access to the above URL? Then you can see the following screen.  

![Login 1](https://c1.staticflickr.com/5/4226/34109197003_fa5b032931_z.jpg)

In this screen, please input the ont time password which showed on your command.(In this time GG9XHVJRB) After input the password, please push the "Continue" button?   

![Login 2](https://c1.staticflickr.com/5/4221/34532666820_0c9588fc18_z.jpg)

Then if you already login to the Azure Portal, you can see the following screen. If you didn't login to the Azure Portal, please login?  Please push the blue button?  

![Login 3](https://c1.staticflickr.com/5/4222/34788073201_b4eefb9ec9_z.jpg
)

Then your login process finished and can see the following screen.  

![Login 4](https://c1.staticflickr.com/5/4225/34532666890_1659112a99_z.jpg)

After finished the above, please go back to the command prompt? Then you can see the following messages.  


```
$ az login
To sign in, use a web browser to open the page https://aka.ms/devicelogin and enter the code GG9XHVJRB to authenticate.
[
  {
    "cloudName": "AzureCloud",
    "id": "acc75193-****-****-****-401e1a2bde7e",
    "isDefault": true,
    "name": "Visual Studio Enterprise with MSDN",
    "state": "Enabled",
    "tenantId": "387983da-****-****-****-61869f94518d",
    "user": {
      "name": "****@**********",
      "type": "user"
    }
  }
]
```

In this result, only one subscription was showed. However you can use many subscription for one user. So please set for the account the specific subscription as follows.  

```
$ az account set --subscription “acc75193-****-****-****-401e1a2bde7e"
```

After set the subscription, please create **"Service Principal"** and **"Client Secret"** for K8S? This information will be used during the creation of Azure Container Services on Azure Portal Screen. Please execut **az ad sp create-for-rbac** command?  

```
$ az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/acc75193-eb5a-48e1-b9a7-401e1a2bde7e"
{
  "appId": "09aa4718-****-****-****-83cf60696d14", <--- Service Principal
  "displayName": "azure-cli-2017-05-27-08-41-39",
  "name": "http://azure-cli-2017-05-27-08-41-39",
  "password": "2402acb0-****-****-****-424c616259bf", <--- Client Secret
  "tenant": "387983da-****-****-****-61869f94518d"
}
```

After executed the above command, **please remember the value of "appId (Service Principal ID)" and "password (Client Secret)"**?  


## Create SSH Keys for login

Please create ssh key file to login to the Azure ACS? In your local environment, please execute ssh-keygen and create key file as follows.  

Create key as yosshi-k8s-acs :  

```
$ ssh-keygen 
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/tyoshio2002/.ssh/id_rsa): yosshi-k8s-acs
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in yosshi-k8s-acs.
Your public key has been saved in yosshi-k8s-acs.pub.
The key fingerprint is:
SHA256:jN4a+q1h9cspxZMA+vIEUTE0YH9yAga6xLO721ylMgI yoterada@Yoshio-no-MacBook-Pro.local
The key's randomart image is:
+---[RSA 2048]----+
|  ..=+*.         |
|.. o.o.o         |
|.+   o+.o        |
|..o o  B.        |
|E.   o..So .     |
|. . ..=o .=      |
| o o *= ....     |
|  = +o.=.. o     |
| o.o..+...+      |
+----[SHA256]-----+


$ ls ~/.ssh/yosshi-k8s-acs*
/Users/foo/.ssh/yosshi-k8s-acs		/Users/foo/.ssh/yosshi-k8s-acs.pub
```

## Create Azure Container Service

After finished your preparation, now we can create the k8s on Azure Container Service.  

At first, please acces to the [Azure Admin Portal](http://portal.azure.com)? Then you can see like following screen. To crete the "Azure Container Service", please push the **"+ New"** button on the left menue?  

![Create Azure Container Service1](https://c1.staticflickr.com/5/4271/34788905411_cef88b0541_z.jpg)

Then you can see the following additional menue. In the search field, please input the **"Container"**? Then you can see the **"Azure Container Service"** as one of the candidate. Please push it?  

![Create Azure Container Service2](https://c1.staticflickr.com/5/4228/34757019452_43fe60951d_z.jpg)

Then you can see the following screen. Please push the **"Azure Container Service"**?  

![Create Azure Container Service](https://c1.staticflickr.com/5/4251/34757019082_4c2cb5543a_z.jpg)

After push the button, you can see the following screen. Please push the **"Create"** button?  

![Create Azure Container Service4](https://c1.staticflickr.com/5/4223/34757019232_b39a527afb_z.jpg)

Then you can see the following screen. At first, please select the **"Orchestrator"** as **"Kubernetes"**?  

![Create Azure Container Service5](https://c1.staticflickr.com/5/4275/34757019412_8cb6fac1c4_z.jpg)

After that, please input the remain field like Subscription, Resource Group name and Location? After you fill out all field, please push the "OK" button?  

![Create Azure Container Service6](https://c1.staticflickr.com/5/4249/34757019582_5d10aaa750_z.jpg)

Then you can see the following screen.   

![Create Azure Container Service7](https://c1.staticflickr.com/5/4269/34757019922_bd521970fc_z.jpg)

In this screen, please fill out all the fields like follows?  

|Input Field| Input Value|
|---|---|
|DNS name prefix|yosshi-k8s|
|User name|yosshi|
|SSH public key|Copy & Paste of following "cat" command|
|Service principal client ID|Please refer to the preparation section again? It looks like following. 09aa4718-****-****-****-83cf60696d14|
|Service principal client secret|Please refer to the preparation section again? It looks like following. 2402acb0-****-****-****-424c616259bf|
|Master count|1|

**Please note that in this Hands on Lab, I selected Master count as 1. However if you need to manage mission critical environment, please increase the number of the master count?**  

You can confirm your SSH public key as follows.  

```
$ cat ~/.ssh/yosshi-k8s-acs.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDolndlxdttOgh
Oo6x0bEsZVQ7XOzKq2lb/******************************
****************************************/**********
***************************************************
***************************************************
********************************************/******
********+******************************************
***********************
```

After fill out the field, please push the "OK" button?  
Then you can see the following screen.  

![Create Azure Container Service8](https://c1.staticflickr.com/5/4202/34788906011_ea3323b71d_z.jpg)

As default, **"Agent Count"** was set as 1. However if you would like to operate more, soon the machine capacity will be over. So please increase the number? In this example, I configure it as 3.
Please push the **"OK"** button?
Then you can see the following screen.  

![Create Azure Container Service9](https://c1.staticflickr.com/5/4247/34757020172_9ea6f0f238_z.jpg)

After the evaluation, you become to be able to push the "OK" buttion. Push the "OK" buttion?  
After push the button, you can see the following screen.  

![Create Azure Container Service10](https://c1.staticflickr.com/5/4245/34788906251_cde41ea3cd_z.jpg)

All of the creation step had done, you can see the following screen.  

![Create Azure Container Service11](https://c1.staticflickr.com/5/4202/34078256974_271767abcf_z.jpg)

In order to login to the master node of the Azure Container Service, please scroll down the screen and please find the master machine? It include the name of "master".  
After you find it, please push the link? 

![Create Azure Container Service12](https://c1.staticflickr.com/5/4204/34789425311_865c62025a_z.jpg)

Then you can see the following screen. And please confirm the **DNS name** of the master node?  
In this example, the DNS of the master node was "yosshi-k8smgmt.japanwest.cloudapp.azure.com".

![Create Azure Container Service13](https://c1.staticflickr.com/5/4247/34078257494_7bf09b39c7_z.jpg)


## Login to the master node of ACS

After you created the ACS, you can login to the master node of ACS with your SSH private key. Please execute the following command and login to the system?

```
$ ssh yosshi@yosshi-k8smgmt.japanwest.cloudapp.azure.com -A -i ~/.ssh/yosshi-k8s-acs
The authenticity of host 'yosshi-k8smgmt.japanwest.cloudapp.azure.com (52.***.***.139)' can't be established.
ECDSA key fingerprint is SHA256:q3agE7fJaAHhDrRPqnr/JuAUckA8DauTs+5Vvc/SrXc.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'yosshi-k8smgmt.japanwest.cloudapp.azure.com,52.***.***.139' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 16.04 LTS (GNU/Linux 4.4.0-28-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

193 packages can be updated.
105 updates are security updates.



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.


yosshi@k8s-master-27AF23F9-0:~$ 
```

After login to the system, you can execute **kubectl** like follows.

```
yosshi@k8s-master-27AF23F9-0:~$ kubectl get nodes
NAME                    STATUS                     AGE
k8s-agent-27af23f9-0    Ready                      6m
k8s-agent-27af23f9-1    Ready                      6m
k8s-agent-27af23f9-2    Ready                      6m
k8s-master-27af23f9-0   Ready,SchedulingDisabled   6m
```

Please confirm the version of k8s like follows?

```
yosshi@k8s-master-27AF23F9-0:~$ kubectl version
Client Version: version.Info{Major:"1", Minor:"5", GitVersion:"v1.5.3", GitCommit:"029c3a408176b55c30846f0faedf56aae5992e9b", GitTreeState:"clean", BuildDate:"2017-02-15T06:40:50Z", GoVersion:"go1.7.4", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"5", GitVersion:"v1.5.3", GitCommit:"029c3a408176b55c30846f0faedf56aae5992e9b", GitTreeState:"clean", BuildDate:"2017-02-15T06:34:56Z", GoVersion:"go1.7.4", Compiler:"gc", Platform:"linux/amd64"}
```

## Scale VM from Azure CLI

You can scale the VM by using Azure CLI command.

### "az acs scale" command Required Parameters

|Parameter| Description|
|---|---|
|--name -n|The name of the container service.|
|--resource-group -g|Name of resource group. You can configure the default group using az configure --defaults group=|
|--new-agent-count|The number of agents for the cluster.|

In order to confirm the name of the Azure Container Service, please refer to the Azure Portal Screen, especially **please find the "Container Service" from the "TYPE"?** It is the value which should specify the argument for "-n". In this example it is "containerservice-k8s-ACS".  
If you find it, please push the link of the "Container Service"?


![](https://c1.staticflickr.com/5/4249/34790933611_2c0ea75b01_z.jpg)

If you push the link of the "Container Service", you can see the following screen. In this screen, you can also confirm the name of "Resource Group" In this example the Resource Grup name is "k8s-ACS". And you should specify it to the argument of "-g".

![](https://c1.staticflickr.com/5/4270/34882499026_83e2603178_z.jpg)

After you got the above informations, please execute the "az acs scale" command like follows?

```
$ az acs scale -n containerservice-k8s-ACS -g k8s-ACS --new-agent-count 5
{
  "agentPoolProfiles": [
    {
      "count": 5,
      "dnsPrefix": "yosshi-k8sagents",
      "fqdn": "",
      "name": "agentpool",
      "vmSize": "Standard_DS2"
    }
  ],
  "customProfile": null,
  "diagnosticsProfile": {
    "vmDiagnostics": {
      "enabled": false,
      "storageUri": null
    }
  },
  "id": "/subscriptions/acc75193-****-****-****-401e1a2bde7e/resourceGroups/k8s-ACS/providers/Microsoft.ContainerService/containerServices/containerservice-k8s-ACS",
  "linuxProfile": {
    "adminUsername": "yosshi",
    "ssh": {
      "publicKeys": [
        {
          "keyData": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDolndlxdttOgh
Oo6x0bEsZVQ7XOzKq2lb/******************************
****************************************/**********
***************************************************
***************************************************
********************************************/******
********+******************************************
***********************"
        }
      ]
    }
  },
  "location": "japanwest",
  "masterProfile": {
    "count": 1,
    "dnsPrefix": "yosshi-k8smgmt",
    "fqdn": "yosshi-k8smgmt.japanwest.cloudapp.azure.com"
  },
  "name": "containerservice-k8s-ACS",
  "orchestratorProfile": {
    "orchestratorType": "Kubernetes"
  },
  "provisioningState": "Succeeded",
  "resourceGroup": "k8s-ACS",
  "servicePrincipalProfile": {
    "clientId": "09aa4718-****-****-****-83cf60696d14",
    "secret": null
  },
  "tags": null,
  "type": "Microsoft.ContainerService/ContainerServices",
  "windowsProfile": null
}
```

For more detai of "az acs" command, please refer to the [Container Service - az acs](https://docs.microsoft.com/en-us/cli/azure/acs) command reference

Finally, please refer to the Azure Portal again? And please confirm the number of agent had increased from 3 to 5?

![](https://c1.staticflickr.com/5/4203/34080026354_83f0fcf8d5_z.jpg)