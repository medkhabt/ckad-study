- create pod with two contains both mounting the same ephemeral volume
after creating the pod with `kubectl create -f [file]` opened an interactive shell session with the first container
```bash
kubectl exec -it two-containers -n ckad --container=one -- /bin/sh
echo "hello World" > /etc/a/hello.txt # container one mounts volume to /etc/a
cat /etc/a/hello.txt
exit
```
then i opened an interactive shell session with the second container
```bash
kubectl exec -it two-containers -n ckad --container=two -- /bin/sh
cat /etc/b/hello.txt # container two moutns volume to /ect/b 
exit
```

- create pv logs-pv, and check if its status is availabe
for checking one just need to run `kubectl get pvc`

- generate first yaml descriptive file for the nginx pod
```bash
kubectl run nginx -n ckad --image=nginx:1.25.1 --port=80 --dry-run=client -o yaml > nginx.yaml
```
then add the volumes and the mount part for the container. ( check the yaml file )

- test if the nginx reverse proxy work
```bash
kubectl run test --rm -it --restart=Never --image=busybox:1.36.1 -- wget 10.112.1.27:80
```
