# Pods and namespaces
# Jobs and Cron Jobs
## Clean up 
After the job is completed, kubernets does not automatically delete the job object, so one can debug and inspect the logs of the job pod. 
However, it is also recommended to auto clean up jobs to avoid cluster performance degradation. 

> It is recommended to set ttlSecondsAfterFinished field because unmanaged jobs (Jobs that you created directly,
> and not indirectly through other workload APIs such as CronJob) have a default deletion policy of orphanDependents causing 
> Pods created by an unmanaged Job to be left around after that Job is fully deleted. Even though the control plane eventually garbage 
> collects the Pods from a deleted Job after they either fail or complete, sometimes those lingering pods may cause cluster performance 
> degradation or in worst case cause the cluster to go offline due to this degradation.
>
> You can use LimitRanges and ResourceQuotas to place a cap on the amount of resources that a particular namespace can consume.

https://kubernetes.io/docs/concepts/workloads/controllers/job/

## Completion and parallelism
the default configuration for the job is for it run one the workload in one pod and expect for the pod to successfully complete.
That's called non-paralle Job.
One can change this default by assigning different values to `spec.completions` and `spec.parallelism`, the first one is the number 
of pods that are expected to complete successfully and the second is for specifying how many parallel executions of the workload must be run, in
other words how many pods that are part of the job must be run in parallel. 

## Restart Behavior
The default restart policy in k8s is `Always`, with this policy, the Kubernetes scheduler will restart the pod even if the container exit code is 0.
Which is not something wanted for a job. One needs to change the `spec.template.spec.restartPolicy`.

The second point is that kubernets scheduler will restart the resource object (in this case job ) a number amount of times before it will considered failing. By default it's 6.
One can also change this value in `spec.backoffLimit`. As you can see, this is not related to the pod specifically and it's more about the status of the resource
(spec -> .. , instead of spec -> template ..) 

For the restart policy, on can chose to restart on failure or also start a new pod on failure.

I just noticed that when using `kubectl create job` it will specify the restartPolicy to `Never`.

## Cronjobs
Job resource represent executions needed to reach the desired state, after reaching this state, the job lifecycle finishes. A cronjob create
job objects periodically. The cronjob manages the job objects while the jobs keep managing their pods.

the cronjobs and their jobs and their pods are easily matched, they all start with the label name of the cron, then the jobs label which is the cron time when it was created
then a hash for the pod. pod label: [cron label]-[job cron time]-[hash] 

One good thing about having a cron is that it proposes a better clean up policy to be able to inspect old job runs without affecting the performance of the cluster. 
By default it keeps the last three sucessful pods and the last failed pod. One can change this limit using `spec.fullJobsHistoryLimit` and `spec.failedJobsHistoryLimit`. 

The book says that there exists `spec.fullJobsHistoryLimit` but the current version of the CronJob resource has `spec.succesfulJobsHistoryLimit` and `spec.failedJobsHistoryLimit` instead.

### Jobs and restart behavior
I noticed that creating the cronjob object, it specifis that the container run on the job object has a restart policy of restart on failure.

## usefull commands
- `kubectl get jobs` 
- `kubectl create job [name] --image=[image] -- [command]` and i guess one can run it in dry-mode and generate the yaml file.
- `kubectl create cronjob [name] --schedule=[cron schedule]" --image=[image] -- [command]` so the only diff to job creationg is that it has a --schedule ( or at least for the simplest config )  

# Volumes
## Relevant Volume types
- emptyDir: empty directory in the pod with read/write access. Lifespan of volume = Lifespan of pod. 
    * usage:
        - cache implementations
        - data exchange between containres of the pod.
- hostPath: (Not meant for production!) file/dir from host node's filesystem. Supported only in single node clusters. 
    * usage: 
        - I guess it's good for ci.
- configMap, secret: 
    * usage: inject configuration data.
- nfs: Network file system share ( accessing a file on a remote machine over a network as if those files where on the local filesystem ), preserve data after pod restart.
- persistentVolumeClaim: Claims a persistent volume.
## Ephemeral volumes
Ephemeral Volume lifespan equals the Pod lifespan. Useful when we want shared memory between containers of a pod. 
During a restart of a pod, the data in the ephemeral volume is lost since it will create a new volume on the restart.

to configure one must:
- Add the volume to `spec.volumes[]`. You provide a name and type. 
- The volume needs to be mounted on a container to a specific path, configured via `spec.containers[].volumeMounts[]`( the name must match the volume name ).
## Persistent Volumes
K8s models persistent data with help of two primitives: `PersistentVolume` and `PersistentVolumeClaim`.
- PersistentVolume: represent in k8s the single storage resource. It describs the source of the storage. A storage can be assigned by mapping to a storage class (TODO: need more info here).
- PersistentVolumeClaim: Requests the resource of PersistentVolume. If I understand correctly it basically rents/claims the PersisentVolume resource so no other pod use it ?
One gotcha: the PersistentVolumeClaim Resource is namespace scoped ( not like PersistentVolume resource )

# Multi-container pods
to chose which container to log or to run exec on, one should specify the container name with the argument `-c/--container`.

# Deployments
## The difference between ReplicaSet and Deployment
In my current understanding, a ReplicatSet manages n identical pods with the same template ( desired state of the pods ) and same labels.
The Deployment is responsible for updating the ReplicatSet if the desired state changes or the labels changes. It creates a new ReplicaSet
object that manages the pods with the new template and label set.
## Rollout 
- `kubectl rollout status deployment []`
- `kubectl rollout history deployment []`
- To add a change cause for the revision, one can add annotation `kubernetes.io/change-cause` with a message. `kubectl annotate deployment backend kubernetes.io/change-cause="[]"` 
- `kubectl rollout undo deploymet [] --to-revision=[int]`
## ReplicaSets
one observation is that after a rollout update the old replicaset still exists in the namespace, just with no replicas up. I had 
the question if maybe just one old replica stays in the namespace, but after testing an other rollout update, i had two old replicasets
still existing (maybe there is a limit to this).
## Horizental Pod Autoscaler
HPA objects interact with the metrics server, so it should be installed on the cluster to benefit from the HPA features.
To do so 
`kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`
or via helm, i did it using helm 
```bash
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
helm upgrade --install metrics-server metrics-server/metrics-server
```
I had certificate aknowldegment issues ( TODO, fix the issue instead of dodging the issue, it has to do with the subject of the certifcate
presented by the kubelet and the private ip address that is used by the metrics-server to call the kubelet )
`helm upgrade metrics-server metrics-server/metrics-sever --set 'args[0]=--kubelet-insecure-tls'`
