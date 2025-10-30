## Task1 RBAC with minimal permissions
-	Create a namespace named security-lab.
-	Create a ServiceAccount called reader in the security-lab namespace.
    -	Create a Role in security-lab that allows only `get`, `list`, and `watch` on the pods resource.
-	Bind that role to the reader ServiceAccount.
-	Retrieve the token for this ServiceAccount and try to list Pods via the Kubernetes API.
-   Remove `get` and `list` from role and repet connection 

<details>
        <summary>Hint</summary>

```bash
kubectl create ns security-lab
kubectl create sa reader -n security-lab
```
        
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: security-lab
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: security-lab
subjects:
- kind: ServiceAccount
  name: reader
  namespace: security-lab
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

```bash
kubectl apply -f rbac.yaml

TOKEN=$(kubectl create token reader -n security-lab)
APISERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')
curl -k -H "Authorization: Bearer $TOKEN" \
  "$APISERVER/api/v1/namespaces/security-lab/pods"
```
</details>

## Task 2 Container hardening
- Create a Pod named locked-pod in the security-lab namespace.
- The container inside this Pod should:
    - use the busybox image,
    - run the command: `id && sleep 3600`
    - run as a `non-root user` (runAsUser: 1000),
    - have `privilege escalation disabled`,
    - have a `read-only filesystem`,
    - have no `additional Linux capabilities`.
- Exec into the Pod and check the output of the `id` command.\

<details>
        <summary>Hint</summary>

       
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: locked-pod
  namespace: security-lab
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "id; sleep 3600"]
    securityContext:
      runAsNonRoot: true
      runAsUser: 1000
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop: ["ALL"]
```

</details>

## Task 3 Network Policies

- In the security-lab namespace, create two Deployments:
    - backend (nginx listening on port 80)
    - frontend (a curl-based container that just sleeps)
    - Each Pod should have the label app=backend or app=frontend respectively.
- Create ClusterIP service to expose backend
- Create a NetworkPolicy in this namespace that:
    - allows traffic only from Pods with label app=frontend to Pods with label app=backend on TCP port 80,
    - denies all other ingress traffic to the backend.
- Add another deployment and try connect to backend

<details>
        <summary>Hint</summary>

       
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: security-lab
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: api
        image: nginx:1.27
        ports:
        - containerPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: security-lab
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: web
        image: curlimages/curl:8.11.1
        command: ["sh", "-c", "sleep 3600"]
```

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
  namespace: security-lab
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 80
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: security-lab
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80
```

```bash
FRONTEND_POD=$(kubectl get pod -n security-lab -l app=frontend -o jsonpath='{.items[0].metadata.name}')

kubectl exec -n security-lab $FRONTEND_POD -- curl -s backend-svc.security-lab.svc.cluster.local

kubectl apply -f allow-frontend-to-backend.yaml
```

</details>