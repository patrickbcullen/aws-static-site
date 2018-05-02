# aws-static-site
AWS CloudFormation templates for building a static website hosted on S3 and CloudFront. The project also includes a simple React website to demonstrate the deployment process.

## Setup 
To deploy the changes you need to create a certificate, deploy the site stack, and deploy the site to S3. We will use `blog.example.com` as our site name as an example.
### Create certificate
Create an ACM certificate in the AWS console for with names `*.example.com` and `example.com`. Use DNS validation and click on the create Route53 records buttons. Wait until the certificate is verified, copy the ARN, and use it as a parameter for the site stack.

### Create the site stack
The static-site.yml CloudFormation template creates the following AWS resources:
1. S3 bucket for the site assets
2. CloudFront distribution to support HTTPS

Example (update parameters to match your site):
```
aws cloudformation create-stack --stack-name blog-example-com-site --template-body file://cfn/static-site.yml --parameters ParameterKey=SiteDomainName,ParameterValue=blog.example.com ParameterKey=RootDomainName,ParameterValue=example.com ParameterKey=CertificateARN,ParameterValue=arn:aws:acm:us-east-1:111111111111:certificate/22222222-2222-222222222-222222222222
```

### Create the deployment pipeline 
The pipeline.yml CloudFormation template creates the follow AWS resources:
1. CodePipeline for deploying the site to S3
2. CodeBuild projects for building and deploying the site
3. IAM policies for both CodePipeline and CodeBuild
4. S3 bucket for hosting CodePipeline and CodeBuild artifacts

Example (update parameters to match your site):
```
aws cloudformation create-stack --stack-name blog-example-com-pipeline --template-body file://cfn/pipeline.yml --capabilities CAPABILITY_IAM --parameters ParameterKey=CloudFrontDistribution,ParameterValue=EEEEEEEEEEEEEE ParameterKey=Pipeline,ParameterValue=blog-example-com ParameterKey=GitHubOwner,ParameterValue=MyOwner ParameterKey=GitHubRepo,ParameterValue=MyRepo ParameterKey=GitHubToken,ParameterValue=MyToken
```

### Deploy the site
The CodePipeline will automatically deploy the site to S3 on every commit. To increase the effeciency of the cache, files have a cache expiration of 1 year. When a new version of the site is deployed all files with the exception of index.html contain a hash that changes when then contents of the file is updated. Then the CloudFront cache is invalidated for the index.html during the deployment so that users see the updated site.
