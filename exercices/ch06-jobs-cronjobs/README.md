- create a job + execute with two pods in parallel 5 compeletions 
``` bash
kubectl create job random-hash --image=alpine:1.17.3 -n ckad --dry-run=client -o yaml -- /bin/sh -c '$RANDOM | base64 | head -c 20' > random-hash.yaml 
```
to generate the yaml file. 
and change the yaml file by adding the `spec.completions` and `spec.parallelism`.

- identify the pods 
``` bash 
kubectl get pods -n ckad
```
check the pods with the suffix random-hash. 5 pods.
- delete job 
``` bash
kubectl delete job random-hash -n ckad
```
the pods are also deleted.

