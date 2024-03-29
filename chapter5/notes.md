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







 
