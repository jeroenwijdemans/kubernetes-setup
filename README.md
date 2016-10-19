# Setup single machine 'cluster'

Setup is easy, follow:  http://kubernetes.io/docs/getting-started-guides/kubeadm/

# Access cluster from outside

## create external user 

on the cluster create a user:

```
kubectl create sa jenkins
kubectl get secret
kubectl describe jenkins-token-qqzod
```

note down the token
fetch the ca from /etc/kubernetes/ssl/ca.crt

## set kubectl config

on your CI / local machine:

```
kubectl config set-context default-context --cluster=chronos --user=jenkins
kubectl config set-credentials jenkins --token=ABCDACBD11121....andtherestofyourtoken
kubectl config set-cluster chronos --server=https://ip:443 --certificate-authority=/path/to/ca.crt 
kubectl config set-context default-context --cluster=chronos --user=jenkins
kubectl config use-context default-context
kubectl config set contexts.default-context.namespace default
kubectl config view
```

## disable authorization 

See http://kubernetes.io/docs/admin/authorization/

Decided for now to disable authoization. On the cluster edit:

```
vi /etc/kubernetes/manifests/kube-apiserver.json
```

and add:

```
"--authorization-mode=AlwaysAllow"
```

in the command. The container will automatically restart

now you should be able to access the cluster from another machin on the network.

# Add nginx ingress

TODO
 