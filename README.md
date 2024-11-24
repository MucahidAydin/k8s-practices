### Kubernetes Practices

#### **1:** Set up a 5-node Kubernetes cluster ("1 Master + 4 Worker Nodes").
<details>

```URL
https://github.com/MucahidAydin/k8s-cluster.git
```
</details>
<br>

#### **2:** Create two namespaces named "test" and "production".
<details>

```bash
kubectl create namespace test
kubectl create namespace production
```
</details>
<br>

#### **3:** Create roles for the "junior" and "senior" groups in the "test" and "production" namespaces with specific permissions, and bind those roles to the respective groups. 
- "junior" group should have full permissions in the "test" namespace and read/list permissions in the "production" namespace.
- "senior" group should have full permissions in both "test" and "production" namespaces and read/list permissions for cluster-wide resources.

<details>

```bash
kubectl apply -f ./RBAC/role/junior-role.yaml
kubectl apply -f ./RBAC/role/senior-role.yaml
kubectl apply -f ./RBAC/rolebinding/junior-rolebinding.yaml
kubectl apply -f ./RBAC/rolebinding/senior-rolebinding.yaml
```
* [User Creation](./RBAC/user/Readme.md)
</details>
<br>

#### **4:** Install an ingress controller of your choice (e.g., nginx, traefik, haproxy).
<details>

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.0-beta.0/deploy/static/provider/cloud/deploy.yaml
```
* ``kubectl get all -n ingress-nginx``
</details>
<br>

#### **5:** Ensure that three selected worker nodes only schedule pods for the "production" environment, while preventing other pods from being scheduled on them.
<details>

```bash
kubectl taint nodes <node-name> app=production:NoExecute
```
</details>
<br>

#### **6:** Deploy the Wordpress application in both the "test" and "production" namespaces using the "wordpress:latest" and "mysql:5.6" images.
- MySQL should be accessible within the cluster as a "ClusterIP" service.
- Persistent volumes should be used for long-term storage of data.
- No sensitive information (e.g., passwords) should be stored in application or yaml files.
- Both applications should be scheduled on the same worker node.
- CPU and memory resource limits should be set for both applications.

<details>

To deploy in the test namespace:
```bash
cd ./wordpress/test
kubectl apply -f .
```

To deploy in the production namespace:
```bash
cd ./wordpress/production
kubectl apply -f .
```
</details>
<br>

#### **7:** Expose the "test" and "production" Wordpress applications to the external world using Ingress, with the following domains:
- "test" namespace application: "testblog.example.com"
- "production" namespace application: "companyblog.example.com"

<details>

To deploy the ingress in the test namespace:
```bash
cd ./wordpress/test
kubectl apply -f wordpress-ingress.yaml
```

To deploy the ingress in the production namespace:
```bash
cd ./wordpress/production
kubectl apply -f wordpress-ingress.yaml
```
</details>
<br>

#### **8:** Create a 5-replica deployment in the "production" namespace from the image "ozgurozturknet/k8s:v1", with a rolling update strategy that allows a maximum of 2 pods to be updated simultaneously. Also, define a "liveness probe" for the "/healthcheck" endpoint and a "readiness probe" for the "/ready" endpoint.
<details>

```bash
cd ./Deployment
kubectl apply -f d-prod.yaml
```
</details>
<br>

#### **9:** Expose the previous deployment to the external world via a "LoadBalancer" type service.
<details>

```bash
cd ./Deployment
kubectl apply -f d-svc.yaml
```
</details>
<br>

#### **10:** Scale the previous deployment to 3 replicas, then scale it up to 10 replicas. After that, update the deployment to use the "ozgurozturknet/k8s:v2" image.
<details>

```bash
kubectl scale deployment d-prod --replicas=3
kubectl scale deployment d-prod --replicas=10
kubectl set image deployment/d-prod d-container=ozgurozturknet/k8s:v2
```
</details>
<br>

#### **11:** Deploy the "fluentd" application as a "daemonset" across the cluster.
<details>

```bash
cd ./DaemonSet
kubectl apply -f .
```
</details>
<br>

#### **12:** Deploy a 2-node "mongodb" cluster as a "statefulset" and ensure it is working. Connect to the MongoDB pod and initiate a replica set.
<details>

```bash
cd ./MongoDB
kubectl apply -f .
```

To connect to the MongoDB pod:
```bash
kubectl exec -it mongodb-0 -- mongosh
```

To initiate the MongoDB replica set:
```bash
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "mongodb-0.mongodb-headless.test.svc.cluster.local:27017" },
    { _id: 1, host: "mongodb-1.mongodb-headless.test.svc.cluster.local:27017" },
    { _id: 2, host: "mongodb-2.mongodb-headless.test.svc.cluster.local:27017" }
  ]
});
```

</details>
<br>

#### **13:** Create a service account with "read" and "list" permissions for all resources across the cluster, then create a pod that uses this service account to list all pods using "curl".
<details>

To create the service account:
```bash
cd ./ServiceAccount
kubectl apply -f .
```

To test:
```bash
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl --insecure https://kubernetes.default.svc.cluster.local/api/v1/namespaces/test/pods --header "Authorization: Bearer $TOKEN"
```
</details>
<br>

#### **14:** Drain all pods from one worker node and prevent new pods from being scheduled on it.
<details>

To drain a node:
```bash
kubectl drain <node-name> --ignore-daemonsets --delete-local-data
```

To uncordon the node:
```bash
kubectl uncordon <node-name>
```
</details>