Elasticsearch on Google Kubernetes Engine (GKE)
This project migrates a local Elasticsearch setup from VirtualBox to a managed GKE Autopilot environment for better networking stability and scalability.

🚀 1. Cluster Setup
First, we initialize the GKE Autopilot cluster. This removes the need to manage individual VM nodes or networking bridges.

Bash
# Set your active project
gcloud config set project [YOUR_PROJECT_ID]

# Create the Autopilot cluster
gcloud container clusters create-auto elastic-cluster --region us-central1

# Get credentials to point kubectl to the cloud
gcloud container clusters get-credentials elastic-cluster --region us-central1
🔐 2. Security & Credentials
We use Kubernetes Secrets to store sensitive credentials. This ensures passwords are not hardcoded in YAML files.

Bash
# Create the secret (Change the password below!)
kubectl create secret generic elastic-auth \
  --from-literal=username=elastic \
  --from-literal=password=YourSuperSecurePassword123
📦 3. Deployment
The deployment uses a StatefulSet to ensure data persistence and a LoadBalancer service to provide a public entry point.

Apply the configuration:

Bash
kubectl apply -f elastic.yml
Wait for the External IP:

Bash
kubectl get svc elasticsearch-svc -w
🌐 4. Accessing the Cluster
Once the EXTERNAL-IP is assigned, access the cluster using Basic Authentication. Note that the default superuser is elastic.

Bash
# Export the IP for convenience
export ELASTIC_IP=$(kubectl get svc elasticsearch-svc -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

# Test the connection
curl -u elastic:YourSuperSecurePassword123 http://$ELASTIC_IP:9200
🛠 5. Maintenance Tasks
Updating the Password
To update the password without breaking the cluster, you must update both the internal Elasticsearch database and the Kubernetes Secret.

Update via API:

Bash
curl -u elastic:OLD_PASS -X POST "http://$ELASTIC_IP:9200/_security/user/elastic/_password" \
-H 'Content-Type: application/json' \
-d '{"password" : "NEW_PASSWORD"}'
Sync the Secret:

Bash
kubectl delete secret elastic-auth
kubectl create secret generic elastic-auth --from-literal=username=elastic --from-literal=password=NEW_PASSWORD
Troubleshooting
Check Pod logs: kubectl logs -f statefulset/elasticsearch

Restart Pods: kubectl rollout restart statefulset elasticsearch

Check Environment Variables: kubectl exec -it elasticsearch-0 -- env | grep xpack
