# ECS Service updates do not trigger a new deployment

## What happened?
After creating an ECS Service, I am unable to change the initially set desiredCount in the provisioned resource.  

It appears changes to desiredCount field **do not trigger a new deployment** or alter the existing deployment.

Initial Service:

```
apiVersion: ecs.aws.crossplane.io/v1alpha1
kind: Service
metadata:
  labels:
    testing.upbound.io/example-name: sample-app-service
  name: sample-app-service
spec:
  forProvider:
    desiredCount: 1
    deploymentController:
      type_: ECS
    deploymentConfiguration:
      maximumPercent: 200
      minimumHealthyPercent: 0
    launchType: FARGATE
    region: us-east-1
    schedulingStrategy: REPLICA
    platformVersion: 1.4.0 
    clusterSelector:
      matchLabels:
        testing.upbound.io/example-name: sample-app-cluster
    loadBalancers:
    - containerName: sample-app-container
      containerPort: 80
      targetGroupARNSelector:
        matchLabels:
          testing.upbound.io/example-name: sample-app-target-group
    networkConfiguration:
      awsvpcConfiguration:
        assignPublicIP: ENABLED
        securityGroupSelector:
          matchLabels:
            testing.upbound.io/example-name: sg-us-east-1
        subnetSelector:
          matchLabels:
            all-subnets: included
    taskDefinitionSelector:
      matchLabels:
        testing.upbound.io/example-name: sample-app-task
```
<br>

Once this is up and running, I change desiredCount to 2 and apply.

I can see the change reflected in the object's Spec section on the cluster:

![image](https://user-images.githubusercontent.com/78125388/219165980-44ea996d-1249-4b9f-a228-26443831ce7c.png)
<br>

But the object's Status section shows that no new deployment has been rolled out and displays the old deployment's config:

![image](https://user-images.githubusercontent.com/78125388/219166386-73c608d5-050e-490c-8b95-aef1deefe6d2.png)
<br>

The object's events section clearly shows only 1 deployment has ever been attempted:

![image](https://user-images.githubusercontent.com/78125388/219167974-b05e1118-2e01-4a58-b7f4-acc5dddfc7fd.png)
<br>

At this point, to get another deployment, _I have to delete the service and create a new one..._ 

**How can I redeploy a service with a new desiredCount value without deleting and re-creating it?** 

In the console, there is a "force new deployment" option that allows this when you hit "Update Service" :

![image](https://user-images.githubusercontent.com/78125388/219170210-c3bc47b1-98e2-4bc0-8cfd-4b3a5e37485f.png)
<br>

Is this functionality available in Crossplane?

## How can we reproduce it?
You can apply all yaml in this repo to lay down a network, ALB, and ECS Fargate cluster, task, service, and role. 
<br>
When you change desiredCount to 2 and apply, you will see no new deployment occuring. 

## What environment did it happen in?
Crossplane version: v1.11.0<br>
Kubectl client version: v1.25.0<br>
Kubectl server version: v1.25.3<br>
OS: Pop!_OS 22.04 LTS<br>
KinD: 0.17.0

