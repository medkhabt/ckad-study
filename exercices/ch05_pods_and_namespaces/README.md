- create a new Pod 
first create a new namespace 
```bash
kubectl create namespace ckad
```

```bash
kubectl run nginx --image=nginx:1.17.0 --port=80 -n ckad
``` 
or with the yaml file nginx.yaml. Remember, each resource has 4 sections in its 
descriptive yaml file. apiVersion, kind, metadata and spec.

- Get details of the pod including its IP address
``` bash
kubetl get pod nginx -o wide
```
the wide output shows also the ip address. 

- Create a temp pod to call the nginx reverse proxy 

``` bash
kubectl run busybox --image=busybox:1.36.1 --rm -it --restart=Never -- wget [ip address of nginx container]:80
```

- Get the logs of the nginx container
``` bash 
kubectl logs nginx -n ckad 
```
- Add env
    check the nginx yaml file 

- Open  shell and ls
``` bash
kubectl exec -it nginx -n ckad -- bash
ls -l
exit
```

- create yaml manifst for pod loop

``` bash
kubectl run loop --image=busybox.1.36.1 -n ckad --dry-run=client --restart=Never -o yaml  -- /bin/sh -c 'for i in 1 2 3 4 5 6 7 8 9 10; do echo "Welcome $i times"; done' > loop.yaml
```

-  edit command part 
can't edit the command part and apply -f . gotta delete the pod first and change they yaml file. check loop.yaml

- inspect events + status loop pod
``` bash
kubectl logs loop -n ckad
```
```  bash
kubectl get pods loop -n ckad
```


