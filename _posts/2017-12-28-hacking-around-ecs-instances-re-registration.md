---
title: Hacking around ECS instances re-registration
layout: post
tags:
- aws
- ecs
- docker
commentIssueId: 1
---

Amazon EC2 Container Service (ECS) is the Docker-based container management service of the notorious cloud giant. If you haven't heard of it, you are most probably not tied up on Amazon's ship or you haven't researched enough for Kubernetes alternatives.  Some years ago, Werner Vogels has posted an [interesting article](http://www.allthingsdistributed.com/2015/07/under-the-hood-of-the-amazon-ec2-container-service.html){:target="_blank"} that gives you a quick grasp of how ECS works underneath.
<!--more-->

The fact is that if you are using Amazon ECS, you are not supposed to re-use ECS instances because you are in the cloud for god's shake, so launch another one of it, why thinking about it? The reality though is sometimes trickier than your best practices' cozy thoughts.

Say that you want to create a brand new ECS cluster for your favorite team's services and then you realise that someone has resorted to store stuff locally on the ECS host. For a start, you want to be nice and fast so you decide to move the current instances to the new ECS cluster and then let them move stuff away at their own pace. And this is how you end up digging in places you haven't before and things start to become interesting.

Essentially, if you really want to re-use an ECS instance and re-register it to a different cluster, the steps that you should follow are pretty much the following:


* First of all, deregister the instance from the current cluster. Using AWS CLI, you can do this like this:
```
$ aws ecs deregister-container-instance --cluster dev-ecs-cluster --container-instance 76e64af5-b389-409f-bf56-67f2f2445384 --region eu-west-1
```

* Next, SSH into the machine to stop the ecs-agent (which is a Docker container after all):
```
$ docker stop ecs-agent
```

* Then we need to edit ecs-agent's configuration file to change the cluster name, and remove the agent's data file:
```
$ sed -i 's/ECS_CLUSTER=.*/ECS_CLUSTER=new-ecs-cluster/' /etc/ecs/ecs.config
$ rm /var/lib/ecs/data/ecs_agent_data.json
```

*  There is one more file that we should manually modify and this is the container's json config file which is usually located under `/var/lib/docker/containers/CONTAINER_ID/config.v2.json`. So get there with your favorite editor and modify the `ECS_CLUSTER` sub-key value accordingly. 

* Finally we can start again the ecs-agent container:
```
$ docker start ecs-agent
```

Finally, wait some seconds to let the ECS agent register with the new cluster and voilà! You should finally see it under the ECS instances tab list in your ECS cluster's page.
