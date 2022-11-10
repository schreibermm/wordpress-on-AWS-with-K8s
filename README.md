# wordpress-on-AWS-with-K8s

![Alternate image text](https://raw.githubusercontent.com/schreibermm/wordpress-on-AWS-with-K8s/main/WP_K8S_AWS.png)


## Prerequisites:
    1. AWS account
    2. Ownership of a DNS name
    3. Availability Zone - decide in which AZ the cluster will be spin up



**Configure EFS:**
  1. Create EFS volume in AWS (<ID_Number> should be unique, but can be random)::
        $ aws efs create-file-system --creation-token <ID_Number>

  2. Identify the Subnet of the and Security Group Id of the EFS volume and note them down:
      $ aws describe-instances 
          1. To get subnet-id, search response for "SubnetId" (for example "subnet-8cc390e8")
          2. To get security-group-id, search response for "SecurityGroups.GroupId" (for example "sg-f1a22797")
    
  3. Create EFS Mount Target using the "FileSystemId" in the same subnet as the cluster:
      $ aws efs create-mount-target --filesystem-id <FileSystemId> --subnet-id <SubnetId> --security-groups <SecurityGroupId>

  4. Update the "wordpress-web.yml" Deployment file with "FileSystemId" and "Region" values:
      spec.template.spec.volumes.nfs.server: eu-west-1a.<FileSystemId>.efs.<Region>.amazonaws.com



**Configure MySQL DB pod with EBS volume:**
  1. kubectl create -f storage.yml
  2. kubectl create -f pv-claim.yml
  3. kubectl create -f wordpress-secrets.yml
  4. kubectl create -f wordpress-db.yml
  5. kubectl create -f wordpress-db-service.yml
  6. Verify the EBS volume:
      - kubectl get pvc
      - kubectl get pod
      - kubectl describe <POD_NAME>

	

**Launch WordPress Pod:**
  1. kubectl create -f wordpress-web.yml
  2. Wait for the pods to be created

	
**Launch WordPress LoadBalancer:**
  1. kubectl create -f wordpress-web-service.yml


**To delete the WordPress cluster:**
  1. Remove the cluster:
      - $ kubectl delete deployment hello-node  
  2. Delete the EFS:
      - AWS Console > EFS > Delete file system



