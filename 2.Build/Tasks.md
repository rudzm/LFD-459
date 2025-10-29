1. Based on the provided files app.py, requirements.txt, and Dockerfile:

	•	Build the container image.

	•	Tag the image as myapp:v1.

    Run the container locally using Docker or Podman:

	•	The container should listen on port 8080.

	•	Verify that running curl http://localhost:8080

    Explain:

	•	Why the Dockerfile has two stages (builder and runtime).

	•	Why the container is run as a non-root user.

    <details>
        <summary>Hint</summary>

        docker build -t myapp:v1 .

        docker run --rm -p 8080:8080 myapp:v1

        curl http://localhost:8080
    </details>

2. Create a Deployment named build-demo:

	•	Number of replicas: 1

	•	The container uses the image myapp:v1

	•	The container listens on port 8080

	The Deployment must have:

	•	a readinessProbe of type HTTP GET on path /, port 8080,
	
    •	a livenessProbe of type HTTP GET on path /, port 8080.
	
    Watch the status of the Pod.
	
    •	Does it become READY 1/1?
	
    Answer:
	
    •	What is the difference between a readinessProbe and a livenessProbe?
	
    •	What happens if the livenessProbe starts returning an error?

    <details>
        <summary>Hint</summary>
        
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: build-demo
    spec:
      replicas: 1
      selector:
        matchLabels:
        app: build-demo
      template:
        metadata:
        labels:
            app: build-demo
        spec:
        containers:
        - name: app
            image: myapp:v1
            ports:
            - containerPort: 8080
            readinessProbe:
            httpGet:
                path: /
                port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
            livenessProbe:
            httpGet:
                path: /
                port: 8080
            initialDelaySeconds: 10
            periodSeconds: 10
    ```
    </details>