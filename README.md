# Setup single machine 'cluster'

## Install kubeadm

Setup is easy, follow:  http://kubernetes.io/docs/getting-started-guides/kubeadm/

Be sure to take step 3 (allow pods to be scheduled on the master)

## Change kubeapi port

The default port for the master is 443. But since we setup a single machine we want 
to use 443 to route to a ingress nginx controller.
 So we change the port to 8443

## systemd

add
 
```
cp kubelet.service /etc/systemd/system/kubelet.service
```

to autostart kubernetes after reboot

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

TODO
 