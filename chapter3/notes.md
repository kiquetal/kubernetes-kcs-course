### Service Account.

Needs to be applied using a rolebinding (and a role)


### Points to remember.

ServiceAccount has only the necessary RBAC permissions.

### Exam tips

Examine existing RoleBindings and ClusterRoleBindings to determine what permissions a service account has.

Design your RBAC setup in such a way that service accounts don't have unnecessary permissios

You can bind multiple roles to an account. Use this to keep Roles separate rather than overloading
them with a lot of permissions

You can bind a ClusterROle with a RoleBinding to provide the necessary perissions only within RoleBinding namespaces


### Restricting Service Account Permissions

```
apiVersion: v1
kind: Namespace
metadata:
  name: sa-permissions-test

---

apiVersion: v1
kind: ServiceAccount
metadata:
  name: shared-sa
  namespace: sa-permissions-test
automountServiceAccountToken: true

---

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: shared-sa-role
  namespace: sa-permissions-test
rules:
- apiGroups: [""]
  resources: ["deployments", "pods", "secrets"]
  verbs: ["get", "list"]

---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: shared-sa-rb
  namespace: sa-permissions-test
subjects:
- kind: ServiceAccount
  name: shared-sa
  namespace: auth
roleRef:
  kind: Role
  name: shared-sa-role
  apiGroup: rbac.authorization.k8s.io

---

apiVersion: v1
kind: Pod
metadata:
  name: deployment-viewer-pod
  namespace: sa-permissions-test
spec:
  serviceAccountName: shared-sa
  containers:
  - name: busybox
    image: radial/busyboxplus:curl
    command: ['sh', '-c', 'TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token); while true; do if curl -s -o /dev/null -k -m 3 -H "Authorization: Bearer $TOKEN" https://kubernetes.default.svc.cluster.local/api/v1/namespaces/auth/deployments/; then echo "[SUCCESS] Successfully viewed Deployments!"; else echo "[FAIL] Failed to view Deployments!"; fi; sleep 5; done']

---

apiVersion: v1
kind: Pod
metadata:
  name: pod-viewer-pod
  namespace: sa-permissions-test
spec:
  serviceAccountName: shared-sa
  containers:
  - name: busybox
    image: radial/busyboxplus:curl
    command: ['sh', '-c', 'TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token); while true; do if curl -s -o /dev/null -k -m 3 -H "Authorization: Bearer $TOKEN" https://kubernetes.default.svc.cluster.local/api/v1/namespaces/auth/pods/; then echo "[SUCCESS] Successfully viewed Pods!"; else echo "[FAIL] Failed to view Pods!"; fi; sleep 5; done']

```
### Lab- Limit Service AccountPermission

Create a Service Account with Appropriate Permissions for the pod-watch Pod
The pod-watch Pod in the auth namespace needs to be able to get and list Pods and Pod logs in the auth namespace.
Create a service account called pod-monitor and assign it appropriate RBAC permissions for the pod-watch Pod.
Modify the pod-watch Pod to use the new service account. Note that you will need to delete and re-create the Pod to do this. A manifest for the Pod can be found in /home/cloud_user.

1- Create Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-and-log-reader
rules:
- apiGroups: [""]
  resources: ["pods", "pods/log"]
  verbs: ["get", "watch", "list"]
```
2- Create SA

```yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: pod-monitor
  namespace: auth
automountServiceAccountToken: true
```

3- Creare RoleBinding

```yaml

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rb-sa-svc-monitor
  namespace: auth
subjects:
- kind: ServiceAccount
  name: sa-svc-monitor
  namespace: auth
roleRef:
  kind: Role
  name: role-svc-monitor
  apiGroup: rbac.authorization.k8s.io

```

4- Update pod

```yaml

apiVersion: v1
kind: Pod
metadata:
  name: pod-watch
  namespace: auth
spec:
  serviceAccountName: pod-monitor
  containers:
  - name: busybox
    image: radial/busyboxplus:curl
    command: ['sh', '-c', 'TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token); while true; do if curl -s -o /dev/null -k -m 3 -H "Authorization: Bearer $TOKEN" https://kubernetes.default.svc.cluster.local/api/v1/namespaces/auth/pods/; then echo "[SUCCESS] Successfully viewed Pods!"; else echo "[FAIL] Failed to view Pods!"; fi; sleep 5; done']

```                                                                                                                                                                                             
                                                                                                                                                                                             
### Restricting access to K8S Apis

- Prohibits access to cluster from outside.

~                                                                                                                                                                                             
~                                                                                                                                                                                             
~                                                                                                                                                                                             
~                                                                                                                                                                                          
