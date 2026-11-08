- First add the repo.
`helm repo add https://prometheus-community.github.io/helm-charts`
- List all helm charts from the repo
`helm search repo prometheus`
- Install the chart prometheus-stack ( latest 91.8.2 at the time of install )
`helm install prometheus-stack prometheus/kube-prometheus-stack -n ckad`
- Get the service port to prepare for the port forwarding.
`kubectl get svc prometheus-operated -n ckad`
- Port forward the 9090 port of the service
`kubectl port-forward service/prometheus-operated 8080:9090 -n ckad`
I had to install socat on the nodes i am using since the port forwarding failed due to the nodes not having this package.
- Uninstall the chart
`helm uninstall prometheus-stack -n ckad`
