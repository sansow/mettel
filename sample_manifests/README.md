# CoolStore on OpenShift — GitOps Overlay

This folder is the **glue** that wraps the upstream Red Hat / community demos into one
coherent retail-on-OpenShift story for a 2 to 2.5 hour exec demo.

## Upstream sources we reuse (do not vendor — reference)

| Layer | Repo | What we use |
|------|------|-------------|
| App services | `redhat-mw-demos/quarkus-coolstore` (fallback: `quarkusio/quarkus-super-heroes`) | Storefront, Catalog, Cart, Order, Recommend |
| VM-on-OCP | `rhpds/openshift-virt-roadshow-cnv` | VirtualMachine pattern, DataVolume, MTV import |
| CI | `openshift/pipelines-tutorial` | Tekton ClusterTasks, build-deploy pipeline |
| CD | `redhat-developer/gitops-operator` | Argo CD on OpenShift |
| Observability | `rhobs/observability-operator` + `loki-operator` + `tempo-operator` | LGTM stack on cluster |
| Optional 2nd obs demo | `grafana/tns` | 3-tier app pre-instrumented with Loki/Tempo |
| Multi-cluster | RHACM (operator) | ApplicationSet / PolicyGenerator |
| Security | RHACS (operator) | Image scan, admission, runtime |

## Folder layout

```
sample_manifests/
  00-prereqs/         Operators we install first (GitOps, Pipelines, Virt, COO, Loki, Tempo, ACS, ACM)
  10-namespaces/      coolstore, coolstore-cicd, coolstore-obs, coolstore-virt
  20-vm-postgres/     Postgres VM (KubeVirt VirtualMachine + Service + Secret)
  30-pipelines/       Tekton pipeline that builds + signs + pushes to Quay
  40-apps/            Container Deployments for storefront/catalog/cart
  50-knative/         Knative Services for order + recommend
  60-observability/   COO MonitoringStack, ServiceMonitors, LokiStack, TempoStack, Grafana datasources
  70-acm-policies/    PolicyGenerator examples to push CoolStore + guardrails to N clusters
  argocd/             AppProject, ApplicationSet wiring everything via app-of-apps
```

## Apply order (manual, for the demo)

1. `oc apply -k 00-prereqs/`            # subscribes to all operators
2. wait for operators to install        # ~3-4 minutes
3. `oc apply -k 10-namespaces/`         # creates the four projects
4. `oc apply -k 20-vm-postgres/`        # spins up Postgres VM
5. `oc apply -k 60-observability/`      # stands up Prometheus/Loki/Tempo/Grafana
6. `oc apply -k argocd/`                # one ApplicationSet that drives everything else

## Apply order (the real way — for production)

```
oc apply -k argocd/
```

That's it. Argo CD reconciles 30/40/50/60/70 from Git. The whole demo is the bootstrap.
