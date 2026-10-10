# Concepts

## Pod

- Smallest deployable unit. Wraps one or more containers that share network and storage.
- A pod is not a container. It is a wrapper around one or more containers.
- A standalone pod that is deleted is not recreated. Controllers handle that.
- Phases: `Pending`, `Running`, `Succeeded`, `Failed`, `Unknown`.

## Manifest

A YAML description of the object you want. Four top-level fields appear in almost every manifest:

| Field | Meaning |
|-------|---------|
| `apiVersion` | Which API group and version the object belongs to |
| `kind` | The type of object (Pod, ReplicaSet, ...) |
| `metadata` | Name, labels, annotations, namespace |
| `spec` | The desired state |

## Imperative vs declarative

| Style | Example | Good for |
|-------|---------|----------|
| Imperative | `kubectl run nginx --image=nginx` | Quick tests, generating a starter YAML |
| Declarative | `kubectl apply -f pod.yaml` | Anything you want to repeat, review or keep in Git |

## Labels

- Key-value pairs on objects, used for **selecting and grouping**.
- Queried with selectors, e.g. `-l app=web`.
- ReplicaSets (and later Services) find their pods through labels.

## Annotations

- Key-value metadata for **information**, not selection.
- Examples: owner, build number, links to docs, tool configuration.
- Cannot be used in selectors.

| | Labels | Annotations |
|---|--------|-------------|
| Used for selection | Yes | No |
| Typical content | `app`, `env`, `tier` | Descriptions, owners, URLs |

## ReplicaSet

- Keeps a set number of identical pod replicas running.
- Three key parts of the spec: `replicas`, `selector`, `template`.
- `selector.matchLabels` must match `template.metadata.labels`.
- Recreates pods that are deleted or crash, and removes extras if there are too many.
- Usually not created directly. A Deployment manages ReplicaSets and adds rolling updates.

## Desired state vs current state

- You declare the **desired** state (e.g. 3 replicas).
- A controller continuously compares it to the **current** state and acts to close the gap.
- This reconcile loop is what makes self-healing work.


## Deployment

- Manages ReplicaSets for you and adds rolling updates and rollbacks on top.
- Hierarchy: Deployment -> ReplicaSet -> Pods. You edit the Deployment, and it creates a new ReplicaSet for each change to the pod template.
- Old ReplicaSets are kept (scaled to 0) so you can roll back to them.
- Use a Deployment for any stateless app. Creating bare ReplicaSets or pods is rare.

| | ReplicaSet | Deployment |
|---|-----------|------------|
| Keeps N replicas running | Yes | Yes (via its ReplicaSet) |
| Rolling updates | No | Yes |
| Rollback | No | Yes |
| Typical use | Managed by a Deployment | What you create directly |

## Scaling

- Change the replica count: `kubectl scale` or edit `spec.replicas` and re-apply.
- Scaling does **not** create a new ReplicaSet or a new revision. Only a pod template change does.
- If you scale by command and later re-apply a manifest with a different `replicas` value, the manifest wins.
- Scaling up can leave pods `Pending` if the cluster has no capacity.

## Rolling updates

- Triggered when `spec.template` changes (image, env, labels on the template, etc.).
- The Deployment creates a new ReplicaSet and gradually moves pods from old to new, so the app stays available.
- Two knobs control the pace:

| Setting | Meaning |
|---------|---------|
| `maxSurge` | How many extra pods above the desired count may exist during the update |
| `maxUnavailable` | How many pods below the desired count may be unavailable during the update |

- A **readiness probe** is what tells Kubernetes a new pod is safe to receive traffic. Without one, a broken new version can be rolled out as if it were healthy.
- Other strategy: `Recreate` kills all old pods first, then starts new ones (causes downtime).

## Rollbacks

- Each template change creates a revision, tracked in the rollout history.
- `kubectl rollout undo` goes back to the previous revision, or to a specific one with `--to-revision`.
- A rollback restores the **pod template** only. A replica count you changed with `scale` is not rolled back.
- `revisionHistoryLimit` controls how many old ReplicaSets are kept (default 10).
- The `kubernetes.io/change-cause` annotation is what appears in `rollout history`, so set it to record why a change was made.

## Service

- A stable network endpoint (fixed IP and DNS name) in front of a changing set of pods.
- Pods get new IPs whenever they are recreated. A Service gives clients one address that doesn't change.
- Finds its pods through a **label selector**, the same mechanism as ReplicaSets.
- Only pods that are **Ready** receive traffic. The list of ready pod IPs is in the Service's Endpoints (EndpointSlices).
- Gets a DNS name: `<service>.<namespace>.svc.cluster.local`, or just `<service>` from within the same namespace.

| Port field | Meaning |
|------------|---------|
| `port` | Port the Service listens on |
| `targetPort` | Port on the pod/container that traffic is forwarded to |
| `nodePort` | Port opened on every node (NodePort and LoadBalancer types only) |

## ClusterIP

- Default Service type.
- Gives the Service an internal virtual IP reachable **only from inside the cluster**.
- Use it for pod-to-pod communication, such as a frontend calling a backend or an app calling a database service.
- To test from your own machine, use `kubectl port-forward`.

## Service types

| Type | Reachable from | How | Typical use |
|------|----------------|-----|-------------|
| `ClusterIP` | Inside the cluster | Internal virtual IP | Internal services |
| `NodePort` | Outside, via any node's IP | Opens a port (30000-32767) on every node, on top of ClusterIP | Quick external access, dev/test |
| `LoadBalancer` | Outside, via an external IP | Asks the cloud provider for a load balancer, on top of NodePort | Production external access on cloud |
| `ExternalName` | Inside the cluster | DNS CNAME to an external hostname, no proxying | Pointing in-cluster apps at an external service |

- Each type builds on the previous: LoadBalancer includes NodePort, which includes ClusterIP.
- On a local kind cluster, `LoadBalancer` stays `<pending>` unless you add something like MetalLB or cloud-provider-kind.