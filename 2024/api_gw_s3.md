
## Delivering images in AWS S3 bucket through AWS API gateway

> cover image: Photo by <a href="https://unsplash.com/@francesco_ungaro?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Francesco Ungaro</a> on <a href="https://unsplash.com/photos/grey-bucket-on-brown-wooden-surface-VSwlS0PpWwc?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash">Unsplash</a>

In this article, we will talk about creating an AWS REST API as an AWS S3 proxy and delivering images in an S3 bucket through the API gateway.

## **What are the different ways to deliver images in the s3 bucket?**

1. Deliver images using public S3 URLs

2. Deliver images in S3 using AWS Lambda API Gateway Integration

3. Creating an AWS REST API as an S3 proxy

## Why deliver images in the S3 bucket through the API gateway?

The simplest way to deliver images in the S3 bucket is public S3 URLs. However, the major drawback of this approach is that the S3 bucket will be public. Public cloud resources are vulnerable to attacks, and avoiding public cloud resources is a good practice.

AWS API gateway will be a public interface for most endpoints. Many security methods can be used with AWS API gateway such as IAM, request throttling, and many more.

AWS Lambda can be integrated into the AWS API gateway. Images in an S3 bucket can be delivered using a Lambda. But then we have to develop and maintain an additional Lambda function. Also, there will be an extra cost for it.

When we use AWS API Gateway REST API as an S3 proxy, we can use security features in the AWS API gateway and we do not want to maintain an additional Lambda function. Also, we can keep our S3 bucket as a private resource.

## Use AWS API Gateway REST API as an S3 proxy

I explain the method using an example scenario. Also, I write this article with multiple steps for better understanding. Names and AWS regions can be changed according to the requirements. I use the new version of the AWS console (as of March 2024) in this article.

1. Create an IAM role for allowing API gateway to access S3 bucket objects
2. Create an S3 bucket and upload an image
3. Create an AWS API Gateway REST API
4. Create REST resource
5. Create REST method
6. Configure method request
7. Configure integration request
8. Configure integration response
9. Configure Method response
10. Enable binary support in API Gateway

## 1. Create an IAM role for allowing API gateway to access S3 bucket objects

1. Log into the AWS console
2. Navigate to `IAM` → `Roles`
3. Click on the `Create role` button
4. Select `Custom trust policy` as the trusted entity type
5. Enter the following policy as the custom trust policy
   ```
       {
           "Version": "2012-10-17",
           "Statement": [
               {
                   "Sid": "",
                   "Effect": "Allow",
                   "Principal": {
                       "Service": "apigateway.amazonaws.com"
                   },
                   "Action": "sts:AssumeRole"
               }
           ]
       }
   ```

6. Click on the `Next` button
7. Search for `AmazonS3ReadOnlyAccess` and select it
8. Then Click on the `Next` button
9. Enter a role name as `iam-test-role`
10. Then click on the `Create role` button

## 2. Create an S3 bucket and upload an image
1. Log into the AWS console
2. Navigate to `S3`
3. Click on the `Create bucket` button
4. Select `us-east-1` as the region
5. Enter `api-gw-image-test-s3` as the bucket name
6. Leave default values for other configurations
7. Click on the `Create bucket` button
8. Create a folder named `images`
9. Upload an image into the created folder

## 3. Create an AWS API Gateway REST API
1. Log into your AWS console
2. Navigate to `API Gateway` → `APIs`
3. Click on the `Create API` button
3. Click on the `Build` button on `Rest API` pane
4. Select `New API` and use `test-api` as the name
5. Leave default values for other configurations
6. Then click on the `Create API` button

## 4. Create REST resource
1. Click on the `Create resource` button
2. Create a resource and create REST resource `s3`

## 5. Create REST method
1. Click on the `Create method` button to create a method
2. Choose as following
    - Method type → GET
    - Integration type → AWS service
    - AWS Region → us-east-1
    - AWS service → Simple Storage Service (S3)
    - HTTP method → GET
    - Action type → Use path override
    - Path override → api-gw-image-test-s3/images/{image}
    - Execution role → arn of the created aws role in the step (1)
3. Leave the other options as default
4. Click on the `Create method` button

## 6. Configure method request
1. Select `Method request` tab pane
2. Click on `Edit` button
3. Expand `URL query string parameters`
4. Click on `Add query string` button
5. Enter `image` as the Name
6. Check `Required` button
7. Click on `Save` button

## 7. Configure integration request
1. Select `Integration request` tab pane
2. Click on `Edit` button
3. Select `When there are no templates defined (recommended)` as `Request body passthrough`
4. Expand `URL path parameters`
5. Click on `Add path parameter` button
6. Enter `image` as the name
7. Enter `method.request.querystring.image` as the path
8. Expand `URL request headers parameters`
9. Click on `Add query string parameter` button
10. Enter `Accept` as `Name`
11. Enter `'*/*'` (Value should be given with single quotes) as `Mapped from`
12. Click on `Save` button

## 8. Configure integration response
1. Select `Integration responses` tab pane
2. Delete `Default - Response`
3. Click on `Create response`
4. Enter `2\d{2}` as HTTP status regex
5. Click on `Create` button

## 9. Configure Method response
1. Select `Method responses` tab pane
2. Click on `Edit` button on `Response 200`
3. Click on `Add header` button
4. Enter `Content-Type` as `Header name`
5. Remove `Response body` items
6. Click on `Save` button
7. Then again select `Integration responses` tab pane
8. Click on `Edit` button
9. Enter `'*/*'` (Value should be given with single quotes) as `Mapping value` of the `Content-Type` response header
10. Click on `Save` button

## 10. Enable binary support in API Gateway
1. Navigate to `API settings`
2. Click on `Manage media type` button on `Binary media types`
3. Click on `Add binary media type` button
4. Enter `*/*` as `Binary media type`
5. Click on `Save changes`


## Finally,
1. Click on `Deploy API` and do an API Gateway deployment
2. Enter the invoked URI on the browser with the image query string (e.g.: https://aabhd7xr1z.execute-api.us-east-1.amazonaws.com/test/s3?image=porsche-911.jpg)
3. Ultimately, the Image is displayed in the browser

## Summary
In this article, we have discussed creating an AWS REST API as an Amazon S3 proxy and delivering images in an S3 bucket through the API gateway. We can use AWS API Gateway as a secure and cost-effective way to deliver S3 images.

## References
- https://docs.aws.amazon.com/apigateway/latest/developerguide/integrating-api-with-aws-services-s3.html
- https://serverlessland.com/patterns/apigateway-rest-s3-sam
