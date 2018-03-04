---
author:  "mch1307"
categories: ["rancher","traefik","containers"]
tags: ["tutorial"]
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

<img class="alignnone size-full wp-image-22350 lazyloaded" src="http://cdn.rancher.com/wp-content/uploads/2017/07/07155202/Traefik-Rancher-cluster.png" alt="" width="768" height="252" />

_Notice the `traefik.lb=true` label on some hosts. We will use them later._

### Deploy Web Application

We will create a new stack for our test app. Create a blank stack, we will call it _iamfoo_. Then add a service using the [emilevauge/whoami][2] image. Add an HTTP health check as follows:

&nbsp;

<img class="alignnone size-full wp-image-22351 lazyloaded" src="http://cdn.rancher.com/wp-content/uploads/2017/07/07155546/Rancher-Traefik-Whoami.png" alt="" width="768" height="383" />

### Prepare API Key

Let’s create a Rancher API key: in the UI, go to API\key, click on “Advanced” and create a new key by hitting the “Add environment API key” button:

<img class="alignnone size-full wp-image-39" src="http://10.32.2.123:8080/wp-content/uploads/2017/08/rancher-api-key.png" alt="Rancher-API-key" width="768" height="344" srcset="http://10.32.2.123:8080/wp-content/uploads/2017/08/rancher-api-key.png 768w, http://10.32.2.123:8080/wp-content/uploads/2017/08/rancher-api-key-300x134.png 300w" sizes="(max-width: 767px) 89vw, (max-width: 1000px) 54vw, (max-width: 1071px) 543px, 580px" />

<img class="alignnone size-full wp-image-41" src="http://10.32.2.123:8080/wp-content/uploads/2017/08/api-key-created.png" alt="API-Key-Created.png" width="738" height="337" srcset="http://10.32.2.123:8080/wp-content/uploads/2017/08/api-key-created.png 738w, http://10.32.2.123:8080/wp-content/uploads/2017/08/api-key-created-300x137.png 300w" sizes="(max-width: 738px) 100vw, 738px" />

*_**Note: **As warned by the UI, write down the secret key as you won’t be able to see it again_

Repeat same operation for the API password, SSL cert and SSL key.

### Deploying Traefik
  
  <p>
    We will use the <a href="https://github.com/mch1307/rancher-traefik">mch1307/rancher-traefik:1.3.1.2</a>. It is an alpine based image, inspired from <a href="https://github.com/rawmind0/rancher-traefik">rawmind0</a>. The image will read the SSL certs and Rancher API key/secret from Rancher secret through environment variables that we will define when creating the service.Create new stack, we will call it prx. Then add a service to it.
  </p>
  
  <p>
    <img class="alignnone size-full wp-image-45" src="http://10.32.2.123:8080/wp-content/uploads/2017/08/deploy-traefik.png" alt="Deploy-Traefik" width="768" height="316" srcset="http://10.32.2.123:8080/wp-content/uploads/2017/08/deploy-traefik.png 768w, http://10.32.2.123:8080/wp-content/uploads/2017/08/deploy-traefik-300x123.png 300w" sizes="(max-width: 767px) 89vw, (max-width: 1000px) 54vw, (max-width: 1071px) 543px, 580px" />
  </p>
  
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

  
  <div id="page" class="hfeed site">
    <div id="content" class="site-content container clearfix">
      <section id="primary" class="content-single content-area"> <article id="post-32" class="post-32 post type-post status-publish format-standard hentry category-uncategorized tag-cattle tag-rancher tag-traefik"> 
      
      <p class="entry-content clearfix">
        In the Secrets tab, set up your secrets:
      </p>
      
      <p>
        <img class="alignnone size-full wp-image-47" src="http://10.32.2.123:8080/wp-content/uploads/2017/08/setup-secrets.png" alt="Setup-secrets.png" width="768" height="238" srcset="http://10.32.2.123:8080/wp-content/uploads/2017/08/setup-secrets.png 768w, http://10.32.2.123:8080/wp-content/uploads/2017/08/setup-secrets-300x93.png 300w" sizes="(max-width: 767px) 89vw, (max-width: 1000px) 54vw, (max-width: 1071px) 543px, 580px" />
      </p>
      
      <div class="entry-content clearfix">
        <p>
          Container environment variables
        </p>
        
        <p>
          To add secrets as environment variables, go to the “Command” tab and define environment variable as follows. The name of the env vars should be:
        </p>
        
        <ul>
          <li>
            TRAEFIK_RANCHER_ACCESSKEY = Rancher API key
          </li>
          <li>
            TRAEFIK_RANCHER_SECRET = Rancher API secret
          </li>
          <li>
            TRAEFIK_SSL_CERT= Wildcard SSL cert (bundle ii with intermediate CA if applicable)
          </li>
          <li>
            TRAEFIK_SSL_PRIVATE_KEY = SSL private key (without password protection)
          </li>
          <li>
            TRAEFIK_RANCHER_ENDPOINT = URL to Rancher API
          </li>
          <li>
            TRAEFIK_RANCHER_DOMAIN = domain to be used in Traefik
          </li>
        </ul>
        
        <p>
          The secrets are available at /run/secrets/alias. Alias is defined in the secrets in the previous step.
        </p>
      </div>
      
      <p>
        *<em>Hint: Copy the table above and paste it to the first environment variable. Rancher UI will create all the variables so that you only need to put appropriate value in the right column.</em>
      </p>
      
      <h3>
        Scheduling
      </h3>
      
      <p>
        Go to the “scheduling” tab and create a new scheduling rule. We will create a rule based on host label. It means we will ask Rancher to run the container on host having a given label. Remember the labels we saw in the Rancher infrastructure (<code>&lt;em>traefik_lb=true&lt;/em></code>), this is where they will enter into action:
      </p>
      
      <div id="page" class="hfeed site">
        <div id="content" class="site-content container clearfix">
          <section id="primary" class="content-single content-area"> <article id="post-32" class="post-32 post type-post status-publish format-standard hentry category-uncategorized tag-cattle tag-rancher tag-traefik"> 
          
          <div class="entry-content clearfix">
            <p>
              <img class="alignnone size-full wp-image-52" src="http://10.32.2.123:8080/wp-content/uploads/2017/08/scheduling.png" alt="Scheduling" width="768" height="161" srcset="http://10.32.2.123:8080/wp-content/uploads/2017/08/scheduling.png 768w, http://10.32.2.123:8080/wp-content/uploads/2017/08/scheduling-300x63.png 300w" sizes="(max-width: 767px) 89vw, (max-width: 1000px) 54vw, (max-width: 1071px) 543px, 580px" />
            </p>
            
            <p>
              &nbsp;
            </p>
          </div>
          
          <p>
            Once done, click on “Create” to trigger container deployment. Rancher will download the image from Docker Hub and them schedule the container. You should have a running Traefik stack within a few minutes.
          </p>
          
          <p>
            <img class="alignnone size-full wp-image-54" src="http://10.32.2.123:8080/wp-content/uploads/2017/08/traefik-stack-rancher.png" alt="Traefik-Stack-Rancher" width="768" height="86" srcset="http://10.32.2.123:8080/wp-content/uploads/2017/08/traefik-stack-rancher.png 768w, http://10.32.2.123:8080/wp-content/uploads/2017/08/traefik-stack-rancher-300x34.png 300w" sizes="(max-width: 767px) 89vw, (max-width: 1000px) 54vw, (max-width: 1071px) 543px, 580px" />
          </p>
          
          <h3>
            Setup the Web Application
          </h3>
          
          <div id="page" class="hfeed site">
            <div id="content" class="site-content container clearfix">
              <section id="primary" class="content-single content-area"> <article id="post-32" class="post-32 post type-post status-publish format-standard hentry category-uncategorized tag-cattle tag-rancher tag-traefik"> 
              
              <div class="entry-content clearfix">
                <p>
                  Traefik needs a few labels at the web app level in order to automatically create the config for the service. These are the different labels we will use:
                </p>
                
                <ul>
                  <li>
                    traefik.enable = true
                  </li>
                  <li>
                    traefik.port = 80
                  </li>
                  <li>
                    traefik.frontend.rule = Host:whoami.domain.com
                  </li>
                </ul>
                
                <div id="page" class="hfeed site">
                  <div id="content" class="site-content container clearfix">
                    <section id="primary" class="content-single content-area"> <article id="post-32" class="post-32 post type-post status-publish format-standard hentry category-uncategorized tag-cattle tag-rancher tag-traefik"> 
                    
                    <div class="entry-content clearfix">
                      <p>
                        Go to the whoami stack, choose upgrade and add the labels.
                      </p>
                      
                      <p>
                        <img class="alignnone size-full wp-image-57" src="http://10.32.2.123:8080/wp-content/uploads/2017/08/traefik-labels.png" alt="Traefik-Labels.png" width="768" height="189" srcset="http://10.32.2.123:8080/wp-content/uploads/2017/08/traefik-labels.png 768w, http://10.32.2.123:8080/wp-content/uploads/2017/08/traefik-labels-300x74.png 300w" sizes="(max-width: 767px) 89vw, (max-width: 1000px) 54vw, (max-width: 1071px) 543px, 580px" />
                      </p>
                    </div>
                    
                    <div class="entry-content clearfix">
                      <p>
                        <em><strong>Hint: </strong>Copy the table above and paste it to the first label. Rancher UI will create all the labels and values. The adapt the values to suit your setup. </em><em>Additional ones can be used, refer to the <a href="https://docs.traefik.io/toml/#rancher-backend">Traefik documentation</a></em>
                      </p>
                    </div>
                    
                    <div id="page" class="hfeed site">
                      <div id="content" class="site-content container clearfix">
                        <section id="primary" class="content-single content-area"> <article id="post-32" class="post-32 post type-post status-publish format-standard hentry category-uncategorized tag-cattle tag-rancher tag-traefik"> 
                        
                        <div class="entry-content clearfix">
                          <p>
                            &nbsp;
                          </p>
                          
                          <p>
                            Once the whoami stack is upgraded, you should be able to access the traefik UI on port 8000:
                          </p>
                          
                          <p>
                            <img class="alignnone size-full wp-image-60" src="http://10.32.2.123:8080/wp-content/uploads/2017/08/access-traefik-port.png" alt="access-traefik-port.png" width="768" height="313" srcset="http://10.32.2.123:8080/wp-content/uploads/2017/08/access-traefik-port.png 768w, http://10.32.2.123:8080/wp-content/uploads/2017/08/access-traefik-port-300x122.png 300w" sizes="(max-width: 767px) 89vw, (max-width: 1000px) 54vw, (max-width: 1071px) 543px, 580px" />
                          </p>
                          
                          <h3>
                            Testing Traefik
                          </h3>
                          
                          <p>
                            You should now define a DNS alias for the whoami that points to the rancher host(s) that runs the traefik container. Once done open your favorite web browser and go to the whoami URL.
                          </p>
                        </div></article> </section> 
                        
                        <p>
                          <img class="alignnone size-full wp-image-64" src="http://10.32.2.123:8080/wp-content/uploads/2017/08/whoami.png" alt="whoami" width="880" height="411" srcset="http://10.32.2.123:8080/wp-content/uploads/2017/08/whoami.png 880w, http://10.32.2.123:8080/wp-content/uploads/2017/08/whoami-300x140.png 300w, http://10.32.2.123:8080/wp-content/uploads/2017/08/whoami-768x359.png 768w" sizes="(max-width: 767px) 89vw, (max-width: 1000px) 54vw, (max-width: 1071px) 543px, 580px" />
                        </p>
                        
                        <p>
                          Traefik will redirect the HTTP traffic to HTTPS using the wildcard cert we configured earlier. Hit F5 several times, you will see different IPs which shows the load balancing feature of Traefik. You can run further test by scaling up or down the number of whoami containers. Check the traefik UI to see the number of whoami backends is updated.
                        </p>
                        
                        <p>
                          <img class="alignnone size-full wp-image-66" src="http://10.32.2.123:8080/wp-content/uploads/2017/08/scaling.png" alt="Scaling-" width="768" height="313" srcset="http://10.32.2.123:8080/wp-content/uploads/2017/08/scaling.png 768w, http://10.32.2.123:8080/wp-content/uploads/2017/08/scaling-300x122.png 300w" sizes="(max-width: 767px) 89vw, (max-width: 1000px) 54vw, (max-width: 1071px) 543px, 580px" />
                        </p>
                      </div>
                    </div>
                    
                    <h3>
                      Conclusion
                    </h3>
                    
                    <div id="page" class="hfeed site">
                      <div id="content" class="site-content container clearfix">
                        <section id="primary" class="content-single content-area"> <article id="post-32" class="post-32 post type-post status-publish format-standard hentry category-uncategorized tag-cattle tag-rancher tag-traefik"> 
                        
                        <div class="entry-content clearfix">
                          <div id="page" class="hfeed site">
                            <div id="content" class="site-content container clearfix">
                              <section id="primary" class="content-single content-area"> <article id="post-32" class="post-32 post type-post status-publish format-standard hentry category-uncategorized tag-cattle tag-rancher tag-traefik"> 
                              
                              <div class="entry-content clearfix">
                                We have seen how traefik can be deployed as proxy / load balancer in a rancher cattle cluster, using basic setup. Traefik offers many other options, consult the <a href="https://docs.traefik.io/">documentation</a> for further information
                              </div></article> </section>
                            </div>
                          </div>
                        </div></article> </section>
                      </div>
                    </div></article> </section>
                  </div>
                </div>
              </div></article> </section>
            </div>
          </div></article> </section>
        </div>
      </div></article> </section>
    </div>
  </div>
</div>

 [1]: http://rancher.com/setting-up-traefik-as-a-dynamically-configured-proxy-and-load-balancer/
 [2]: https://github.com/emilevauge/whoamI