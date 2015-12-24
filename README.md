# kubernetes-up
The goal of this project is to provide a minimalistic script which fires up a
single node [Kubernetes](http://kubernetes.io) cluster on the local machine with
no requirements besides [Docker](http://docker.io).


##Status

Trying to get this actually working by moving skydns and kube2sky into docker rather than
running under kuberentes.
If there is no dns server running locally, this might just work.

Last run:

Failed to run the docker skydns bits

Manually ran them

Found that port 53 was already bound by dnsmasq

Manually killed it


## Prerequisites
The only prerequisite for running this is Docker. You *do not* need any
Kubernetes binaries besides what's provided in this repository.

For the skydns availability check you additionally need ``nslookup``.

The script was tested with Ubuntu 14.10, Mint 17.1 ('Rebecca') as well as with Docker 1.6.2 and Docker 1.7.0

## Example
To fire up a single-node Kubernetes cluster on your local machine all you need to do is call the ``kube-up.sh`` script:
```
$ ./kube-up.sh
Starting etcd   OK
Starting k8s    OK
Starting proxy  OK
Waiting for API OK
Starting skydns OK
```

Once this is done you can launch pods and services:
```
$ ./kubectl create -f example/nginx.service.yaml
services/nginx
$ ./kubectl create -f example/nginx.pod.yaml
pods/nginx
```

Check that the pod was created and entered the ``Running`` state. This might
take several minutes the first time because Docker has to download the images.
```
$ ./kubectl get pods
POD              IP            CONTAINER(S)         IMAGE(S)                                         HOST                  LABELS                                                              STATUS    CREATED
k8s-master-127                 controller-manager   gcr.io/google_containers/hyperkube:v0.14.1       127.0.0.1/127.0.0.1   <none>                                                              Running   5 minutes
                               apiserver            gcr.io/google_containers/hyperkube:v0.14.1                                                                                                           
                               scheduler            gcr.io/google_containers/hyperkube:v0.14.1                                                                                                           
kube-dns-q7x4h   172.17.0.20   etcd                 quay.io/coreos/etcd:v2.0.3                       127.0.0.1/127.0.0.1   k8s-app=kube-dns,kubernetes.io/cluster-service=true,name=kube-dns   Running   4 minutes
                               kube2sky             gcr.io/google_containers/kube2sky:1.2                                                                                                                
                               skydns               gcr.io/google_containers/skydns:2015-03-11-001                                                                                                       
nginx            172.17.0.21   nginx                nginx                                            127.0.0.1/127.0.0.1   name=nginx                                                          Running   3 minutes
```

Check that the service was created and also get its IP address:
```
$ ./kubectl get services
NAME            LABELS                                                              SELECTOR           IP           PORT(S)
kube-dns        k8s-app=kube-dns,kubernetes.io/cluster-service=true,name=kube-dns   k8s-app=kube-dns   10.0.0.10    53/UDP
kubernetes      component=apiserver,provider=kubernetes                             <none>             10.0.0.2     443/TCP
kubernetes-ro   component=apiserver,provider=kubernetes                             <none>             10.0.0.1     80/TCP
nginx           name=nginx                                                          name=nginx         10.0.0.169   80/TCP
```

If everything worked you should be able to access nginx at the service IP address (``10.0.0.169`` in this case):
```
$ curl 10.0.0.169
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
## Kubernetes version
The example currently uses Kubernetes 0.18.2.

## References
- This script is based on the instructions in the [Running kubernetes locally via Docker](https://github.com/GoogleCloudPlatform/kubernetes/blob/master/docs/getting-started-guides/docker.md) article.
- The skydns configuration originates from [DNS in Kubernetes](https://github.com/GoogleCloudPlatform/kubernetes/tree/master/cluster/addons/dns).
