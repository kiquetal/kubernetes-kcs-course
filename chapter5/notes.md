### Minimizing microservices vulnerabilities

- securityContext
- podSecurityPolicies (deprecated)
- OPA Gatekeeper
- Secrets
- Runtime sandbox
- mtls and certificate


### SecurityContext

A portion of the pod and container specification to enforce security. 

```yaml

apiVersion: v1
kind: Pod
metadata:
  name: test-pod

spec:
  securityContext:
    runAsUser: 1000
  containers:
   - name: nginx
     image: nginx
     securityContext:
       runAsUser: 1000

```

***Check the API for difference between the pod level and the container level.***

### Governing Pod Configurations

This is deprecated!

- First activate PodSecurityPolicy --enable-admision-plugins

- Create the policy
 
```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: psp-nonpriv
spec:
  privileged: false
  runAsUser:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  volumes:
  - configMap
  - downloadAPI
  - emptyDir
  - persistentVolumeClaim
  - secret
  - projected

``` 


- Create the serviceaccount

```yaml


```


- ClusterRole

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cr-use-psp-psp-nonpriv
rules:
- apiGroups: ['policy']
  resources: ['podsecuritypolicies']
  verbs:     ['use']
  resourceNames:
  - psp-nonpriv

```

- Rolebinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rb-psp-test
  namespace: psp-test
roleRef:
  kind: ClusterRole
  name: cr-use-psp-psp-nonpriv
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: psp-test-sa
  namespace: psp-test

```

- Example pod

```yaml
apiVersion:v1
kind: Pod
metadata:
  name: pod-psp-test-priv
  namespace: psp-test
spec:
  serviceAccountName: psp-test-sa
  containers:
  - name: nginx
    image: nginx:1.14.2
    securityContext:
     privileged: true

```


### OPA Gaekeepr

Open Policy Agent Gatekeeper (OPA)

You can control:

- Image repos
- ResourceLimits
- Labels

### Secrets

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: YWRtaW4K
  password: Y2FudGZpbmRtZQo

```

kubectl get secret my-secret -o yaml >> db.yaml


 
