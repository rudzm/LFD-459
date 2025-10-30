## Task 1	
    - Create a PersistentVolumeClaim named data-pvc with a size of 1Gi.
	
    -	Create a Deployment named config-demo:
        -	Replicas: 1
	    -	Container: nginx:1.27
	    -	Mount the following into the container:
	    -	data-pvc at /var/app/data
	    -	ConfigMap app-config at /etc/app/config (read-only)
	    -	Secret db-secret at /etc/app/secret (read-only)

	-	Exec into the running Pod and:
	    -	list the /etc/app/config directory,
	    -	list the /etc/app/secret directory,
	    -	try to create a file test.txt in /var/app/data.

	-	Scale the config-demo Deployment from 1 to 3 replicas.

	-	Observe the Pods after scaling.
	    -	Did all of them start correctly? If not, explain why.

    -   Remove Volume from deployment

	-	Perform a rolling update by changing the image to nginx:1.27-alpine, and then perform a rollback.

## Task 2

    - Modify the config-demo Deployment so that it:
	    -	uses the RollingUpdate strategy,
	    -	has maxUnavailable: 1,
	    -	has maxSurge: 1,
	    -	sets revisionHistoryLimit: 5.

    - Explain
	    -	Why limit maxUnavailable?
	    -	Why keep rollout history?

<details>
        <summary>Hint</summary>
        
```yaml
revisionHistoryLimit: 5
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1
    maxSurge: 1
```
</details>
