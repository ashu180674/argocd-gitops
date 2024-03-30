<a name="kix.ut7cf78uih67"></a>**Task 1: Setup and Configuration**

**Step1: Create a github repo**
![Screenshot from 2024-03-30 20-11-41](https://github.com/ashu180674/argocd-gitops/assets/105533911/a9708669-f3ec-4e06-81df-65412f2af449)




- **Installing  Argo CD on Kubernetes Cluster**:

I have created a kubernetes  cluster on  my local machine with the help of minikube using docker driver.

To install agrocd on your cluster enter the below commands 

kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

namespace/argocd created


- **Installing Argo Rollouts**

kubectl create namespace argo-rollouts

kubectl apply -n argo-rollouts -f https://raw.githubusercontent.com/argoproj/argo-rollouts/stable/manifests/install.yaml



### <a name="kix.8uk1z16uyqwi"></a><a name="_w75rncmehnlw"></a>**Task 2: Creating the GitOps Pipeline**

**Step 1:  Dockerize the App**


*# Stage 1: Build the React app*

FROM node:alpine as build

WORKDIR */app*

COPY *package.json*  *./*

RUN *yarn* *install*

COPY *.* *.*

RUN *yarn* *build*

*# Stage 2: Serve the React app using Nginx*

FROM nginx:alpine

COPY *--from*=build */app/build* */usr/share/nginx/html*

EXPOSE *80*

CMD *[*"nginx"*,* "-g"*,* "daemon off;"*]*






Build the Docker image by running the following command in your terminal

docker build -t ashu1806/my-react-app .

Now push the created image to the dockerhub by running the following command:

Docker push  ashu1806/my-react-app 



- **Deploy the Application Using Argo CD:**

Modify the Kubernetes manifests in your repository to use the Docker image you pushed.

Deployment.Yaml

apiVersion: apps/v1

kind: Deployment

metadata:

` `name: web-app

spec:

` `replicas: 3

` `selector:

`   `matchLabels:

`     `app: web-app

` `template:

`   `metadata:

`     `labels:

`       `app: web-app

`   `spec:

`     `containers:

`     `- name: web-app

`       `image: ashu1806/my-react-app:latest

`       `ports:

`       `- containerPort: 80

\---

apiVersion: v1

kind: Service

metadata:

` `name: web-app

spec:

` `selector:

`   `app: web-app

` `ports:

`   `- protocol: TCP

`     `port: 80

`     `targetPort: 80


Configure Argo CD to monitor your repository and automatically deploy changes to your Kubernetes cluster.


### <a name="kix.3tg86jpo1bx7"></a>**Task 3: Implementing a Canary Release with Argo Rollouts**


Define a Rollout Strategy:

Rollout.yaml

apiVersion: argoproj.io/v1alpha1

kind: Rollout

metadata:

` `name: web-app-rollout

spec:

` `replicas: 3

` `selector:

`   `matchLabels:

`     `app: web-app

` `template:

`   `metadata:

`     `labels:

`       `app: web-app

`   `spec:

`     `containers:

`     `- name: web-app

`       `image: ashu1806/my-react-app:latest

`       `ports:

`       `- containerPort: 80

` `strategy:

`   `canary:

`     `steps:

`     `- setWeight: 10

`     `- pause: {}

`     `- setWeight: 20

`     `- pause:

`         `duration: 30s

`     `- setWeight: 30





- **Apply the manifest** 

Kubectl apply -f deployment.yaml

kubectl apply -f rollout.yaml

![Screenshot from 2024-03-30 23-40-38](https://github.com/ashu180674/argocd-gitops/assets/105533911/065b9db7-1f97-47ba-b6f8-51905f5e0da2)






**To get the dashboard**

kubectl argo rollouts dashboard

![Screenshot from 2024-03-30 23-51-53](https://github.com/ashu180674/argocd-gitops/assets/105533911/eca8e937-02ab-499e-808c-08f3157e879b)
