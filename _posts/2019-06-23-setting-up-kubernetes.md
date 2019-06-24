---
layout: post
category : article
title: "Setting up a flexible Kubernetes environment"
comments: true
tags : [kubernetes]
hidden: true
---

In this cloud-native era, one cannot miss the importance of Kubernetes. It has become a somewhat de facto standard to deploy containerized application in enterprise environments. But the knowledge of many is limited to the abstract principles of the platform, or they have played around with minikube or k3s. But what does it take to create a multi cluster environment which is also fairly easy to maintain.

After much playing around, getting great tips and tricks from [https://twitter.com/krishofmans](@krishofmans) and reading what he also demonstrated on the [https://tripled.io/blog](TripleD blog), I want to share with you a setup that I think is quite neat.

What am I going to set up is the following: a company (ACME) with a 2-cluster k8s environment (clean installs) which will be accessed through another k8s cluster which will act as a gateway and which will also provide the SSL certificated for all the endpoints that it exposes. The 2 main clusters, which I will call Asterix and Obelix from now on are not accessible directly through the internet. In other words. there is no direct line of access to those servers from the internet. Only the gateway, which I'll call Idefix, can be accessed through the internet. The person managing the DNS of the company (which is using Cloudflare for DNS) was also so kind to buy a new domainname `acme.wtf` and add a domain record for all the subdomains (`*.acme.wtf`) that points to that server.

![Kubernetes setup](/img/kubernetes-setup.png)

Asterix and Obelix contain all the business applications of ACME. These are the cluster that developers will deploy on. Why 2 clusters? Well, the mechanism that I'll show you is just as applicable when using namespaces, but separate cluster provide a bit more safety, for exaple Obelix being the testing cluster and Asterix the actual producton cluster. If someone really wets the bed and messes up on Obelix, Asterix will not be affected at all. That said, most of what I'll be showing is done on Idefix.

First we need to configure Idefix. There are 2 things that need to be installed there: an ingress gateway and cert manager. For an ingress gateway I'll be using the nginx ingress gatewsy, but there's nothing stopping you from using another gateway like Traefik. Installing this can be done through Helm or manually. If you're using Helm, you'll also need to install that on Idefix. Once the ingress is installed, you can go to any URL that ends wih `*.acme.wtf` and you will be greeted by the default 404 response of the ingress.

To install Helm on your cluster execute the following command:

```
helm init --history-max 200
```

To install the nginx ingress, execute the following Helm command:

```
helm install stable/nginx-ingress --name nginx-ingress --namespace default --set controller.replicaCount=2
```

The other thing we need is cert-manager. The purpose of cert-manager is to issue SSL certificates though Let's Encrypt (LE). This will ensure that all sites will be served with a valid SSL certificate when accessing them through HTTPS. You can install cert-manager through Helm as well. 

To install cert-manager, follow the step defined [here](https://docs.cert-manager.io/en/latest/getting-started/install/kubernetes.html)

We'll now configure cert-manager to create a wildcard certificate for `*.acme.wtf` (yes, LE provides **free** wildcard certificates!). The first thing we need to do is a create a secret for the Cloudflare API, which will be used by LE for the certificate creation. For this you'll need a Cloudflare API key, which can be contained through the web console of Cloudflare. Once you have to you need to create the following secret.

```
kubectl create secret generic cloudflare-api-key --from-literal=api-key.txt=<Cloudflare API key> -n cert-manager
```

Next up is to create issuers for Let's Encrypt. We'll be creating 2 issuers, one that connects to the staging API of LE and the other to the production API. If you're playing around, first always do your experiments with the staging environment of LE, as the production API has strict limits on how frequent you can call it.

The Kubernetes resources for both issuers on Idefix looks like this:

```yaml
apiVersion: certmanager.k8s.io/v1alpha1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: youremail@acme-corp.com
    privateKeySecretRef:
      name: letsencrypt-staging
    dns01:
      providers:
      - name: cf-dns
        cloudflare:
          email: youremail@acme-corp.com  # this must match your Cloudflare account email 
          apiKeySecretRef:
            name: cloudflare-api-key
            key: api-key.txt
---
apiVersion: certmanager.k8s.io/v1alpha1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: youremail@acme-corp.com
    privateKeySecretRef:
      name: letsencrypt-prod
    dns01:
      providers:
      - name: cf-dns
        cloudflare:
          email: youremail@acme-corp.com # this must match your Cloudflare account email 
          apiKeySecretRef:
            name: cloudflare-api-key
            key: api-key.txt
```

With this in place, we can now create a wildcard certificate for `*.acme.wtf` by creating the following resource on Idefix.

```yaml
apiVersion: certmanager.k8s.io/v1alpha1
kind: Certificate
metadata:
  name: acme-wtf-wildcard
spec:
  secretName: cert-server-wildcard
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  commonName: '*.acme.wtf'
  dnsNames:
  - '*.acme.wtf'
  acme:
    config:
    - dns01:
        provider: cf-dns
      domains:
      - '*.acme.wtf'
```

After a while, if you execute `kubectl describe certificate acme-wtf-wildcard` on Idefix, you'll see the certificate as issued corrected. With that, we no longer need to worry about any certificates.

Now we need to connect Idefix to Asterix and Obelix. Mind you, Asterix and Obelix can be in a completely different locations or even on different cluster providers, but for simplicity sake, let's assume the following IPs for the  clusters:
- `10.0.0.1` for Obelix
- `10.0.0.2` for Asterix

The first thing we need to do is make sure Idefix knows about Asterix and Obelix. To do this, we'll configure 2 external services in Kubernetes. External services allow you to expose arbitrary services as Kubernetes services. The resources for both clusters on Idefix look like this:

```
apiVersion: v1
kind: Service
metadata:
  name: asterix-cluster
spec:
  type: ExternalName
  externalName: 10.0.0.2
  ports:
  - name: http
    port: 80
---
apiVersion: v1
kind: Service
metadata:
  name: obelix-cluster
spec:
  type: ExternalName
  externalName: 10.0.0.1
  ports:
  - name: http
    port: 80
```

Now we can make 2 ingresses on Idefix to route traffic to Asterix and Obelix. In this case, everything `*.prod.acme.wtf` will go to Asterix and everything `*.test.acme.wtf` will go to Obelix.

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: asterix-cluster-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
    - hosts:
      - prod.acme.wtf
      secretName: acme-wtf-wildcard
  rules:
  - host: prod.acme.wtf
    http:
      paths:
      - path: /*
        backend:
          serviceName: asterix-cluster
          servicePort: 80
---
kind: Ingress
metadata:
  name: obelix-cluster-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
    - hosts:
      - test.acme.wtf
      secretName: acme-wtf-wildcard
  rules:
  - host: test.acme.wtf
    http:
      paths:
      - path: /*
        backend:
          serviceName: obelix-cluster
          servicePort: 80
```

And that's it for Idefix.

Let's now show you how easy it is to deploy something on one of cluster, say Obelix here. Both Asterix and Obelix need to have an ingress gateway installed, but you can use Helm there as well to help you with that. We'll deploy the echoserver container to be accessed through `https://echo.test.acme.wtf`. These are resources for Obelix:

```
apiVersion: v1
kind: Service
metadata:
  name: echo
  labels:
    app: echo
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
    name: http
  selector:
    app: echo
---
apiVersion: v1
kind: ReplicationController
metadata:
  name: echo
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: echo
    spec:
      containers:
      - name: echo
        image: k8s.gcr.io/echoserver:1.4
        ports:
        - containerPort: 8080
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: echo-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: echo.test.acme.wtf
    http:
      paths:
      - path: /*
        backend:
          serviceName: echo
          servicePort: 80
```

And that's it, the only 'special' thing about it is the ingress host rule.

Once you have this setup installed, you now have an environment that is flexible and secure. In real environment however you need to be aware that Idefix, the gateway Kubernetes, basically is a single point of failure. I highly recommend making sure this is a not a single node cluster. That said, it doesn't have heavy requirements, as its only role is to delegate traffic. 

Adding a new cluster to the mix, for example a staging cluster servicing `*.staging.acme.wtf` is just a matter of adding a service and ingress to Idefix. There's a lot that becomes possible with this setup: multi-provider setups (where you're using for example GKS and AKS that are exact mirrors) are easy. Say that you have a cluster on IP 212.1.1.1 on GKS and another that has IP 168.1.1.1 on AKS. You can create a service like this:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-multicluster-service
spec:
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
---
apiVersion: v1
kind: Endpoints
metadata:
  name: my-multicluster-service
subsets:
  - addresses:
      - ip: 212.1.1.1
      - ip: 168.1.1.1
    ports:
      - port: 80
---
```

Idefix will then round-robin the service across different Kubernetes providers!

