# NTLM vs AWS Elastic Load Balancing 

This lambda function is about enabling NTLM authentication with AWS Elastic Load Balancer (ELB).

## Challanges
For NTLM to work through ELB, two things are required;
1) Session stickiness
2) HTTP headers for Authentication

## Challanges
It is possible to resolve stickiness issue with CLB (classic Load Balancer) thanks to its application side cookie feature.

However, for NTLM to work, NLB is required due to the fact that ALB and CLB overwrite authentication headers required for NTLM.
But NLB doesn’t support stickiness (it is possible in Ireland and some other regions though but not in every region at the time of writing).
 
Therefore neither CLB nor NLB provided a complete solution to run 2x load-balanced backend servers with NTLM authentication enabled.
 
We also considered Windows Failover Cluster but that also didn’t provide a complete solution as there is no built-in mechanism in WFC to perform a failover if IIS application fails (require custom PowerShell scripts, etc.) and it also seemed to be more difficult to implement in AWS.
 
This solution is an alternative since it configures “a single” backend server behind an NLB at a to help us achieve;
-	clients will be provided with a single URL for access (NLB)
-	we can support NTLM
-	lambda will enable backend server rotation in case of application failures based on NLB health-checks (simulates an active/passive setup)
-	other (backup) instance will be kept offline for cost saving andl allows primary server to run with more resources


### Adjustments

1. Fill in information for some parameters
  `a. Primary instance id`
  `b. Secondary instance id`
  `c. NLB Target Group ARN`
  
  Dockerfile. It is created by [following this tutorial](https://runnable.com/docker/python/dockerize-your-flask-application).
2. I have changed it to accomodate latest version of ununtu and `python3`
3. To build docker image `docker build -t todo-flask:latest .`
4. To run the docker container `docker run -it -p 5000:8888 todo-flask `

