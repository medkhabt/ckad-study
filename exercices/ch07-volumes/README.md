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
