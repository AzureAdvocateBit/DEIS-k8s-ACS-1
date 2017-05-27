# Run the Docker Image on Azure Web App Linux

In this seciton, you can run the Docker image on Azure Web App Linux(Preview).

Perhaps already you created the "Azure Container Registry" and push to some image as follows.
[Create Azure Container Registry (Private Docker Registry)](https://github.com/yoshioterada/DEIS-on-k8s-on-ACS/blob/master/CreateAzureContainerRegistry.md)  
If you didn't create it yet now, please create it before proceeding the following?  


## Nginx App on Docker on Azure Web App Linux(Preview)

If you login to the ["Azure Portal"](http://portal.azure.com "Azure Admin Portal"), you can see the following screen. In order to create the new "Web App Linux(Preview)", please push the link of "+New" on the left side of the panel?  

![](https://c1.staticflickr.com/5/4270/34908315655_8765d0f78f_z.jpg "")

If you push the link, you may see the search window with some technical category list. In this list, please select "Web + Mobile" category?  

![](https://c1.staticflickr.com/5/4222/34867824506_97e2e0096c_z.jpg)

After you selected the category, you can see some "FEATURED APPS". In this list, please select **"Web App On Linux (preview)"** ?  

![](https://c1.staticflickr.com/5/4200/34867824726_2af7ea741f_z.jpg)

Then you can see some input form like follows. Please fill out the input form like follows.  

| Input Field | Input value |
|---|---|
| App name | yosshi-docker |
| Subscription | ****** |
| Resource Group | Yosshi-Docker-WebAppLinux |
| App Service plan(Location) | ***** |

After inputed the above, please push the **"Configure Container"** ?

Then you can configure the Docker Container information. In this time please select **Private Registry"**? And fill out like following information.  

| Input Field | Input value |
|---|---|
| Image and Optional tag| yosshi.azurecr.io/tyoshio2002/mynginx:1.0 |
| Server URL | https://yosshi.azurecr.io|
| Login username | yosshi|
| Password| **************** (Azure Container Registry Access Key Password) |

After finished the input, please push the "OK" button?  

![](https://c1.staticflickr.com/5/4221/34908315755_d7e29af8f1_z.jpg)

Please push the "Create" button?

![](https://c1.staticflickr.com/5/4252/34867825146_ca320ba4cf_z.jpg)

Then it will start to create the Azure Web App on Linux. Please wait a moment until finish the creation?

![](https://c1.staticflickr.com/5/4275/34908315865_d3bac7d953_z.jpg)

After created the "Azure WebApp Linux", you can confirm the following screen.

![](https://c1.staticflickr.com/5/4267/34097373123_26ac77ced2_z.jpg)

Please copy the URL address for the service on right side of the link?

![](https://c1.staticflickr.com/5/4220/34867828636_3629b13685_z.jpg)

Finally, please access to the Azure WebApp Linux by using browser? It is very easy !!  

![](https://c1.staticflickr.com/5/4223/34097371833_2fa1179d73_z.jpg)

## Java App on Docker on Azure Web App Linux(Preview)

In the above confirmation, you can use the Nginx Docker image. In this section, you will be able to confirm how to update or change the Docker image as follows.  

At first, please push the link of **"Docker Container"** ? 

![](https://c1.staticflickr.com/5/4226/34521111820_143607c24b_z.jpg)

Then you can see the following screen, then you can update or change the name of the image or number of tags. In this time, I will change the docker image from nginx to Java Application. So please change the imange name to **"helloworld:1.0"** ?  

Ather filed is same as bofore.

| Input Field | Input value |
|---|---|
| Image and Optional tag| yosshi.azurecr.io/tyoshio2002/helloworld:1.0 |
| Server URL | https://yosshi.azurecr.io|
| Login username | yosshi|
| Password| **************** (Azure Container Registry Access Key Password) |

You changed the image name, please push the "Save" button?

![](https://c1.staticflickr.com/5/4227/34908316015_8198bd1f56_z.jpg)

After finished the saving, you can see the following dialog on the screen.

![](https://c1.staticflickr.com/5/4201/34097372033_18c4406ca3_z.jpg)

### Application Port Binding Configuration
In the above nginx sample, it was provided the HTTP service with 80 port as default. However sometimes you would like to provide or **EXPOSE** the service with another port number.  
In this Java Application, it provide the service with 8080 port. So you need to configure the Port binding configuration as follows. In order to configure the port binding, please push the **"Application Setting"** from left side of the menu?    

![](https://c1.staticflickr.com/5/4250/34097372753_bb5c63e7e6_z.jpg)

If you push the link, you can see the following screen. Please scroll down the screen?  

![](https://c1.staticflickr.com/5/4247/34867827856_e98f2bc91f_z.jpg)

If you scroll down the screen, you can see the additional input form like follows. Then please input the key as **"PORT"** and value as **"8080"**? After input it, please push the "Save" button on top of the screen?  

![](https://c1.staticflickr.com/5/4195/34521111950_8cc639b98b_z.jpg)

After finished the saving process, you can see the following diaglog.  

![](https://c1.staticflickr.com/5/4199/34867829386_8f56448cb4_z.jpg)

After finished the saving, please access to the same URL? Then you noticed that you can confirm the same screen of old version (in this time nginx). Please don't worrry about it? Because the system need to download the new image and prepare the service start. And it will take a time. So please wait a moment ?  

![](https://c1.staticflickr.com/5/4223/34097371833_2fa1179d73_z.jpg)

After some second later, you can access to the new version of the service. In the following screen, you can confirm that it started the HTTP daemon.   

![](https://c1.staticflickr.com/5/4225/34908316715_47683d2b94_z.jpg)

Finally you can see the new version of the service as follows.

![](https://c1.staticflickr.com/5/4272/34097373023_57685683b6_z.jpg)


