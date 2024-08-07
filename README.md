# About

This project uses Kubernetes (k8s) to deploy a Next.js + Node.js + MongoDB app.

# How to Use

## Prerequisites

1. Ensure you have a Kubernetes cluster.
2. Install `kubectl` and connect it to your cluster.
3. Install and initialize Docker (for Option A)

## Option A: Build and Push Docker Image

1. **Build the Docker Image**

    ```bash
    docker build -t registry.gitlab.com/<project_name>/<repository_name> .
    ```

2. **Push the Docker Image to GitLab Container Registry**

    ```bash
    docker login registry.gitlab.com
    docker push registry.gitlab.com/<project_name>/<repository_name>
    ```

3. **Create a Kubernetes Secret for GitLab Registry**

    ```bash
    kubectl create secret docker-registry gitlab-registry-secret \
        --docker-server=registry.gitlab.com \
        --docker-username=your-username \
        --docker-password=your-password \
        --docker-email=your-email@example.com
    ```

    Replace `gitlab-registry-secret` with the desired name for your Kubernetes secret.

4. **Update Deployment Configuration**

    In your `deployment.yaml`, use the created secret by adding it to `spec.template.spec.imagePullSecrets.name`.

## Option B: Use a Pre-built Public Image

Use the public image `pwstaging/demo-crm:latest` without needing Kubernetes secrets.

## Deploy the Application

1. **Apply the Kubernetes Configuration**

    ```bash
    kubectl apply -f .
    ```

2. **Verify MongoDB StatefulSet**

    Check that the MongoDB StatefulSet is running:

    ```bash
    kubectl get statefulset
    ```

    You should see:

    ```
    NAME    READY   AGE
    mongo   3/3     3m57s
    ```

    Ensure the `READY` column matches the number of replicas specified in `spec.replicas`.

3. **Initialize MongoDB Replica Set**

    Execute the following command to initialize the MongoDB replica set:

    ```bash
    kubectl exec -it mongo-0 -- mongosh <<< '
    rs.initiate({
        _id: "mongoReplSet",
        version: 1,
        members: [
            { _id: 0, host: "mongo-0.mongo-svc:27017" },
            { _id: 1, host: "mongo-1.mongo-svc:27017" },
            { _id: 2, host: "mongo-2.mongo-svc:27017" }
        ]
    })'
    ```

    Replace `mongoReplSet`, `mongo-N`, and `mongo-svc` with the appropriate values from your `statefulset.yaml`.

4. **Check Replica Set Status**

    Verify the replica set initialization:

    ```bash
    kubectl exec -it mongo-0 -- mongosh <<< 'rs.status()'
    ```

    You should see a JSON output starting with `"mongoReplSet [direct: primary] test>"`.

5. **Test Your Application**

    Get the external IP of the `demo-crm-svc` service:

    ```bash
    kubectl get svc
    ```

    Use the external IP to access the application, add a new customer, and verify that the customer appears on the dashbo
