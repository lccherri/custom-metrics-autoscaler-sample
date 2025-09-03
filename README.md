# Custom Metrics Autoscaler Sample

This demo uses the OpenShift Custom Autoscaler Operator (upstream KEDA) to scale an application based on a custom metric provided by OpenShift Monitoring.

## Prerequisites
- OpenShift 4.x cluster with cluster-admin privileges
- OpenShift Custom Metrics Autoscaler Operator (CMA, based on KEDA) installed
- Access to OpenShift Monitoring (user-workload monitoring enabled)

---

## Deploying the Demo

### 1. Deploy CMA Configuration

The `01-cma-config` folder contains the configuration resources required by the Custom Metrics Autoscaler.

```bash
oc apply -f 01-cma-config/
```

This will create:
- **PersistentVolumeClaim (00-pvc.yaml)**: Storage for CMA components.
- **KedaController (01-kedacontroller.yaml)**: Deploys the KEDA controller.
- **ServiceAccount (02-serviceaccount.yaml)**: SA used by CMA.
- **Secret (03-secret.yaml)**: Authentication data for the metrics source.
- **ClusterTriggerAuthentication (04-clustertriggerauthentication.yaml)**: Global authentication for trigger.
- **Role (05-role.yaml)**: Required RBAC permissions.

---

### 2. Deploy Sample Application

The `02-app-sample` folder contains the application and scaling configuration.

```bash
oc apply -f 02-app-sample/
```

This will create:
- **Namespace (00-traefik-ns.yaml)**: Dedicated namespace for the app.
- **Traefik Deployment (01-traefik.yaml)**: Sample application to be autoscaled.
- **Router (02-router.yaml)**: Route exposing the application.
- **Security Context Constraints (03-traefik-scc.yaml)**: SCC permissions for Traefik pods.
- **PodMonitor (04-podmonitor.yaml)**: Enables scraping metrics for Traefik pods.
- **ScaledObject (05-scaledobject.yaml)**: Defines the autoscaling rules using the custom metric.

---

### 3. Enable User Workload Monitoring

To allow Prometheus to scrape application metrics, enable **user workload monitoring**:

```bash
cat << EOF | oc create -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-monitoring-config
  namespace: openshift-monitoring
data:
  config.yaml: |
    enableUserWorkload: true
EOF
```

If the ConfigMap already exists, edit it instead and make sure the flag `enableUserWorkload` is set to `true`:

```bash
oc edit configmap cluster-monitoring-config -n openshift-monitoring
```

---

## Verify Deployment

1. Check that CMA pods are running:

```bash
oc get pods -n openshift-keda
```

2. Confirm application pods are deployed:

```bash
oc get pods -n traefik-sample
```

3. Check if `ScaledObject` is active and scaling:

```bash
oc describe scaledobject -n traefik-sample
```

---

## Cleanup

To remove the demo:

```bash
oc delete -f 02-app-sample/
oc delete -f 01-cma-config/
```