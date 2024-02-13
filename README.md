This repository will provide you with what you need in order to build an AWS CI/CD pipeline using ECS Fargate tasks. Previous work by Cloudposse provided the foundation for this project.

**Prequisite tools**
  
* Git
* Terraform
* AWS CLI


If the AWS CLI is not configured but it is installed, you can run the command 'aws configure' and this will prompt for you a secret key, ID, region, and output. The secret key and ID are required. The rest of the prompts are optional, and specifying a region is highly recommended.

*Going forward this README assumes you have the AWS CLI configured for the AWS account you wish to launch this infrastructure.*

# Getting started

**Clone this git repo**  
Execute the following commands:

>git clone https://github.com/roman-agentic/terraform-ecs-ci-cd-pipeline.git

**Terraform set up**  
Execute the following commands:
>1. cd terraform-ecs-ci-cd-pipeline/examples/complete/  
>2. terraform init

In this directory there is a tfvars file that provides key configuration details to Terraform. Here are a few values I have defaulted:  

* region = "us-east-2"
* name = "ecs-codepipeline"
* container_name = "kinda\_confused"
* container_image = "alpine"
* image\_repo\_name = "terraform-aws-ecs-codepipeline2"
* codecommit\_repo\_name = "test2"

The image repo name refers to the ECR repository that will get created.

codecommit\_repo\_name refers to the codecommit repository that will get created.

You should receive a message that says "Terraform has been successfully initialized!"

**Terraform apply**  
Execute the following commands:
>terraform apply --var-file="variables.tfvars"

**Getting CodeCommit Credentials**  
Execute the following commands:
> aws iam create-service-specific-credential --user-name codecommit\_test\_user --service-name codecommit.amazonaws.com

You will receive some output like this:  
> {
    "ServiceSpecificCredential": {
        "CreateDate": "xxxxxxxx",
        "ServiceName": "codecommit.amazonaws.com",
        "ServiceUserName": "xxxxxxxxx",
        "ServicePassword": "xxxxxxxx": "ServiceSpecificCredentialId": "ACCA5SHSXAGFLVQQO4PHC",
        "UserName": "codecommit\_test\_user",
        "Status": "Active"
    }
}

You need to save the value of ServiceUserName, ServicePassword, and ServiceSpecificCredentialId. You will need all of these values later. 

**Warning**  
*Because we manually generated these credentials, they must also be manually removed before we do a terraform destroy. Otherwise the Terraform destroy will fail*

----
**Pushing to CodeCommit**  
Execute the following commands:
> aws codecommit get-repository --repository-name test2

From the output received, you want to save the value for "cloneUrlHttp":

Now cd to a directory of your choosing or stay in the current directory and do a git clone using the url you received from the cloneURLHttp:
> git clone 'placeholder-for-clone-url-http-value'

You will then be prompted for a username, use the value you saved earlier from ServiceUserName. You will then be prompted for a password, use the value from ServicePassword.

I already have a dockerfile and buildspec prepared, so navigate to https://github.com/roman-agentic/infrastructure-CI-CD  

Get these files and then place them in the directory for test2.

Once this is ready, be sure to do git add, git commit, and then git push origin master in the test2 folder to the test2 codecommit repository.

# Congratulations! 
# A deployment is underway
This should trigger a build and your infrastructure should automatically be deployed. For this test, we have launched an nginx container.

The only thing we need to do now is verify the build has completed.
-

**How to view your working nginx task**  
Execute the following commands:
>aws ecs list-tasks --cluster eg-test-ecs-codepipeline

You want to get the long string of digits after in the arn for the task ID. 

Next
> aws ecs describe-tasks --task b3f09cxxxxxxxxx30c22a407bcd --cluster eg-test-ecs-codepipeline

We are looking for the ENI, which should be at the bottom of this output in this section:

{
                            "name": "networkInterfaceId",
                            "value": "eni-0xxxxxxx"
                        },

Next

> aws ec2 describe-network-interfaces --network-interface-ids eni-0xxxxxxxx

Finally look through this output for public IP or DNS  
Association": {
                        "IpOwnerId": "amazon",
                        "PublicDnsName": "ec2-1xxxxxxx.us-east-2.compute.amazonaws.com",
                        "PublicIp": "1x.xxxx.xxx.xx"
                    }
> 

Once you have this, you can enter the DNS or IP in your browser to view the working Nginx task, or you can curl right in your terminal

> curl 1x.xxxx.xxx.xx

You should get output back like this


<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

# Clean up   
If you wish to clean this environment up, run the following commands:

> aws iam delete-service-specific-credential --user codecommit\_test\_user --service-specific-credential-id xxxxxxQQO4PHC

(We saved this ID from earlier)   
Change back into the terraform-ecs-ci-cd-pipeline/examples/complete/ which contains our TF state
> terraform destroy --var-file="variables.tfvars"


# That is all folks. 
Thanks for trying this out.
