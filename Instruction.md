1. Deploying the Application to Kubernetes

To deploy the Todo App into Kubernetes, follow these steps:

Ensure you have a running Kubernetes cluster (kind, minikube, or a cloud provider).

Create a namespace for the app:

kubectl create namespace todoapp


Apply the deployment manifest:

kubectl apply -f deployment.yml


Apply the service manifest to expose the app:

kubectl apply -f service.yml


(Optional) Apply the HPA configuration:

kubectl apply -f hpa.yml

2. Resource Requests and Limits

In the deployment manifest, I used:

resources:
  requests:
    memory: "64Mi"
    cpu: "30m"
  limits:
    memory: "128Mi"
    cpu: "60m"

Rationale:

Requests represent the minimum guaranteed resources for the pod.

64Mi of memory and 30m of CPU are enough for a lightweight application like a simple Todo app.

Limits represent the maximum resources a pod can consume.

128Mi of memory and 60m CPU ensures that the pod doesn’t overconsume cluster resources while still being able to handle peak load.

This configuration balances efficiency and stability, preventing memory overcommitment or CPU starvation.

3. Horizontal Pod Autoscaler (HPA)

Example HPA config (hpa.yml):

apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: todoapp-hpa
  namespace: todoapp
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: todo-deployment
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70

Rationale:

minReplicas: 2 → ensures at least 2 pods are always available for redundancy.

maxReplicas: 5 → prevents the system from spawning too many pods and overloading the cluster.

CPU utilization at 70% → a balance between efficiency and responsiveness, so scaling happens before pods get overloaded.

4. Deployment Strategy

The deployment uses:

strategy:
  type: RollingUpdate

Rationale:

RollingUpdate → ensures zero downtime during updates.

Default settings replace pods gradually, so users always have access to the app while new pods roll out.

Combined with readiness probes, this ensures new pods are only added to the load balancer once they are healthy.

5. Accessing the Application

The app is exposed via a Service. Example (service.yml):

apiVersion: v1
kind: Service
metadata:
  name: todoapp-service
  namespace: todoapp
spec:
  type: NodePort
  selector:
    app: todo-deployment
  ports:
    - port: 80
      targetPort: 8080
      nodePort: 30007

Access:

Inside the cluster:

curl http://todoapp-service.todoapp.svc.cluster.local


From your machine (NodePort):

curl http://localhost:30007


(or use <node-ip>:30007 if not running locally).

✅ Once you add this file as INSTRUCTION.md, commit your changes and push to your branch:

git checkout -b add-instructions
git add INSTRUCTION.md
git commit -m "Add deployment instructions for Todo app"
git push origin add-instructions


Then create a Pull Request on your platform (GitHub/GitLab/Bitbucket).