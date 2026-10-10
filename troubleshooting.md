# Troubleshooting

## Debugging flow

1. `kubectl get pods` - what is the status?
2. `kubectl describe pod <name>` - read the **Events** section at the bottom
3. `kubectl logs <name>` - what did the application say? (`--previous` if it restarted)
4. `kubectl exec -it <name> -- /bin/sh` - check config, files and connectivity from inside

## Symptom -> cause -> fix

| Symptom | Likely cause | How to check | Fix |
|---------|--------------|--------------|-----|
| `Pending` | No node can schedule it (resources, taints) or image still pulling | `describe pod` -> Events | Free up resources, fix node selectors, or wait for the pull |
| `ImagePullBackOff` / `ErrImagePull` | Wrong image name or tag, or no registry access | `describe pod` -> Events | Correct the image name or tag, check registry credentials |
| `CrashLoopBackOff` | Container starts and keeps crashing | `logs <pod> --previous` | Fix the app error, command, or config; check exit code in `describe` |
| `Running` but not working | App-level problem, not a Kubernetes one | `logs`, then `exec` | Fix app config or dependencies |
| `Error from server (NotFound)` | Wrong name or wrong namespace | `get pods -A` | Add `-n <namespace>` |
| Deleted pod came back | It is owned by a ReplicaSet (or Deployment) | `describe pod` -> Controlled By | Delete or scale the owner, not the pod |
| Deleted pod did not come back | Standalone pod, no controller | `get pods` | Recreate it, or use a controller next time |
| ReplicaSet rejected at apply: selector does not match template labels | `spec.selector.matchLabels` differs from `template.metadata.labels` | Compare the two blocks in the YAML | Make them match |
| ReplicaSet shows fewer pods than expected | Pods failing, or not enough capacity | `describe rs`, then `describe pod` | Fix the underlying pod problem |
| ReplicaSet "adopted" an existing pod | A standalone pod already had a matching label | `get pods --show-labels` | Use unique labels per workload |
| Pod no longer managed after relabel | Label no longer matches the selector, so the RS creates a replacement | `get pods --show-labels` | Restore the label, or delete the orphaned pod |
| `error converting YAML to JSON` | Indentation or syntax error | Check the line number in the error | Use spaces only, fix indentation |
| `no matches for kind` | Wrong `apiVersion` or typo in `kind` | `kubectl explain <kind>` | Use the correct `apiVersion` (ReplicaSet is `apps/v1`) |


## Deployments, rollouts and scaling

| Symptom | Likely cause | How to check | Fix |
|---------|--------------|--------------|-----|
| Rollout hangs at "Waiting for rollout to finish" | New pods are not becoming Ready (bad image, failing readiness probe, crash) | `rollout status`, then `get pods`, `describe pod`, `logs` on the new pods | Fix the cause, or `rollout undo` |
| Rollout fails with `ProgressDeadlineExceeded` | Update made no progress within `progressDeadlineSeconds` (default 600s) | `describe deploy` -> Conditions | Same as above. The Deployment does not roll back automatically |
| New pods `ImagePullBackOff` during update | Typo in image name or tag | `describe pod` -> Events | `rollout undo`, or fix the tag with `set image` |
| Old pods are still serving during a stuck update | Expected: `maxUnavailable` protects capacity while new pods fail | `get rs` shows old RS still holding pods | Fix or undo. The old version is still up |
| `field is immutable` on apply | `spec.selector` was changed after creation | Read the error message | Delete and recreate the Deployment, or restore the original selector |
| Deployment and a ReplicaSet fight over pods | Overlapping selectors (e.g. an old `app: web` ReplicaSet and a Deployment using the same label) | `get pods --show-labels`, `describe rs` | Use unique labels per workload, delete the stray ReplicaSet |
| `kubectl scale` worked, then replicas went back | A later `kubectl apply` of a manifest with a different `replicas` | Compare YAML `replicas` with the live value | Update `replicas` in the manifest too |
| Scaled up but pods stay `Pending` | Not enough CPU/memory on the nodes | `describe pod` -> Events (`Insufficient cpu`) | Lower resource requests, add nodes, or scale down |
| `rollout undo` did not restore replica count | A rollback only restores the pod template, not `replicas` | `get deploy` | Re-scale with `kubectl scale` |
| `rollout history` shows `<none>` under CHANGE-CAUSE | No `kubernetes.io/change-cause` annotation | `rollout history` | Annotate after each change |
| `rollout undo` says nothing to roll back to | Only one revision exists, or old ReplicaSets were pruned (`revisionHistoryLimit`) | `rollout history`, `get rs` | Redeploy the known-good version with `set image` |
| Update replaced a healthy version with a broken one that "looked fine" | No readiness probe, so Kubernetes treated a starting pod as ready | `describe deploy` -> pod template | Add a readiness probe |

Rollout debugging order: `rollout status` -> `get rs` (which ReplicaSet has the pods?) -> `describe pod` on a new pod -> `logs` -> decide: fix forward or `rollout undo`.

## Services

| Symptom | Likely cause | How to check | Fix |
|---------|--------------|--------------|-----|
| Service exists but requests fail, no endpoints | Selector does not match pod labels | `get endpoints <svc>` is empty, then compare `describe svc` Selector with `get pods --show-labels` | Fix the selector or the pod template labels |
| Endpoints exist but the pod isn't listed | Pod is not Ready | `get pods` (READY column), `describe pod` | Fix the readiness probe or the app. Unready pods are removed from endpoints |
| `Connection refused` through the Service | `targetPort` does not match the port the app listens on | `describe svc` (TargetPort), then `exec` into the pod and check the listening port | Correct `targetPort` |
| `Connection refused` only through the Service, direct pod works | Same as above, or wrong `port` used by the client | Test a pod directly with `exec` and `curl localhost` | Align `port`, `targetPort` and `containerPort` |
| Works by Service IP, fails by name | DNS problem or wrong namespace | Run `nslookup <svc>` from a temporary pod | Use `<svc>.<namespace>` from another namespace, check CoreDNS pods in `kube-system` |
| Can't reach ClusterIP from your laptop | Expected: ClusterIP is internal only | Check the Service type | Use `kubectl port-forward`, or switch to NodePort/LoadBalancer |
| Service sends traffic to old and new versions during an update | Expected: both versions are Ready during a rolling update | `get rs`, `get endpoints` | Tune `maxSurge`, `maxUnavailable`, or use `Recreate` |
| Wrong pods behind a Service | Another workload shares the same label | `get pods -l <selector> --show-labels` | Use unique labels |

### NodePort

| Symptom | Likely cause | How to check | Fix |
|---------|--------------|--------------|-----|
| `provided port is not in the valid range` | `nodePort` outside 30000-32767 | Read the error | Choose a port in range, or omit it |
| `provided port is already allocated` | Another Service already uses that nodePort | `get svc -A` | Use another port or omit |
| NodePort works inside the cluster but not from your machine (kind) | The node is a Docker container, so its IP is not reachable from the host (especially on Docker Desktop) | `get nodes -o wide` | Use `port-forward`, or create the kind cluster with `extraPortMappings` |
| Reachable on some nodes only | Firewall or security group blocks the port | Test each node | Open the NodePort range |

### LoadBalancer

| Symptom | Likely cause | How to check | Fix |
|---------|--------------|--------------|-----|
| `EXTERNAL-IP` stays `<pending>` | No cloud provider or load balancer implementation (always true on plain kind) | `describe svc` -> Events | Use MetalLB or cloud-provider-kind locally, or test with NodePort or port-forward |
| Load balancer created but unhealthy | Wrong `targetPort`, pods not Ready, or health check blocked | `get endpoints`, then the cloud console | Fix the port or probe |

### ExternalName

| Symptom | Likely cause | How to check | Fix |
|---------|--------------|--------------|-----|
| Name does not resolve | Typo in `externalName`, or no outbound DNS from the cluster | `nslookup <svc>` from a temporary pod | Correct the hostname, check egress and DNS |
| Connects but gets TLS errors | The client sends the Service name as the hostname, not the real one | Check the certificate error | Use the real hostname in the client config |

Service debugging order: `get svc` (type, ports) -> `get endpoints` (are pods listed?) -> `describe svc` (selector vs pod labels) -> test a pod directly (`exec` + `curl localhost`) -> test the Service from a temporary pod -> only then look at node, firewall or DNS.

## Useful checks 

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get pod <name> -o yaml | grep -A5 ownerReferences
kubectl api-resources
kubectl rollout status deploy/<name>
kubectl get deploy,rs,pods -l app=<label>
kubectl get endpoints <svc>
kubectl describe svc <svc> | grep -E "Selector|Port|Endpoints"
```