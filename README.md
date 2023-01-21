# dapr-rbac

The files here represent a mutated version of the Dapr RBAC resources for K8s. The changes here include convering all cluster roles to roles, with the exception of the following:

### Dapr Injector Cluster Role

```yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dapr-injector
  labels:
    {{- range $key, $value := .Values.global.k8sLabels }}
    {{ $key }}: {{ tpl $value $ }}
    {{- end }}
rules:
  - apiGroups: [""]
    resources: ["serviceaccounts"]
    verbs: ["list"]
```

This cluster role binding is used for the Dapr sidecar injector to identify that the Kubernetes API server is the one requesting sidecar injections and not a malicious actor in the network. Cluster role is required because the service account identifier that is used by the Kubernetes API server resides in the `kube-system` namespace, along with a list of other allowed well known service accounts. This cluster role is widely used in many mutating webhooks, and in Dapr's case, taking this approach of allowing pre-approved and well known service accounts is more secure than webhooks that allow any injection.

### Dapr Sentry Cluster Role

```yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dapr-sentry
  labels:
    {{- range $key, $value := .Values.global.k8sLabels }}
    {{ $key }}: {{ tpl $value $ }}
    {{- end }}
rules:
  - apiGroups: ["authentication.k8s.io"]
    resources: ["tokenreviews"]
    verbs: ["create"]
```

This cluster role is used for the Dapr certificate authority to verify that Dapr sidecars requesting a workload identity are coming from Pods residing in the cluster. The permission given here is to create a token review request from the Kubernetes API server (verifying the Pod token) and doesn't allow for anything other than requesting an authentication request. It is extremely unlikely this cluster role can be abused for malicious purposes.
