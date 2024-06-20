# notes-app-ci/cd-example
Simple Node.js application with unit tests to show how to automate tests using Harness CI!

## Prerequisites
> - Free [Harness cloud](https://www.harness.io/products/continuous-integration?utm_source=internal&utm_medium=social&utm_campaign=devadvocacy&utm_content=pavan_notes_article&utm_term=get-started) account to set up continuous integration 
> - [Node.js](https://nodejs.org/en/download/) installed 


# Deploying an Application on Kubernetes: A Complete Guide!

![guide](https://imgur.com/2TkZCg7.png)

**Kubernetes** is an open-source platform for automating the deployment, scaling, and management of containerized applications. It is a popular tool for container orchestration and provides a way to manage large numbers of containers as a single unit rather than having to manage each container individually.

## ✍️ **Importance of Kubernetes**

**Kubernetes** has become an essential tool for managing and deploying modern applications, and its importance lies in its ability to provide a unified platform for automating and scaling the deployment, management, and scaling of applications. With Kubernetes, organizations can achieve increased efficiency and agility in their development and deployment processes, resulting in faster time to market and reduced operational costs. Kubernetes also provides a high degree of scalability, allowing organizations to scale their applications as their business grows and evolves easily.

Additionally, Kubernetes offers robust security features, ensuring that applications are protected against potential threats and vulnerabilities. With its active community and extensive ecosystem, Kubernetes provides organizations with access to a wealth of resources, tools, and services that can help them improve and enhance their applications continuously. Overall, the importance of using Kubernetes lies in its ability to provide a flexible, scalable, and secure platform for managing modern applications and enabling organizations to stay ahead in a rapidly evolving digital landscape.

## ✍️ **Here's a basic overview of how to use Kubernetes**:

* ### **Set up a cluster**:
    

To use Kubernetes, you need to set up a cluster, which is a set of machines that run the Kubernetes control plane and the containers. You can set up a cluster on your infrastructure or use a cloud provider such as Amazon Web Services (AWS), Google Cloud Platform (GCP), or Microsoft Azure.

I used minikube, download and install [minikube](https://minikube.sigs.k8s.io/docs/start/?arch=%2Fmacos%2Fx86-64%2Fstable%2Fbinary+download) 

* ### **Package your application into containers**:
    

To run your application on Kubernetes, you need to package it into one or more containers. A container is a standalone executable package that includes everything needed to run your application, including the code, runtime, system tools, libraries, and settings.

* ### **Define the desired state of your application using manifests**:
    

Kubernetes uses manifests, which are files that describe the desired state of your application, to manage the deployment and scaling of your containers. The manifests specify the number of replicas of each container, how they should be updated, and how they should communicate with each other.

* ### **Push your code to an SCM platform**:
    

Push your application code to an SCM platform such as GitHub.

* ### **Use a CI/CD tool to automate**:
    

Use a specialised CI/CD platform such as Harness to automate the deployment of your application. Once you set it up, done; you can easily and often deploy your application code in chunks whenever a new code gets pushed to the project repository.

* ### **Expose the application**:
    

Once you deploy your application, you need to expose the application to the outside world by creating a Service with a type of LoadBalancer or ExternalName. This allows users to access the application through a stable IP address or hostname.

* ### **Monitor and manage your application**:
    

After your application is deployed, you can use the kubectl tool to monitor the status of your containers, make changes to the desired state, and scale your application up or down.

These are the general steps to deploy an application on Kubernetes. Depending on the application's complexity, additional steps may be required, such as configuring storage, network policies, or security. However, this should give you a good starting point for deploying your application on Kubernetes.

Today, we will see how to automate simple application deployment on Kubernetes using Harness.

## 👉 **Prerequisites**

* Free [Harness Cloud](https://app.harness.io/auth/#/signup/?module=cd&utm_source=internal&utm_medium=social&utm_campaign=devadvocacy&utm_content=pavan_notes_cicd_article&utm_term=get-started) account
    
* Download and install [Node.js and npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)
    
* GitHub account, we will be using our [deploying-simple-nodejs-app-on-k8s](https://github.com/awsactivators/deploying-simple-nodejs-app-on-k8s)
    
* Kubernetes cluster access, you can use [Minikube](https://minikube.sigs.k8s.io/docs/start/) or [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/) to create a single-node cluster
    

## 👉 **Tutorial**

[![Kubernetes deployment](https://res.cloudinary.com/practicaldev/image/fetch/s--ryaqUHB9--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_800/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qm7dy8qnl1kyg3q10v0b.png)](https://res.cloudinary.com/practicaldev/image/fetch/s--ryaqUHB9--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_800/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qm7dy8qnl1kyg3q10v0b.png)

We will use our sample application that is already in the GitHub repository. We will use a Kubernetes cluster to deploy our application. Next, we will use a CI/CD platform, Harness, in this tutorial to show how we can automate the software delivery process easily.

### **Step 1: Test the sample application locally**

Fork and clone the [deploying-simple-nodejs-app-on-k8s](https://github.com/awsactivators/deploying-simple-nodejs-app-on-k8s)

Go to the application folder with the following command

```bash
cd deploying-simple-nodejs-app-on-k8s
```

Install dependencies with the following command

```bash
npm install
```

Run the application locally to see if the application works perfectly well

```bash
node app.js
```

### **Step 2: Containerize the application**

You can see the Dockerfile in the sample application repository.

```go
FROM node:14-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 3000

CMD [ "npm", "start" ]
```

Use the following command to build, tag and push the image to any container registry of your choice. I will push it to Docker Hub in this tutorial.

For Mac M1, use the following command

```bash
docker buildx build --platform=linux/arm64 --platform=linux/amd64  -t docker.io/$your docker hub user name/$image name:$tag name --push  -f ./Dockerfile .
```

For other than Mac M1, use the below commands to build and push the image,

```bash
docker build -t $your docker hub user name/$image name .
```

```bash
docker push $your docker hub user name/$image name .
```

[![build and pushed docker image to dockerhub](https://res.cloudinary.com/dqtokk1cn/image/upload/v1718913623/k8s-nodejs-app/dwe7km3hgltcej59gkee.png)]

### **Step 3: Create or get access to a Kubernetes cluster**

Make sure to have access to a Kubernetes cluster from any cloud provider. You can even use Minikube or Kind to create a cluster. In this tutorial, we are going to make use of a Kubernetes cluster from Google Cloud (GCP)

I already have an account on Google Cloud, so creating a cluster will be easy.

### **Step 4: Make sure the Kubernetes manifest files are neat and clean**

You need deployment yaml and service yaml files to deploy and expose your application. Make sure both files are configured properly.

You can see that we have deployment.yaml and service.yaml file already present in the sample application repository.

**Below is our deployment.yaml file.**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: notes-app-deployment
  labels:
    app: notes-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: notes-app
  template:
    metadata:
      labels:
        app: notes-app
    spec:
      containers:
      - name: notes-app-deployment
        image: iamvieve/note-app:0.2.0
        resources:
          requests:
            cpu: "100m"
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 3000
```

**Below is our service.yaml file**

```yaml
apiVersion: v1
# Indicates this as a service
kind: Service
metadata:
 # Service name
 name: notes-app-deployment
spec:
 selector:
   # Selector for Pods
   app: notes-app
 ports:
   # Port Map
 - port: 80
   targetPort: 3000
   protocol: TCP
 type: LoadBalancer
```

Apply the manifest files with the following commands. Starting with deployment and then service yaml file.

```bash
kubectl apply -f deployment.yaml
```

```bash
kubectl apply -f service.yaml
```

Verify the pods are running properly as expected after applying the kubectl apply commands.

```bash
kubectl get pods
```

You should see the pods and their status.

[![running pods/services/deployments](https://res.cloudinary.com/dqtokk1cn/image/upload/v1718913623/k8s-nodejs-app/kyznin1l5ovaeu5f4mnv.png)]

### **Step 5: Let’s confirm our note app**

To view the app externally you need to get the service name

```bash
kubectl get service
```

[![kubectl get service](https://res.cloudinary.com/dqtokk1cn/image/upload/v1718924956/k8s-nodejs-app/wta3luawnokaqakbgvw1.png)]

Then run the minikube command

```bash
minikube service service-name
```

[![expose service outside minikube](https://res.cloudinary.com/dqtokk1cn/image/upload/v1718924957/k8s-nodejs-app/ka6bqb3vh8atx4sjlriu.png)]

The app should open on your browser automatically

[![App running](https://res.cloudinary.com/dqtokk1cn/image/upload/v1718924957/k8s-nodejs-app/d5tbnsaxax1enpnnw5c6.png)]