This lab builds upon lab 1


## 5. Build the Docker Images and push them to ECR

Populate some environment variables that we will be using

```
sudo yum install -y jq

AWS_ACCOUNT_ID=$(aws sts get-caller-identity | jq -r '.Account')

```

First create the ECR repositories for the 2 applications.

```
aws ecr create-repository --repository-name colorteller

aws ecr create-repository --repository-name colorgateway

```

In the terminal of Cloud9, clone the code

```
git clone https://github.com/tohwsw/aws-app-mesh-examples.git

```

Retrieve the login command to use to authenticate your Docker client to your registry.

```
$(aws ecr get-login --no-include-email --region ap-southeast-1)

```

Go to the folder examples/apps/colorapp/src/colorteller. Execute a docker build with the respective repository uri for colorteller and push it to the repository.

```
cd ~/environment/aws-app-mesh-examples/examples/apps/colorapp/src/colorteller

docker build -t $AWS_ACCOUNT_ID.dkr.ecr.ap-southeast-1.amazonaws.com/colorteller .

docker push $AWS_ACCOUNT_ID.dkr.ecr.ap-southeast-1.amazonaws.com/colorteller:latest
```

Go to the folder examples/apps/colorapp/src/gateway. Execute a docker build with the respective repository uri for colorgateway and push it to the repository.

```
cd ~/environment/aws-app-mesh-examples/examples/apps/colorapp/src/gateway

docker build -t $AWS_ACCOUNT_ID.dkr.ecr.ap-southeast-1.amazonaws.com/colorgateway .

docker push $AWS_ACCOUNT_ID.dkr.ecr.ap-southeast-1.amazonaws.com/colorgateway:latest

```



## 6. Deploy the application

To deploy the app, download colorapp.yml and and deploy it.

```
curl -O https://raw.githubusercontent.com/tohwsw/kubernetes-basics/master/Lab2-Deployment/colorapp.yml

```

**Important** You will need to update color.yaml with the ECR respository uri of your colorgateway and colorteller docker images.
You can do so by substituting the account id with your own.

```
sed -i 's/571708975227/<$AWS_ACCOUNT_ID>/g' colorapp.yml

```

Next, deploy the applications.


```
kubectl apply -f colorapp.yml

```

Verify the pods are runnning


```
kubectl get pods

```



## 7. Test the application

An application load balancer is also deployed. Identify the address of the alb via the command.

```
export LB=$(kubectl get svc -o wide | grep colorgateway-service | awk '{print $4}')

```


Next, paste the following to request the colorgateway service. Do note that the alb might take a few minutes to become active.

```
while [ 1 ]; do  curl -s --connect-timeout 2 $LB:/color;echo;sleep 1; done

```

You should see the default color white is returned on each request.

## 8. Optional

You can remove the deployment and try out the deployment using imperative commands

```
kubectl apply -f colorapp.yml

```

```
kubectl create deployment colorteller --image='284245693010.dkr.ecr.ap-southeast-1.amazonaws.com/colorteller:latest'
kubectl set env deployment/colorteller SERVER_PORT=9080
kubectl scale --replicas=2 deployment/colorteller
kubectl expose deployment colorteller --port=9080 --target-port=9080
kubectl create deployment colorgateway --image='284245693010.dkr.ecr.ap-southeast-1.amazonaws.com/colorgateway:latest'
kubectl set env deployment/colorgateway SERVER_PORT=9080 'COLOR_TELLER_ENDPOINT=colorteller.default.svc.cluster.local:9080' 'TCP_ECHO_ENDPOINT=colorteller.default.svc.cluster.local:9080'
kubectl scale --replicas=2 deployment/colorgateway
kubectl expose deployment colorgateway --port=80 --target-port=9080 --type=LoadBalancer

```

## 9. Lab Cleanup

To clean up the lab, please delete using the following command

```
eksctl delete cluster --name=$CLUSTER --region=ap-southeast-1

```

## 10. Optional Labs

There are more labs hosted at https://eksworkshop.com/. Do check them out!








