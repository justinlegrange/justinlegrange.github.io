---
title: 'Kubernetes Homelab: Setting Up Glance'
date: '2026-08-06' # YYYY-MM-DD
lastMod: '2026-08-06'
summary: "Desc Text."
draft: true
# series: ["Kubernetes Homelab"]
# series_order: 2
categories: ["kubernetes", "devops"]
tags: ["kubernetes", "glance", "volumes", "gateway api", "configmaps"]
---

## Intro

Placeholder.

## Choosing the First Service

From part 1 of the series, I set up a way to host and expose services with the Kubernetes Gateway API


### Storage Considerations

Do we need persistent storage?
- Not really - we'll need some volumes, but nothing persistent
- ConfigMaps for the config yml files

Checking the minimal Docker compose:
```yml
services:
  glance:
    container_name: glance
    image: glanceapp/glance
    restart: unless-stopped
    volumes:
      - ./config:/app/config
      - ./assets:/app/assets
      - /etc/localtime:/etc/localtime:ro
      # Optionally, also mount docker socket if you want to use the docker containers widget
      # - /var/run/docker.sock:/var/run/docker.sock:ro
    ports:
      - 8080:8080
    env_file: .env
```

Will need to mount the volumes needed
- configmap for the `/etc/localtime`
- cm for our glance configs
- cm for assets folder css

using the example yml here for testing: https://github.com/glanceapp/docker-compose-template/tree/main/root

easiest way is to make the files locally, then create configmap from them: `k create cm localtime-cm --from-file=/etc/localtime --dry-run=client -o yaml > glance/dev/dev-cm.yml`

Can also do this for the config files - easier to write out the YAML for our configurations and then import the directory into a configmap


## Setting Up a Base Manifest

Placeholder.

## It's Always DNS

So, I ran into a really annoying issue
- No pods could resolve anything _outside_ of the cluster by default

Digging into the logs shows a bunch of errors:

```console
❯ k logs -n dev glance-dev-deployment-b797b67d4-kl96p
2026/08/06 22:41:19 Starting server on :8080 (base-url: "", assets-path: "/app/assets")
2026/08/06 22:41:42 ERROR Failed to get RSS feed url=https://selfh.st/rss/ error="Get \"https://selfh.st/rss/\": context deadline exceeded (Client.Timeout exceeded while awaiting headers)"
2026/08/06 22:41:42 ERROR Failed to get RSS feed url=https://ciechanow.ski/atom.xml error="Get \"https://ciechanow.ski/atom.xml\": context deadline exceeded (Client.Timeout exceeded while awaiting headers)"
2026/08/06 22:41:42 ERROR Failed to get RSS feed url=https://www.joshwcomeau.com/rss.xml error="Get \"https://www.joshwcomeau.com/rss.xml\": context deadline exceeded (Client.Timeout exceeded while awaiting headers)"
2026/08/06 22:41:42 ERROR Failed to get RSS feed url=https://samwho.dev/rss.xml error="Get \"https://samwho.dev/rss.xml\": context deadline exceeded (Client.Timeout exceeded while awaiting headers)"
2026/08/06 22:41:42 ERROR Failed to get RSS feed url=https://ishadeed.com/feed.xml error="Get \"https://ishadeed.com/feed.xml\": context deadline exceeded (Client.Timeout exceeded while awaiting headers)"
[...]
```

I tried remoting into the `glance` pod, only to be greeted with:
```
/app# nslookup google.com nslookup: write to '<CLUSTER_IP_ADDRESS>': Connection refused ;; connection timed out; no servers could be reached
```

After some digging, found out I have to restart CoreDNS: `kubectl delete pod -n kube-system -l k8s-app=kube-dns`

Edited the CoreDNS resolver to point at my network Pihole instance instead of `/etc/resolv.conf` for queries outside the cluster - it works!