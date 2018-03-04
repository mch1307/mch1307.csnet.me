---
title: 'Traefik/Consul demo app: my first Go “dev”'
author: mch1307
type: post
date: 2017-08-20
url: /2017/08/20/traefikconsul-demo-app-my-first-go-dev/
featured: tr-consul.png
featuredpath: date
categories:
  - Uncategorized
tags:
  - consul
  - demo
  - golang
  - traefik

---
If you don&#8217;t mind my interesting story, scroll down to the &#8220;quick demo overview&#8221; 😉

In the last weeks, I have been playing with HashiCorp&#8217;s Consul, for service registration and key/value store.

After playtime, I had to introduce the product to internal Teams, so I prepared a few slides on the architecture and main features of Consul. To complete my presentation I prepared a brief demo which was first using consul-template watching for Consul service catalog and restarting the simple, static HAProxy I had setup in front on my &#8220;demo&#8221; web app.

The demo web app was the [&#8220;whoamI&#8221;][1] app, &#8220;a tiny Go web server that prints os information&#8221;, from Emile Vauge, the creator of [Traefik proxy][2]. I initially had seen him using it during one cool [Traefik demo][3] and already used it for the [Traefik on Rancher Cattle post][4]

The &#8220;problem&#8221;, for me, was that whoamI is available as a docker image but not as a simple executable, and I had no Docker engine available for my demo.

As learning Go was one of my &#8220;wish-list&#8221;, I decided to clone the GitHub repo, compile it and have it running for my small demo. This was not a big deal and I easily finished the first version of my demo which was consisting in stopping one of the whoamI instances and having consul-template automatically updating a static HAProxy config and restarting HAProxy.

While reviewing my simple demo I said &#8220;Ok, nice. But should better illustrate the service registration and key/value possibilities&#8221;.

So I started adding the consul client to the &#8220;whoamI&#8221; application so that it would register/deregister itself to the service catalog. I took me a few evenings, but at the end I managed to have it working!!

The demo took more value, at least to my eyes, but I wanted something a bit more &#8220;visual&#8221;. So I bravely decided to change the web page showed by the application by adding a banner, that would be read from the Consul k/v store. Fortunately, this time it was slightly easier for me and I didn&#8217;t have to spend another few evenings to achieve that.

As I like Traefik and that it supports Consul back-end (among many others), I dropped HAProxy and consul-template from the demo and ended also briefly introducing Traefik 😉

It&#8217;s for sure not the best Go code you will find, but it does the job and hopefully can help anyone wanting to make a simple presentation/demo of Traefik and Consul.

The code is available [on GitHub][5], any comment/feedback/star is welcome.

&nbsp;

### **Quick demo overview:**

We assume you have a working [Consul][6] cluster/host, and a Traefik proxy connected to the Consul backend. You will also need a DNS alias pointing whoami to the Traefik host. We will run the whoamI app on 2 hosts.

Parameters available when starting the app:

> <pre>-consul string
     Consul service catalog address
 -consulPort int
     Consul service catalog port (default 8500)
 -consulToken string
     Consul ACL token (optional)
 -kvPath string
     Consul KV path for banner (optional) (default "PUBLIC/whoamI")
 -port int
     Port number for HTTP listen (default 8080)
 -service string
     Service name that will be registered (fqdn better) (default "whoamI")</pre>

Let&#8217;s start the app on two hosts:

> whoamI-consul -port 8080 -consul consul.csnet.me -kvPath PUBLIC/whoamI -service whoami.csnet.me

Once the app is started, go to Consul UI and check the service tab:

<img class="alignnone size-full wp-image-109" src="http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-22-17-at-08-12-am.png" alt="Screen Shot 08-22-17 at 08.12 AM" width="1551" height="626" srcset="http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-22-17-at-08-12-am.png 1551w, http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-22-17-at-08-12-am-300x121.png 300w, http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-22-17-at-08-12-am-768x310.png 768w, http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-22-17-at-08-12-am-1024x413.png 1024w" sizes="(max-width: 767px) 89vw, (max-width: 1000px) 54vw, (max-width: 1071px) 543px, 580px" />

Check the load balancer state on Traefik UI:

<img class="alignnone size-full wp-image-110" src="http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-22-17-at-08-22-am.png" alt="Screen Shot 08-22-17 at 08.22 AM" width="1463" height="652" srcset="http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-22-17-at-08-22-am.png 1463w, http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-22-17-at-08-22-am-300x134.png 300w, http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-22-17-at-08-22-am-768x342.png 768w, http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-22-17-at-08-22-am-1024x456.png 1024w" sizes="(max-width: 767px) 89vw, (max-width: 1000px) 54vw, (max-width: 1071px) 543px, 580px" />

We have no k/v setup on Consul at PUBLIC/whoamI:

<img class="alignnone size-full wp-image-98" src="http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-20-17-at-09-08-pm.png" alt="Screen Shot 08-20-17 at 09.08 PM" width="1790" height="737" srcset="http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-20-17-at-09-08-pm.png 1790w, http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-20-17-at-09-08-pm-300x124.png 300w, http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-20-17-at-09-08-pm-768x316.png 768w, http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-20-17-at-09-08-pm-1024x422.png 1024w" sizes="(max-width: 767px) 89vw, (max-width: 1000px) 54vw, (max-width: 1071px) 543px, 580px" />

Browse to the app web page, the default banner is displayed, and the requests are balanced among the 2 hosts. If we stop the app one one of the 2 hosts, Traefik will automatically stop sending requests to the &#8220;failing&#8221; host.

<img class="alignnone size-full wp-image-111" src="http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-22-17-at-08-30-am.png" alt="Screen Shot 08-22-17 at 08.30 AM" width="445" height="279" srcset="http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-22-17-at-08-30-am.png 445w, http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-22-17-at-08-30-am-300x188.png 300w" sizes="(max-width: 445px) 100vw, 445px" />

<img class="alignnone size-full wp-image-112" src="http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-22-17-at-08-30-am-001.png" alt="Screen Shot 08-22-17 at 08.30 AM 001" width="445" height="279" srcset="http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-22-17-at-08-30-am-001.png 445w, http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-22-17-at-08-30-am-001-300x188.png 300w" sizes="(max-width: 445px) 100vw, 445px" />

&nbsp;

Now we setup the banner k/v in Consul:

<img class="alignnone size-full wp-image-97" src="http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-20-17-at-09-07-pm.png" alt="Screen Shot 08-20-17 at 09.07 PM" width="1737" height="614" srcset="http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-20-17-at-09-07-pm.png 1737w, http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-20-17-at-09-07-pm-300x106.png 300w, http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-20-17-at-09-07-pm-768x271.png 768w, http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-20-17-at-09-07-pm-1024x362.png 1024w" sizes="(max-width: 767px) 89vw, (max-width: 1000px) 54vw, (max-width: 1071px) 543px, 580px" />

And refresh our browser to see the new banner displayed:

<span style="color:#808080;"><em><strong><img class="alignnone size-full wp-image-113" src="http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-22-17-at-08-33-am.png" alt="Screen Shot 08-22-17 at 08.33 AM" width="926" height="279" srcset="http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-22-17-at-08-33-am.png 926w, http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-22-17-at-08-33-am-300x90.png 300w, http://10.32.2.123:8080/wp-content/uploads/2017/08/screen-shot-08-22-17-at-08-33-am-768x231.png 768w" sizes="(max-width: 767px) 89vw, (max-width: 1000px) 54vw, (max-width: 1071px) 543px, 580px" /><br /> </strong></em></span>

The source code is available [on GitHub][5], any comment/feedback/star is welcome.

&nbsp;

 [1]: https://github.com/emilevauge/whoamI
 [2]: http://traefik.io
 [3]: https://youtu.be/QvAz9mVx5TI
 [4]: http://blog.csnet.me/2017/07/11/rancher-traefik/
 [5]: https://github.com/mch1307/whoamI-consul
 [6]: https://www.consul.io/