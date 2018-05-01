# aws-static-site
AWS CloudFormation templates for building a static website.

## Deploying
To deploy the changes you need to create a certificate, deploy the site stack, and deploy the site to S3. We will use `blog.example.com` as our site name as an example.
### Create certificate
Create an ACM certificate in the AWS console for with names `*.example.com` and `example.com`. Use DNS validation and click on the create Route53 records buttons. Wait until the certificate is verified, copy the ARN, and use it as a parameter for the site stack.

### Create the site stack
```
aws cloudformation create-stack --stack-name blog-example-com-site --template-body file://cfn/static-site.yml --parameters ParameterKey=SiteDomainName,ParameterValue=blog.example.com ParameterKey=RootDomainName,ParameterValue=example.com ParameterKey=CertificateARN,ParameterValue=arn:aws:acm:us-east-1:111111111111:certificate/22222222-2222-222222222-222222222222
```

### Deploy the site to S3
```
aws s3 cp public/index.html s3://blog.example.com
aws s3 cp public/404.html s3://blog.example.com
```
