## Task 1

- Create a Deployment named web-app:
	-	Replicas: 2
	-	Image: nginx:1.27
	-	Container port: 80
	-	Label: app=web-app
-	Create a ClusterIP Service named web-app-svc that targets the web-app Pods on port 80.
-	Create a NodePort Service named web-app-nodeport that:
	-	targets the same Pods,
	-	exposes port 30080 on every node in the cluster.
-	(If your cluster has an Ingress Controller) Create an Ingress resource:
	-	host: demo.local
	-	route path / to web-app-svc.