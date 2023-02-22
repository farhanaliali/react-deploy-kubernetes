# Deploying React app on kubernetes

## Demp react app 

Clone the demo react app from github 

    git clone https://github.com/IBM/deploy-react-kubernetes

Go to deploy-react-kubernetes 

    cd deploy-react-kubernetes

Now run docker build command to build this image 

    docker build -t deploy-react-kubernetes .

Test you image to run locally 

    docker run -p 3000:3000 deploy-react-kubernetes

Now make sure your application is running fine on 

    http://localhost:3000


Now our image is ready so we need to push this image to registry i am using docker public registry 
Create dockerhub account and go to setting ---> security ----> create token with write access.
Now tag the docker image with your registry username in my case 

    docker tag deploy-react-kubernetes farhanluckali/deploy-react-kubernetes

Login to docker  

    docker login -u farhanali 
Paste the access token when it ask for password 

Now push image to repo 

    docker push farhanluckali/deploy-react-kubernetes

verfiy image from your dockerhub account 


## Deploying your image on  Elastic Kubernetes Service 

create iam user with administrator access (https://docs.aws.amazon.com/streams/latest/dev/setting-up.html)

Genrate access keys from you iam user https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html

Download the awscli and configure your iam user on it.

downlaod and install eksctl  (https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)

To install or update eksctl on Linux

    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
Move the extracted binary to /usr/local/bin.

    sudo mv /tmp/eksctl /usr/local/bin

Test that your installation was successful with the following command.

    eksctl version


Now run the below command to create the cluster 

    eksctl create cluster

Wait util every thing is ready 

Meanwhile we will isntall kubectl tool for cluster managemnet 

    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

    kubectl version --client

Now kubectl istalled and our cluster is ready 
verify cluster 

    kubectl get nodes 


## Deploying appliction to kubernetes cluster 


create deployment file for react app (deployment.yaml)
    
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name:  react-app
      labels:
        app:  react-app
    spec:
      selector:
        matchLabels:
          app: react-app
      replicas: 1
      template:
        metadata:
          labels:
            app:  react-app
        spec:
          containers:
          - name:  react-app
            image:  farhanluckali/react-kubernetes
            ports:
            - containerPort: 3000

save and exit 

Now deploy this file from kubectl 

    kubectl create -f deployment.yaml 

Verify the deployment 

    kubectl get pods 

When pod is in running state then we will create the service file (service.yaml)


    apiVersion: v1
    kind: Service
    metadata:
      name: react-app
      namespace: default
    spec:
      selector:
        app: react-app
      type: LoadBalancer
      ports:
      - name: react-app
        protocol: TCP
        port: 80
        targetPort: 3000

save and exit 

Now run the below command to deploy service file 

    kubectl create -f service.yaml 

Now wait until you will get the external ip 

    kubectl get svc 


Copy the external URL and paste in to browser 

    http://a45af503454414721b3d0350940ae1c8-353256290.ap-southeast-1.elb.amazonaws.com/


You application is now accessable 