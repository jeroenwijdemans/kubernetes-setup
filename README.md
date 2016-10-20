# Setup single machine 'cluster'

## Install kubeadm

Setup is easy, follow:  http://kubernetes.io/docs/getting-started-guides/kubeadm/

Be sure to take step 3 (allow pods to be scheduled on the master)

## systemd

To autostart kubernetes after reboot add this systemd file to /etc/systemd/system/kubelet.service
 
```
[Unit]
Description=Kubelet
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStartPre=/usr/bin/mkdir -p /etc/kubernetes/manifests
ExecStart=/usr/bin/kubelet \
  --api-servers=http://127.0.0.1:8080 \
  --allow-privileged=true \
  --config=/etc/kubernetes/manifests \
  --v=1
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```


## Change kubeapi port

The default port for the master is 443. But since we setup a single machine we want 
to use 443 to route to a ingress nginx controller.
 So we change the port to 8443
 
 Change this in 
 - /etc/kubernetes/admin.conf
 - /etc/kubernetes/kubelet.conf
 - /etc/kubernetes/manifests/kube-apiserver.json
 
then restart kubernetes (reboot for example) 


# Access cluster from outside

## create external user 

on the cluster create a user:

```
kubectl create sa jenkins
kubectl get secret
kubectl describe jenkins-token-qqzod
```

 - note down the token 
 - and fetch the ca from /etc/kubernetes/ssl/ca.crt

## set kubectl config

On your CI / local machine install kubectl:


```
apt-get install -y kubectl
```

see setup above.
Then add your context. Insert token, ca.crt location and the ip of the cluster.
(note that this config is already expecting the port on the cluster to run on a not default port 8443 - usually it would be 443)

```
kubectl config set-context default-context --cluster=chronos --user=jenkins
kubectl config set-credentials jenkins --token=ABCDACBD11121....andtherestofyourtoken
kubectl config set-cluster chronos --server=https://ip:8443 --certificate-authority=/path/to/ca.crt 
kubectl config set-context default-context --cluster=chronos --user=jenkins
kubectl config use-context default-context
kubectl config set contexts.default-context.namespace default
kubectl config view
```


## disable authorization 

See http://kubernetes.io/docs/admin/authorization/

Decided for now to disable authorization. On the cluster edit this file:

```
vi /etc/kubernetes/manifests/kube-apiserver.json
```

and add:

```
"--authorization-mode=AlwaysAllow"
```

in the command. The container will automatically restart

Now you should be able to access the cluster from another machine on the network.

## Test 

To test in your browser use this proxy :

```
kubectl proxy
```

this proxies the https connection via local kube/config user to the cluster. 

# Add nginx ingress

yaml files based on these examples: https://github.com/jetstack/kube-lego/tree/master/examples/nginx 

## execute

setup default backend and nginx ingress
run:

```
kubectl apply -f nginx-namespace.yaml

kubectl apply -f default-http-backend-svc.yaml
kubectl apply -f default-http-backend-deployment.yaml

kubectl apply -f nginx-configmap.yaml
kubectl apply -f nginx-svc.yaml
kubectl apply -f nginx-deployment.yaml
```

now everything that hits port 80 will get a 
```
default backend - 404
```

## SSL

as shown here: https://github.com/kubernetes/contrib/blob/master/ingress/controllers/nginx/configuration.md
we are using the ssl-redirect annotation in the ingress
ingress.kubernetes.io/ssl-redirect: true

