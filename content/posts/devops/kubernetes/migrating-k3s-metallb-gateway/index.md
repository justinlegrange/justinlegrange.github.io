---
title: 'Migrating My K3s: A Postmortem on Adding in MetalLB and the Gateway API'
date: '2026-07-23' # YYYY-MM-DD
lastMod: '2026-07-29'
summary: "In this post, I go over the mistakes I made when trying to migrate my local K3s cluster from ServiceLB to MetalLB and setting up the Traefik Gateway API."
draft: false
# series: ["Themes Guide"]
# series_order: 1
categories: ["kubernetes", "devops"]
tags: ["kubernetes", "metallb", "traefik", "gateway api"]
---

## Why MetalLB + Gateway API?

Good afternoon everyone! If you've messed with Kubernetes homelabbing for a bit, you'll probably know that an on-prem deployment won't have a single entrypoint for your network. This causes issues for DNS among other things - how do we set a DNS entry for a service which can hop from node to node? - and ultimately, that line of thinking lead me to the [open-source MetalLB project](https://metallb.io/). My goal for the project is to create an easy way for me to stand up services, advertise them on the local network, and give an easy point-of-entry for my DNS server to point at for a given service. MetalLB and the Kubernetes Gateway API help solve that dilemma pretty efficiently.

For those unfamiliar, MetalLB is a L2/L4 load balancer that uses either ARP or BGP (depending on configuration) to advertise which IP addresses it controls. The ideal use-case is for on-prem, bare-metal K3s/K8s (or any other variant of K8s) deployments as a substitute for what load balancer functionality would normally be handled by the given cloud provider (GCP, AWS, etc). It gives us the ability to expose network services on a set of addresses, rather than by exposing via `NodePort` services. We can use it in a myriad of ways, including setting the IP it advertises to a single point for an entryway.

That's the point at which the Traefik Gateway API comes in to play. With MetalLB alone, we won't be able to host multiple services - we'd have to traditionally set up something like Ingress to handle the routing portion. The [Kubernetes Gateway API](https://kubernetes.io/docs/concepts/services-networking/gateway/), however, is being written to replace Ingress, and it gives us access to more flexibility and easier configuration for services. It can match based on a combination of hostnames and routes, passing traffic to our backend services.

The idea was that I could set up MetalLB to advertise the traffic, Traefik Gateway API to handle the routing, and I'd quickly be able to set up new services to help me keep learning Kubernetes as I study for my CKA. As you'll see, it doesn't take long for my setup to go sideways!

## Setting up MetalLB

All you need to get started setting up MetalLB is a functioning, bare-metal Kubernetes cluster. For reference, here's what my local cluster setup looks like:
```
[main-pc] - my desktop PC, configured to work with my K3s cluster
[controlplane] - K3s server node
[node1] - K3s worker node #1
[node2] - K3s worker node #2
```

All of the servers I'm using are SFF front-office PCs I got secondhand on eBay or otherwise. I'm not going to go into details on how I got started designing and setting up the cluster itself in this post; the K3s docs have a [fantastic quickstart guide](https://docs.k3s.io/quick-start) that can help get your ideal cluster up-and-running in a matter of minutes.

It's also important to point out that this post will require some swapping between the servers - I'm using SSH, but this could also be a physical connection, or swapping panes in a hypervisor. When I switch, you'll see the prompt change from one box to the other, i.e. from `[controlplane]` to `[node1]` or vice-versa. Just know that those correspond to different users on different boxes and it should all make sense.

Finally, you'll see me using `k <command>` - that's just a shell alias I have set up to shorten `kubectl` down to `k`.

In order to test our configuration changes, we'll need to have a service and deployment running. I keep around a test Nginx service that just runs the `hello-world-nginx` container for these occasions. The manifest for those pieces looks like this...

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: hello-world-nginx
  name: hello-world-nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello-world-nginx
  strategy: {}
  template:
    metadata:
      labels:
        app: hello-world-nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: hello-world-nginx-svc
  name: hello-world-nginx-svc
spec:
  ports:
  - name: default
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: hello-world-nginx
  type: NodePort
```

Simple, right? Now on to the actual MetalLB setup. There's three ways to install - for my use case, I chose to do the manifest deployment. I created a file in `metallb-config/metallb-config.yml` with the following content, IP addresses substituted for generic ones:

```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: homelab-pool
  namespace: metallb-system
spec:
  addresses:
  - 192.168.1.30-192.168.1.50
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: metal-advert
  namespace: metallb-system
spec:
  ipAddressPools:
  - homelab-pool
```

Before we deploy that manifest, we need to restart our server node with the default ServiceLB bundled with K3s _disabled_ to prevent load balancing conflicts:

```console
[controlplane]$ curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server" sh -s - --disable=servicelb
```

Once we've disabled that and our server node is back up and running, we can try to apply our MetalLB config:

```console
[main-pc]$ k apply -f metallb-config/metallb-config.yml
Error from server (InternalError): error when creating "metallb-config/metallb-config.yml": Internal error occurred: failed calling webhook "ipaddresspoolvalidationwebhook.metallb.io": failed to call webhook: Post "https://metallb-webhook-service.metallb-system.svc:443/validate-metallb-io-v1beta1-ipaddresspool?timeout=10s": context deadline exceeded
Error from server (InternalError): error when creating "metallb-config/metallb-config.yml": Internal error occurred: failed calling webhook "l2advertisementvalidationwebhook.metallb.io": failed to call webhook: Post "https://metallb-webhook-service.metallb-system.svc:443/validate-metallb-io-v1beta1-l2advertisement?timeout=10s": context deadline exceeded
```

Well, that's not great. I tried a ton of different troubleshooting steps - checking ports with `ss`, checking services, reading Kubernetes logging, trying to `curl` from one box to another - and on the surface, everything seemed on the up-and-up. No issues graced my screen. In the end, I had to temporarily disable my firewall rules between the hosts to figure out that _they_ were the piece that was causing headaches and communication errors:

```console
[node1]$ sudo systemctl stop firewalld.service
[node2]$ sudo systemctl stop firewalld.service
[...]
[main-pc]$ k apply -f metallb-config/metallb-config.yml
ipaddresspool.metallb.io/homelab-pool created
l2advertisement.metallb.io/metal-advert created
```

Now, if we run `k get svc` we should see our service advertising on an external IP address that's controlled by MetalLB:
```console
[main-pc]$ k get svc
NAME                    TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)          AGE
hello-world-nginx-svc   LoadBalancer   10.43.19.71   192.168.1.30    8080:31766/TCP   113d
kubernetes              ClusterIP      10.43.0.1     <none>          443/TCP          116d
```

Perfect! Let's test with a simple `curl` to see if we can reach the service on our new, shiny MetalLB endpoint.
```console
[main-pc]$ curl 192.168.1.30:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
[...]
```

### Fixing the Firewall

We've confirmed that the firewalls are indeed our problem point, but how do we get around to solving it? This part is pretty easy, just a bit time consuming from repeatedly issuing the commands. First, we remove the MetalLB configuration - we'll want a clean state to start from in order to verify our fix worked:

```console
[main-pc]$ k delete -f metallb-config/metallb-config.yml
ipaddresspool.metallb.io "homelab-pool" deleted from metallb-system namespace
l2advertisement.metallb.io "metal-advert" deleted from metallb-system namespace
```

And now we go about enabling traffic flow - for whichever firewall you have enabled, you'll need to allow traffic from quite a few sources. This is what it looks like setting up rules for `firewalld` on RHEL systems from the [K3s official docs](https://docs.k3s.io/installation/requirements?os=rhel):

```
firewall-cmd --permanent --add-port=6443/tcp #apiserver
firewall-cmd --permanent --zone=trusted --add-source=10.42.0.0/16 #pods
firewall-cmd --permanent --zone=trusted --add-source=10.43.0.0/16 #services
firewall-cmd --reload
```

And similarly, the same operation using `ufw` on Debian systems:
```
ufw allow 6443/tcp #apiserver
ufw allow from 10.42.0.0/16 to any #pods
ufw allow from 10.43.0.0/16 to any #services
```

No matter what firewall you use, this is a (potentially non-exhaustive) list of potential traffic sources you may want to look into adding:
```
# DEFAULT K3S ROUTES
10.42.0.0/16 #pods
10.43.0.0/16 #services

# K3S PORTS: https://docs.k3s.io/installation/requirements?os=debian#inbound-rules-for-k3s-nodes
2379/tcp
2380/tcp
6443/tcp
8472/udp
10250/tcp
51820/udp
51821/udp

# METALLB PORTS: https://metallb.io/#requirements
7946/tcp
7946/udp
9120/tcp
```

But another question comes up - how did I find the _second_ MetalLB port? The first port set (`7946/tcp` and `7946/udp`) is explicitly called out in [the MetalLB requirements section](https://metallb.io/#requirements), but the other I had to dig a bit for. The documentation only mentions that the "other port can be configured" with no references to what that configuration is by default. Eventually, I dug around in [the v0.16.1 manifest](https://raw.githubusercontent.com/metallb/metallb/v0.16.1/config/manifests/metallb-frr-k8s.yaml) and found the following `--port` argument passed to the both the `controller` and `speaker` containers:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: metallb
    component: controller
  name: controller
  namespace: metallb-system
[...]
      spec:
      containers:
      - args:
        - --port=9120
        - --log-level=info
        env:
[...]
```

The individual port may change from version to version - it has in the past, based on some guides I read while troubleshooting - but the method for getting it again _should_ work, barring any breaking changes from the MetalLB team.

As it turned out for my cluster, I had forgotten to add in some of the required K3s default port rules and _those_ were preventing the webhook service from communicating, even though they didn't necessarily have anything to do with the original listed webhook port of `443`. Four-ish hours of late-night troubleshooting later, and we're on our way to the Gateway API!

## Gateway API

The setup for the Gateway API isn't as difficult as I initially thought - and took me 1/16th of the time that our firewall debacle managed to eat up. To deploy the Gateway API, the documentation says we need to deploy a Helm configuration to our K3s instance - the easiest way to do so is to edit `/var/lib/rancher/k3s/server/manifests/traefik-config.yaml`, as [I found in this Github issue](https://github.com/k3s-io/k3s/discussions/11100):

```yaml
# /var/lib/rancher/k3s/server/manifests/traefik-config.yaml
apiVersion: helm.cattle.io/v1
kind: HelmChartConfig
metadata:
  name: traefik
  namespace: kube-system
spec:
  valuesContent: |-
    providers:
      kubernetesGateway:
        enabled: true
```

Verifying the installation is extremely simple. All we need to do is check if we have a Traefik `gatewayclass` object - if so, the Gateway API is up and running:

```console
[main-pc]$ k get gatewayclass
NAME      CONTROLLER                      ACCEPTED   AGE
traefik   traefik.io/gateway-controller   True       58s
```

### Hooking a Gateway Up with a MetalLB IP

If you're doing much Googling by this point like I was, you'll probably hit a _ton_ of articles outlining the process of hooking up the Ingress gateway with MetalLB, but not too many around using the Gateway API with it. Thankfully it's pretty straightforward - Traefik will include a `GatewayController` by default, so we just need to set up a `Gateway` and some `HTTPRoute` objects underneath it to get our traffic flowing.

At least, that's the theory - nothing ever comes _too_ easy in my homelab.

In order to take incoming traffic and route it to the appropriate backend service, we need to create two additional pieces. The `Gateway` houses `listeners` that match traffic based on hostname; when a listener is triggered, it passes the traffic to the `HTTPRoute` [that's most appropriate](https://gateway-api.sigs.k8s.io/docs/concepts/traffic-matching/), which then matches and passes the traffic off to the appropriate service via the `backendRefs` parameter. I set up another manifest for this under `metallb-config/test-gateway-service.yml`:

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: test-nginx-gateway
  namespace: default
spec:
  gatewayClassName: traefik
  listeners:
  - name: http
    protocol: HTTP
    port: 80
    hostname: testnode.jlegrange.dev
    allowedRoutes:
      namespaces:
        from: Same
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: test-nginx-httproute
spec:
  parentRefs:
  - name: test-nginx-gateway
  hostnames:
  - testnode.jlegrange.dev
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: hello-world-nginx-svc
      port: 80
```

Unfortunately for me, applying that manifest using `k apply -f manifest.yml` doesn't work right out of the box:
```console
[main-pc]$ k describe gateway
Name:         test-nginx-gateway
[...]
Listeners:
    Attached Routes:  0
    Conditions:
      Last Transition Time:  2026-07-24T19:24:07Z
      Message:               Cannot find entryPoint for Gateway: no matching entryPoint for port 80 and protocol "HTTP"
      Observed Generation:   3
      Reason:                PortUnavailable
      Status:                False
      Type:                  Accepted
    Name:                    http
```

My first stop was to check out the Traefik `GatewayController` to see if it illuminated any issues with the setup:

```console
[main-pc]$ k get gc
NAME      CONTROLLER                      ACCEPTED   AGE
traefik   traefik.io/gateway-controller   True       15h

[main-pc]$ k describe gc traefik
[...]
  Supported Features:
    Name:  BackendTLSPolicy
    Name:  GRPCRoute
    Name:  Gateway
    Name:  GatewayPort8080
    Name:  HTTPRoute
    Name:  HTTPRouteBackendProtocolH2C
    Name:  HTTPRouteBackendProtocolWebSocket
    Name:  HTTPRouteBackendRequestHeaderModification
    Name:  HTTPRouteDestinationPortMatching
    Name:  HTTPRouteHostRewrite
    Name:  HTTPRouteMethodMatching
    Name:  HTTPRoutePathRedirect
    Name:  HTTPRoutePathRewrite
    Name:  HTTPRoutePortRedirect
    Name:  HTTPRouteQueryParamMatching
    Name:  HTTPRouteResponseHeaderModification
    Name:  HTTPRouteSchemeRedirect
    Name:  ReferenceGrant
    Name:  TLSRoute
    Name:  TLSRouteModeMixed
    Name:  TLSRouteModeTerminate
```

It definitely has the `HTTPRoutes` and all of the other components set up. At first, I thought that the `GatewayPort8080` was the issue, but it turned out to be a red herring - that port is the default for the Traefik dashboard. Here's what the Traefik docs have to say about it:

> [!QUOTE]+ Tip
> In the Helm Chart, the entryPoints web (port 80), websecure (port 443), traefik (port 8080) and metrics (port 9100) are created by default. The entryPoints web, websecure are exposed by default using a Service.
> 
> The default behaviors can be overridden in the Helm Chart.
{icon="fire"}

You'll notice that the Traefik port definitions we have from our config don't necessarily match what's shown in the docs. The Traefik Helm chart allows for some modification, but first let's just get the base install working. Modifying the gateway listener port is the easiest way to produce the desired result:
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: test-nginx-gateway
  namespace: default
spec:
  gatewayClassName: traefik
  listeners:
  - name: http
    protocol: HTTP
    port: 8000
    hostname: testnode.jlegrange.dev
    allowedRoutes:
      namespaces:
        from: Same
```

All that's left is for us to set up DNS in some format so that the host resolves correctly. You can do this in a myriad of ways - dedicated DNS server, the hosts file - I opted to drop an entry into `/etc/hosts`, since this is just a quick test service.

```console
[main-pc]$ curl -I -v http://testnode.jlegrange.dev -H 'Host: testnode.jlegrange.dev'
* Host testnode.jlegrange.dev:80 was resolved.
* IPv6: (none)
* IPv4: 192.168.1.30
*   Trying 192.168.1.30:80...
* Established connection to testnode.jlegrange.dev (192.168.1.30 port 80) from main-pc port 42076
* using HTTP/1.x
> HEAD / HTTP/1.1
> Host: testnode.jlegrange.dev
> User-Agent: curl/8.18.0
> Accept: */*
>
* Request completely sent off
< HTTP/1.1 200 OK
HTTP/1.1 200 OK
< Accept-Ranges: bytes
Accept-Ranges: bytes
< Content-Length: 896
Content-Length: 896
< Content-Type: text/html
Content-Type: text/html
< Date: Fri, 24 Jul 2026 19:37:52 GMT
Date: Fri, 24 Jul 2026 19:37:52 GMT
< Etag: "6a57af42-380"
Etag: "6a57af42-380"
< Last-Modified: Wed, 15 Jul 2026 16:03:14 GMT
Last-Modified: Wed, 15 Jul 2026 16:03:14 GMT
< Server: nginx/1.31.3
Server: nginx/1.31.3
<

* Connection #0 to host testnode.jlegrange.dev:80 left intact
```

Success! We now have a Traefik Gateway API, running on port 8000 - but it irks me that I haven't gotten the ports listening on 80 and 443, respectively...

Let's fix that.

### Breaking What's Already Fixed - Changing Listener Ports

Initially, I tried to modify the listener ports for Traefik by modifying the configuration to something like this (an approximation, because I got excited and forgot to take notes here):

```yaml
# /var/lib/rancher/k3s/server/manifests/traefik-config.yaml
apiVersion: helm.cattle.io/v1
kind: HelmChartConfig
metadata:
  name: traefik
  namespace: kube-system
spec:
  valuesContent: |-
    providers:
      kubernetesGateway:
        enabled: true
    ports:
      web:
        port: 80
      websecure:
        port: 443
    api:
      dashboard: false
    gateway:
      listeners:
        web:
          port: 80
        websecure:
          port: 443
```

This, put bluntly, was never going to work. I wildly misunderstood both the data format it was expecting, and the configuration in general. Saving that config caused the `helm-install` service to enter the `CrashLoopBackOff` state:

```console
[main-pc]$ k get po -n kube-system
NAME                                      READY   STATUS                 RESTARTS       AGE
coredns-5f5694d56b-6ffl2                  0/1     CreateContainerError   1 (22d ago)    13d
helm-install-traefik-crd-qqqgf            0/1     Completed              0              13d
helm-install-traefik-hkt9x                0/1     CrashLoopBackOff       5 (107s ago)   4m58s
local-path-provisioner-58d557dc48-fd4qm   1/1     Running                3 (16h ago)    13d
metrics-server-c8774f4f4-9ppq9            1/1     Running                13 (8d ago)    121d
traefik-7bb6b86f4b-jfqrn                  1/1     Running                0              5d11h
```

Eventually, I tried modifying the `traefik-config.yml` to something closer to this (again, an approximation):
```yaml
# /var/lib/rancher/k3s/server/manifests/traefik-config.yaml
apiVersion: helm.cattle.io/v1
kind: HelmChartConfig
metadata:
  name: traefik
  namespace: kube-system
spec:
  valuesContent: |-
    providers:
      kubernetesGateway:
        enabled: true
    ports:
      web:
        port: 80
        nodePort: 30000
      websecure:
        port: 443
        nodePort: 30001
    api:
      dashboard: false
    gateway:
      listeners:
        web:
          port: 80
          protocol: HTTP
          namespacePolicy:
            from: All
        websecure:
          port: 443
          protocol: HTTPS
          namespacePolicy:
            from: All
          mode: Terminate
```

Deploying that _also_ earned me another `CrashLoopBackOff`, so let's dig in and see what's wrong. If we check the logs for that container, we see an error about the `certificateRefs` not being declared:

```console
[main-pc]$ k logs -n kube-system helm-install-traefik-hkt9x
[...]
+ echo 'Upgrading helm chart'
+ echo 'Upgrading traefik'
+ shift 1
Upgrading traefik
+ helm upgrade --set-string global.systemDefaultRegistry= traefik https://10.43.0.1:443/static/charts/traefik-40.1.3+up40.1.0.tgz --values /config/values-0-000-HelmChart-ValuesContent.yaml --values /config/values-1-000-HelmChartConfig-ValuesContent.yaml
Error: UPGRADE FAILED: execution error at (traefik/templates/gateway.yaml:52:12): ERROR: certificateRefs needs to be specified using HTTPS
```

...which makes a ton of sense. In order to use TLS termination, you need to _give_ the service a TLS key to encrypt/decrypt the traffic with. Let's set up a quick self-signed certificate, just to test that we can get the service functioning correctly - we'll deploy an actual, signed certificate later on.

```console
[controlplane]$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=*.jlegrange.dev"
[controlplane]$ kubectl create ns traefik
[controlplane]$ kubectl create secret tls local-selfsigned-tls --cert=tls.crt --key=tls.key --namespace traefik
```

Now that we have a key, we need to add it to the `traefik-config` under the `certificateRefs` section:

```yaml
# /var/lib/rancher/k3s/server/manifests/traefik-config.yaml
apiVersion: helm.cattle.io/v1
kind: HelmChartConfig
metadata:
  name: traefik
  namespace: kube-system
spec:
  valuesContent: |-
[...]
        websecure:
          port: 443
          protocol: HTTPS
          namespacePolicy:
            from: All
          mode: Terminate
          certificateRefs:
            - kind: Secret
              name: local-selfsigned-tls
              group: ""
```

After we deploy that, the server goes live - we can check our work using the same method as before to see that we have an attached `HTTPRoute`:

```console
[main-pc]$ k describe gateway test-nginx-gateway
[...]
  Listeners:
    Attached Routes:  1
    Conditions:
      Last Transition Time:  2026-07-29T15:27:05Z
      Message:               No error found
      Observed Generation:   8
      Reason:                Accepted
      Status:                True
      Type:                  Accepted
      Last Transition Time:  2026-07-29T15:27:05Z
      Message:               No error found
      Observed Generation:   8
      Reason:                ResolvedRefs
      Status:                True
      Type:                  ResolvedRefs
      Last Transition Time:  2026-07-29T15:27:05Z
      Message:               No error found
      Observed Generation:   8
      Reason:                Programmed
      Status:                True
      Type:                  Programmed
    Name:                    http
```

Now we just need to modify the `Gateway` listener in our manifest to use port 80 instead of 8000...
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: test-nginx-gateway
  namespace: default
spec:
  gatewayClassName: traefik
  listeners:
  - name: http
    protocol: HTTP
    port: 80
    allowedRoutes:
      namespaces:
        from: Same
```

...and once we deploy it, the service should be live!

```console
[main-pc]$ curl testnode.jlegrange.dev:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
[...]
</html>
```

One route down, one to go!

### HTTPS

This section caused a small troubleshooting headache - as soon as I tried setting up a new `TLSRoute`, we got hit with a TLS certificate error.

```console
[main-pc]$ k describe gateway test-nginx-gateway
[...]
Conditions:
      Last Transition Time:  2026-07-29T15:47:46Z
      Message:               Cannot load CertificateRef default/local-selfsigned-tls: getting secret: secret "local-selfsigned-tls" not found
      Observed Generation:   10
      Reason:                InvalidCertificateRef
      Status:                False
      Type:                  ResolvedRefs
      Last Transition Time:  2026-07-29T15:47:46Z
      Message:               Invalid CertificateRefs
      Observed Generation:   10
      Reason:                Invalid
      Status:                False
      Type:                  Programmed
    Name:                    https
```

That happened because in my initial listener setup, I pointed the secret to the `traefik` namespace, but never added it to the manifest. We need to add the `certificateRefs` array under `spec.listeners.[*].tls.certificateRefs`:

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: test-nginx-gateway
  namespace: default
spec:
  gatewayClassName: traefik
  listeners:
  [...]
  - name: https
    protocol: HTTPS
    port: 443
    tls:
      mode: Terminate
      certificateRefs:
        - kind: Secret
          name: local-selfsigned-tls
          namespace: traefik
```

Even with this change, though, we get the following `ReferenceGrant` error when we deploy the manifest.

```console
[main-pc]$ k describe gateway test-nginx-gateway
[...]
Conditions:
      Last Transition Time:  2026-07-29T15:49:51Z
      Message:               Cannot reference CertificateRef traefik/local-selfsigned-tls: missing ReferenceGrant
      Observed Generation:   11
      Reason:                RefNotPermitted
      Status:                False
      Type:                  ResolvedRefs
      Last Transition Time:  2026-07-29T15:49:51Z
      Message:               Invalid CertificateRefs
      Observed Generation:   11
      Reason:                Invalid
      Status:                False
      Type:                  Programmed
    Name:                    https
```

This happened because the `test-nginx-gateway` Gateway was trying to reach across namespaces to get the `local-selfsigned-tls` secret, from `default` to `traefik`. ReferenceGrants are the solution to this - they grant access for certain resources to reach across namespaces, such as an `HTTPRoute` reaching a `Service` in a different namespace. We need to set up a grant so that it can reach the `local-selfsigned-tls` secret to do the TLS termination - but doing so gives access to _all_ secrets in the namespace. Let's pivot and limit our scope by creating a new namespace, migrating our keys to it, and then giving the grant to _that_ instead of the full set of `kube-system` secrets:

```console
[main-pc]$ k create ns certs
namespace/certs created
[main-pc]$ k delete secret/local-selfsigned-tls -n kube-system
secret/local-selfsigned-tls deleted
[controlplane]$ kubectl create secret tls local-selfsigned-tls --cert=tls.crt --key=tls.key --namespace certs
secret/local-selfsigned-tls created
```

Now, we need to change the `traefik-config.yml` Helm configuration to point our `Secret` to the newly-created `certs` namespace:

```yaml
# /var/lib/rancher/k3s/server/manifests/traefik-config.yaml
apiVersion: helm.cattle.io/v1
kind: HelmChartConfig
metadata:
  name: traefik
  namespace: kube-system
spec:
  valuesContent: |-
[...]
        websecure:
          port: 443
          protocol: HTTPS
          namespacePolicy:
            from: All
          mode: Terminate
          certificateRefs:
            - kind: Secret
              name: local-selfsigned-tls
              namespace: certs
              group: ""
```

And finally, we verify that the Helm configuration change completes successfully:

```console
[main-pc]$ k logs helm-install-traefik-w9vsx -n kube-system
[...]
Release "traefik" has been upgraded. Happy Helming!
NAME: traefik
LAST DEPLOYED: Wed Jul 29 16:41:26 2026
NAMESPACE: kube-system
STATUS: deployed
REVISION: 5
DESCRIPTION: Upgrade complete
TEST SUITE: None
NOTES:
traefik with docker.io/rancher/mirrored-library-traefik:3.7.4 has been deployed successfully on kube-system namespace!
```

Now that the `Secret` is pointing to the correct place, we need to create the `ReferenceGrant`. Update one of our manifests or create a new one that includes a new `ReferenceGrant` object. Don't forget to change the Gateway TLS listener to use the `certs` namespace:
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: test-nginx-gateway
  namespace: default
spec:
  gatewayClassName: traefik
  listeners:
[...]
  - name: https
    protocol: HTTPS
    port: 443
    tls:
      mode: Terminate
      certificateRefs:
        - kind: Secret
          name: local-selfsigned-tls
          namespace: certs
    allowedRoutes:
      namespaces:
        from: Same
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
[...]
---
apiVersion: gateway.networking.k8s.io/v1
kind: TLSRoute
[...]
---
apiVersion: gateway.networking.k8s.io/v1
kind: ReferenceGrant
metadata:
  name: tls-cert-grant
  namespace: certs
spec:
  from:
  - group: gateway.networking.k8s.io
    kind: Gateway
    namespace: default
  to:
  - group: ""
    kind: Secret
    name: local-selfsigned-tls
```

Make sure that the grant is attached to the _gateway_, not the TLSRoute - I sat on that mistake for a bit, wondering why it wouldn't take. If you do that, you'll repeatedly see the `Cannot reference CertificateRef certs/local-selfsigned-tls: missing ReferenceGrant` error sticking out in the `kubectl describe` commands, regardless of your creation of a `ReferenceGrant`. Once you set it right in the manifest and deploy, this is how the `TLSRoute` on the Gateway should look:

```console
[main-pc]$ k describe gateway test-nginx-gateway
[...]
Conditions:
      Last Transition Time:  2026-07-29T16:48:03Z
      Message:               No error found
      Observed Generation:   12
      Reason:                Accepted
      Status:                True
      Type:                  Accepted
      Last Transition Time:  2026-07-29T16:48:03Z
      Message:               No error found
      Observed Generation:   12
      Reason:                ResolvedRefs
      Status:                True
      Type:                  ResolvedRefs
      Last Transition Time:  2026-07-29T16:48:03Z
      Message:               No error found
      Observed Generation:   12
      Reason:                Programmed
      Status:                True
      Type:                  Programmed
    Name:                    https
```

Now, we just need to test that the service is terminating TLS correctly - `-k` is used to ignore self-signed certs with `curl`:
```
[main-pc]$ curl -k https://testnode.jlegrange.dev
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
[...]
</html>
```

I've never been so happy to see the default Nginx page.

## Moving On

If you've stuck with me this far, thank you - it was a long ride, and a longer read. I'm looking forward to more posts in this vein - I have plans to deploy more than just a test app to my homelab, after all. Eventually I'll have services, metrics and logging, and a ton of other things - lots of ideas in the pipeline. Until then, I'll see you around!