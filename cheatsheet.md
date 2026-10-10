# kubectl Cheatsheet

Grouped by task, not by topic.

## Cluster and context

| Task | Command |
|------|---------|
| Create a kind cluster | `kind create cluster --name my-first-cluster` |
| List kind clusters | `kind get clusters` |
| Cluster info | `kubectl cluster-info` |
| List contexts | `kubectl config get-contexts` |
| Current context | `kubectl config current-context` |
| Switch context | `kubectl config use-context <context>` |
| List nodes | `kubectl get nodes` |
| Delete a kind cluster | `kind delete cluster --name my-first-cluster` |

## Inspect

| Task | Command |
|------|---------|
| List pods | `kubectl get pods` |
| More columns (node, IP) | `kubectl get pods -o wide` |
| Full object as YAML | `kubectl get pod <name> -o yaml` |
| Full object as JSON | `kubectl get pod <name> -o json` |
| All namespaces | `kubectl get pods -A` |
| Specific namespace | `kubectl get pods -n <namespace>` |
| Details and events | `kubectl describe pod <name>` |
| Field documentation | `kubectl explain pod.spec.containers` |
| Watch for changes | `kubectl get pods -w` |

## Debug

Order: `get` (status) -> `describe` (events) -> `logs` (app output) -> `exec` (look inside).

| Task | Command |
|------|---------|
| View logs | `kubectl logs <pod>` |
| Follow logs | `kubectl logs -f <pod>` |
| Logs of a specific container | `kubectl logs <pod> -c <container>` |
| Logs of the previous crashed container | `kubectl logs <pod> --previous` |
| Shell inside a pod | `kubectl exec -it <pod> -- /bin/sh` |
| Run a single command | `kubectl exec <pod> -- ls /` |

The `--` separates kubectl flags from the command to run inside the container.

## Create and apply

| Task | Command |
|------|---------|
| Quick pod (imperative) | `kubectl run nginx --image=nginx` |
| Generate YAML without creating | `kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml` |
| Create or update from file | `kubectl apply -f <file>.yaml` |
| Preview changes | `kubectl diff -f <file>.yaml` |
| Delete what a file created | `kubectl delete -f <file>.yaml` |
| Delete a pod | `kubectl delete pod <name>` |

## Labels and annotations

| Task | Command |
|------|---------|
| Show labels | `kubectl get pods --show-labels` |
| Filter by label | `kubectl get pods -l app=web` |
| Multiple conditions | `kubectl get pods -l app=web,env=dev` |
| Set-based filter | `kubectl get pods -l 'env in (dev,test)'` |
| Add a label | `kubectl label pod <name> env=dev` |
| Change a label | `kubectl label pod <name> env=prod --overwrite` |
| Remove a label | `kubectl label pod <name> env-` |
| Add an annotation | `kubectl annotate pod <name> owner="platform-team"` |
| See annotations | `kubectl describe pod <name>` |

## ReplicaSets and scaling

| Task | Command |
|------|---------|
| List ReplicaSets | `kubectl get rs` |
| Details and events | `kubectl describe rs <name>` |
| Scale | `kubectl scale rs <name> --replicas=5` |
| Delete RS and its pods | `kubectl delete rs <name>` |
| Delete RS, keep its pods | `kubectl delete rs <name> --cascade=orphan` |


## Deployments

| Task | Command |
|------|---------|
| Create a Deployment (imperative) | `kubectl create deployment web-deploy --image=nginx:1.25 --replicas=3` |
| Generate YAML without creating | `kubectl create deployment web-deploy --image=nginx:1.25 --dry-run=client -o yaml > deployment.yaml` |
| List Deployments | `kubectl get deploy` |
| See the whole hierarchy | `kubectl get deploy,rs,pods` |
| Details and events | `kubectl describe deploy web-deploy` |
| Delete a Deployment (and its RS and pods) | `kubectl delete deploy web-deploy` |

## Scaling

| Task | Command |
|------|---------|
| Scale a Deployment | `kubectl scale deploy web-deploy --replicas=5` |
| Scale to zero (stop pods, keep config) | `kubectl scale deploy web-deploy --replicas=0` |
| Scale from the manifest | edit `spec.replicas`, then `kubectl apply -f deployment.yaml` |
| Watch pods come up | `kubectl get pods -w` |

## Rolling updates and rollbacks

| Task | Command |
|------|---------|
| Update the image | `kubectl set image deploy/web-deploy nginx=nginx:1.26` |
| Update via manifest | change the image in YAML, then `kubectl apply -f deployment.yaml` |
| Record why it changed | `kubectl annotate deploy web-deploy kubernetes.io/change-cause="upgrade to 1.26"` |
| Watch the rollout | `kubectl rollout status deploy/web-deploy` |
| Rollout history | `kubectl rollout history deploy/web-deploy` |
| Inspect one revision | `kubectl rollout history deploy/web-deploy --revision=2` |
| Roll back to previous revision | `kubectl rollout undo deploy/web-deploy` |
| Roll back to a specific revision | `kubectl rollout undo deploy/web-deploy --to-revision=1` |
| Pause a rollout | `kubectl rollout pause deploy/web-deploy` |
| Resume a rollout | `kubectl rollout resume deploy/web-deploy` |
| Restart all pods (new rollout, same spec) | `kubectl rollout restart deploy/web-deploy` |
| See old and new ReplicaSets | `kubectl get rs -l app=web-deploy` |

## Services

| Task | Command |
|------|---------|
| Expose a Deployment (ClusterIP) | `kubectl expose deploy web-deploy --name=web-clusterip --port=80 --target-port=80` |
| Expose as NodePort | `kubectl expose deploy web-deploy --name=web-nodeport --port=80 --type=NodePort` |
| Expose as LoadBalancer | `kubectl expose deploy web-deploy --name=web-lb --port=80 --type=LoadBalancer` |
| List Services | `kubectl get svc` |
| More detail (selector column) | `kubectl get svc -o wide` |
| Details and events | `kubectl describe svc web-clusterip` |
| Which pods back the Service | `kubectl get endpoints web-clusterip` |
| Same, newer API | `kubectl get endpointslices -l kubernetes.io/service-name=web-clusterip` |
| Delete a Service | `kubectl delete svc web-clusterip` |

## Testing Services

| Task | Command |
|------|---------|
| Reach a Service from your machine | `kubectl port-forward svc/web-clusterip 8080:80` then `curl localhost:8080` |
| Test from inside the cluster | `kubectl run tmp --rm -it --image=busybox:1.36 --restart=Never -- wget -qO- http://web-clusterip` |
| Check DNS resolution | `kubectl run tmp --rm -it --image=busybox:1.36 --restart=Never -- nslookup web-clusterip` |
| Test one pod directly | `kubectl exec <pod> -- curl -s localhost:80` |
| NodePort node address (kind) | `kubectl get nodes -o wide` |