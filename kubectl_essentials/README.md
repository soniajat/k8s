# 02 - kubectl Essentials

## Quick Concepts

| Concept | What It Means |
|---|---|
| `get` | Lists resources in a short table. |
| `describe` | Shows detailed info and recent events for one resource. |
| Output format | `-o wide`, `-o yaml`, `-o json` change how much detail `get` shows. |
| `explain` | Built-in documentation for any resource and its fields. |
| Namespace | A logical partition of the cluster. Default is `default`. |

## Commands to Remember

| Task | Command |
|---|---|
| List a resource type | `kubectl get <resource>` |
| Extra columns (node, IP) | `kubectl get <resource> -o wide` |
| Full object as YAML / JSON | `kubectl get <resource> <name> -o yaml` / `-o json` |
| Detailed info and events | `kubectl describe <resource> <name>` |
| Docs for a resource | `kubectl explain pod` |
| Docs for a nested field | `kubectl explain pod.spec.containers` |
| List namespaces | `kubectl get namespaces` |
| Use a specific namespace | `kubectl get pods -n kube-system` |
| Use all namespaces | `kubectl get pods -A` |

## Key Takeaways

- `get` is for a quick view, `describe` is for troubleshooting (the Events section at the bottom shows why things fail).
- `-o yaml` shows the full live object, including status.
- `kubectl explain` saves you from searching docs while writing YAML.
- Without `-n`, kubectl only looks at the `default` namespace. Use `-A` to see everything.
- System components live in `kube-system`.