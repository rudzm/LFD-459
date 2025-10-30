1. Create a Pod named design-demo.

	•	The Pod must have the following labels:

	    •	app=checkout
	    •	tier=backend
	    •	env=dev
	    •	team=workflows

	The Pod must include:

	•	an initContainer that writes a text file startup.txt containing information about the startup time,
	
    •	the main application container using the image nginx:1.27,
	
    •	an additional log-sidecar container that continuously displays (tails) the contents of that file.
	
    The main application container must receive configuration via environment variables:
	
    •	LOG_LEVEL from the app-config ConfigMap,
	
    •	DB_USER and DB_PASS from the db-secret Secret.
	
    Answer the following questions:
	
    •	What is the purpose of the initContainer?
	
    •	What is the purpose of the sidecar?
	
    •	Why isn’t the configuration baked directly into the image?

    <details>
        <summary>Hint</summary>
        
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: design-demo
  labels:
    app: checkout
    tier: backend
    env: dev
    team: workflows
spec:
  volumes:
  - name: shared-logs
    emptyDir: {}
  initContainers:
  - name: init-config
    image: busybox
    command: ["sh", "-c", "echo 'Boot at ' $(date) > /workdir/  startup.txt"]
    volumeMounts:
    - name: shared-logs
      mountPath: /workdir
  containers:
  - name: app
    image: nginx:1.27
    env:
    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: LOG_LEVEL
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: DB_USER
    - name: DB_PASS
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: DB_PASS
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/app
  - name: log-sidecar
    image: busybox
    command: ["sh", "-c", "tail -F /var/log/app/startup.txt"]
    volumeMounts:
      - name: shared-logs
        mountPath: /var/log/app
```
    </details>

2. Modify the app container definition in design-demo so that it:

	•	runs as a non-root user,

	•	cannot escalate privileges,

	•	has a read-only filesystem,

	•	has resource constraints (both requests and limits for CPU and memory).

    <details>
        <summary>Hint</summary>
        
    ```yaml
    securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    allowPrivilegeEscalation: false
    readOnlyRootFilesystem: true
    capabilities:
        drop: ["ALL"]
    resources:
    requests:
        cpu: "250m"
        memory: "256Mi"
    limits:
        cpu: "500m"
        memory: "512Mi"
    ```
    </details>