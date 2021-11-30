## Concepts

![](./diagrams/overview.drawio.svg)

## Install Google Cloud SDK (gcloud)

https://cloud.google.com/sdk/docs/install

If you have homebrew (on linux or mac) you may also try:

```
brew install --cask google-cloud-sdk
```

## Login to Google Cloud

```
gcloud auth login
```

## Login to the Cluster

```
gcloud container clusters get-credentials csu-workshop --region us-central1 --project csu-workshop-333616
```

To check that you are logged in, run:

```
kubectl get nodes
```

You should see something like:

```
NAME                                          STATUS   ROLES    AGE     VERSION
gk3-csu-workshop-default-pool-8cb550ee-b2rc   Ready    <none>   8m30s   v1.21.5-gke.1302
gk3-csu-workshop-default-pool-ebded107-03nj   Ready    <none>   8m30s   v1.21.5-gke.1302
```

## Create a React App

```
npx create-react-app react-example --use-npm --scripts-version=4.0.3
```

`cd` into the newly-created directory:

```
cd react-example
```

Add a dockerfile:

```
cat <<EOF > Dockerfile
FROM node:16-alpine as build
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build

FROM nginxinc/nginx-unprivileged
COPY --from=build /app/build /usr/share/nginx/html
EOF
```

Build and run the docker app locally:

```
export IMAGE=us-central1-docker.pkg.dev/csu-workshop-333616/workshop/$USERNAME-react:v1
docker build --no-cache -t $IMAGE .
docker run -it -p 8080:8080 --rm $IMAGE
```

## Docker Login

```
gcloud auth configure-docker us-central1-docker.pkg.dev
```

## Push the Image

```
docker push $IMAGE
```

## Create a Kubernetes Namespace

```
kubectl create namespace $USERNAME
```

## Create Deployment Configuration File

Create a `deployment.yaml` file:

```
cat <<EOF > deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: react
  namespace: $USERNAME
  labels:
    app: react
spec:
  replicas: 1
  selector:
    matchLabels:
      app: react
  template:
    metadata:
      labels:
        app: react
    spec:
      containers:
        - name: react
          image: $IMAGE
          ports:
            - containerPort: 8080
EOF
```

## Deploy Your Application

```
kubectl apply -f deployment.yaml
```

You should see:

```
deployment.apps/react created
```

**List your pods**

```
kubectl get pods -n $USERNAME
```

## Create the Service Configuration File

```
cat <<EOF > service.yaml
apiVersion: v1
kind: Service
metadata:
  name: react
  namespace: $USERNAME
spec:
  selector:
    app: react
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
EOF
```

## Create the Service

```
kubectl apply -f service.yaml
```

## Get the IP address

Run this command. It may take a few minutes before the external IP appears

```
kubectl get svc -n $USERNAME
```
