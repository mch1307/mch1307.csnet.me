---
author:  "mch1307"
categories: ["containers"]
tags: ["rancher","traefik"]
date: "2017-07-12"
description: Traefik as a Dynamically-Configured Proxy and Load-Balancer
featured: grpc.png
featuredpath : date
linktitle: Traefik as a Dynamically-Configured Proxy and Load-Balancer
title: Traefik as a Dynamically-Configured Proxy and Load-Balancer
type: post

---
_**This post has also been published on [Rancher.com][1] on 12-07-2017**_

When deploying applications in the container world, one of the less obvious points is how to make the application available to the external world, outside of the container cluster. One option is to use the host port, which basically maps one port of the host to the container port where the application is exposed. While this option is fine for local development, it is not viable in a real cluster with many applications deployed. One solution is to use an HTTP proxy/load balancer. This container will be exposed using standard HTTP/HTTPS ports on the host, and will route traffic to the application running as a container.

In this post, we will setup Traefik as an HTTP proxy / load balancer for web services running in a Rancher Cattle setup. Traefik will dynamically update its configuration using the Rancher API. An SSL wildcard cert will be used. The (nice) Let’s Encrypt ACME feature Traefik is offering will not be used here. We will make use of Rancher secret feature. If you plan to use Traefik with Let’s Encrypt SSL certs, I encourage you to use the Traefik stack available in Rancher Community Catalog.

### Context

In this demo, we already have a running Rancher environment, using the Cattle orchestrator with three nodes and one web application to be published so that it can be accessed outside of the Cattle cluster. Our application will be the simple “<a href="https://github.com/emilevauge/whoamI" target="_blank" rel="noopener noreferrer">whoamI</a>” application, basically showing the IP address of the node it is running on.

We will use a wildcard SSL domain, as the Rancher install is not available from outside the LAN (on-premises platform). We will also leverage Rancher’s experimental secrets to store the SSL certs, as well as the Rancher API key and secret.

### Rancher Setup

We have three nodes running on our Cattle cluster:

![](/wp-content/uploads/2017/08/traefik-rancher-cluster1.png)

_Notice the `traefik.lb=true` label on some hosts. We will use them later._

### Deploy Web Application

We will create a new stack for our test app. Create a blank stack, we will call it _iamfoo_. Then add a service using the [emilevauge/whoami][2] image. Add an HTTP health check as follows:

&nbsp;

![](/wp-content/uploads/2017/08/rancher-traefik-whoami.png")

### Prepare API Key

Let’s create a Rancher API key: in the UI, go to API\key, click on “Advanced” and create a new key by hitting the “Add environment API key” button:

![](/wp-content/uploads/2017/08/rancher-api-key.png")

![](/wp-content/uploads/2017/08/api-key-created.png")


*_**Note: **As warned by the UI, write down the secret key as you won’t be able to see it again_

Repeat same operation for the API password, SSL cert and SSL key.

### Deploying Traefik
  
We will use the [mch1307/rancher-traefik:1.3.1.2][3] image. It is an alpine based image, inspired from [rawmind0][4]. 

The image will read the SSL certs and Rancher API key/secret from Rancher secret through environment variables that we will define when creating the service.Create new stack, we will call it prx. 

Then add a service to it.

![](/wp-content/uploads/2017/08/deploy-traefik.png)
  
  <p>
    following ports:
  </p>
  
  <ul>
    <li>
      80 -> http
    </li>
    <li>
      443 -> https
    </li>
    <li>
      8000 -> Traefik UI
    </li>
  </ul>
  
  <p>
    Those ports will be exposed on the host(s) that will run the Traefik container(s). Make sure they are not yet in use, or choose other ones.
  </p>
  

### Secrets

  
In the Secrets tab, set up your secrets:

![](/wp-content/uploads/2017/08/setup-secrets.png)

      
Container environment variables

To add secrets as environment variables, go to the “Command” tab and define environment variable as follows. The name of the env vars should be:
        
* TRAEFIK_RANCHER_ACCESSKEY = Rancher API key
* TRAEFIK_RANCHER_SECRET = Rancher API secret
* TRAEFIK_SSL_CERT= Wildcard SSL cert (bundle ii with intermediate CA if applicable)
* TRAEFIK_SSL_PRIVATE_KEY = SSL private key (without password protection)
* TRAEFIK_RANCHER_ENDPOINT = URL to Rancher API
* TRAEFIK_RANCHER_DOMAIN = domain to be used in Traefik
        
The secrets are available at /run/secrets/alias. Alias is defined in the secrets in the previous step.
      
>  Hint: Copy the table above and paste it to the first environment variable. Rancher UI will create all the variables so that you only need to put appropriate value in the right column.
      
### Scheduling
      

Go to the “scheduling” tab and create a new scheduling rule. We will create a rule based on host label. It means we will ask Rancher to run the container on host having a given label. Remember the labels we saw in the Rancher infrastructure (`traefik_lb=true`), this is where they will enter into action:
      
![](/wp-content/uploads/2017/08/scheduling.png) 

Once done, click on “Create” to trigger container deployment. Rancher will download the image from Docker Hub and them schedule the container. You should have a running Traefik stack within a few minutes.
          

![](/wp-content/uploads/2017/08/traefik-stack-rancher.png)

          
### Setup the Web Application

Traefik needs a few labels at the web app level in order to automatically create the config for the service. These are the different labels we will use:
* traefik.enable = true
* traefik.port = 80
* traefik.frontend.rule = Host:whoami.domain.com
                
Go to the whoami stack, choose upgrade and add the labels.
                      
![](/wp-content/uploads/2017/08/traefik-labels.png)
                    
>Hint: </strong>Copy the table above and paste it to the first label. Rancher UI will create all the labels and values. Then adapt the values to suit your setup. Additional ones can be used, refer to the <a href="https://docs.traefik.io/toml/#rancher-backend">Traefik documentation</a>
                    
Once the whoami stack is upgraded, you should be able to access the traefik UI on port 8000:
![](/wp-content/uploads/2017/08/access-traefik-port.png)
                          

### Testing Traefik

You should now define a DNS alias for the whoami that points to the rancher host(s) that runs the traefik container. Once done open your favorite web browser and go to the whoami URL.
![](/wp-content/uploads/2017/08/whoami.png)
                        

Traefik will redirect the HTTP traffic to HTTPS using the wildcard cert we configured earlier. Hit F5 several times, you will see different IPs which shows the load balancing feature of Traefik. You can run further test by scaling up or down the number of whoami containers. Check the traefik UI to see the number of whoami backends is updated.

                        
![](/wp-content/uploads/2017/08/scaling.png)
                    
### Conclusion

We have seen how traefik can be deployed as proxy / load balancer in a rancher cattle cluster, using basic setup. Traefik offers many other options, consult the <a href="https://docs.traefik.io/">documentation</a> for further information


 [1]: http://rancher.com/setting-up-traefik-as-a-dynamically-configured-proxy-and-load-balancer/
 [2]: https://github.com/emilevauge/whoamI
 [3]: https://github.com/mch1307/rancher-traefik
 [4]: https://github.com/rawmind0/rancher-traefik
