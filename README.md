# Voting App Helm Chart.

This Helm chart deploys the Example Voting App microservices (PostgreSQL, Redis, Vote, Result, Worker) on Kubernetes.

## Usage

1. **Install Helm** (if not already installed):
   https://helm.sh/docs/intro/install/

2. **Deploy the chart:**
   ```sh
   helm install voting-app .
   ```

3. **Customize values:**
   Edit `values.yaml` to change images, ports, credentials, etc.
   Or override values on the command line:
   ```sh
   helm install voting-app . \
     --set postgresql.credentials.username=myuser \
     --set postgresql.credentials.password=mypassword
   ```

4. **Upgrade the release:**
   ```sh
   helm upgrade voting-app .
   ```

5. **Uninstall:**
   ```sh
   helm uninstall voting-app
   ```

## Configuration

All configuration is in `values.yaml`:
- **postgresql**: image, replicaCount, service, credentials, persistence
- **redis**: image, replicaCount, service, persistence
- **vote**: image, replicaCount, service (NodePort)
- **result**: image, replicaCount, service (NodePort)
- **worker**: image, replicaCount

## Structure
- **templates/postgresql.yaml**: PostgreSQL secret, deployment, and service
- **templates/redis.yaml**: Redis deployment and service
- **templates/vote.yaml**: Vote app deployment and service
- **templates/result.yaml**: Result app deployment and service
- **templates/worker.yaml**: Worker deployment
- **values.yaml**: Default configuration values

## Notes
- The database credentials are stored as a Kubernetes Secret and referenced by the PostgreSQL deployment.
- NodePort is used for vote and result services for external access.
- All services and deployments are fully configurable via `values.yaml`. 