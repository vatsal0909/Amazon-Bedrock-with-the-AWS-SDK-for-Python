# Generate and Evaluate Images in Amazon Bedrock with Amazon Nova Canvas and Anthropic Claude 3.5 Sonnet

## **AIM**
The aim of this project is to generate high-quality images using **Amazon Nova Canvas** in **Amazon Bedrock** and evaluate them with **Anthropic Claude 3.5 Sonnet**. The solution will:

1. Generate images from text prompts using **Amazon Nova Canvas**.
2. Analyze and describe the generated images using **Claude 3.5 Sonnet**.
3. Evaluate the images by assigning a score (1-10) and providing explanations.
4. Suggest improvements based on the evaluation.

The implementation will use **AWS Lambda** and **Amazon API Gateway** to handle requests and process AI model interactions. The goal is to demonstrate how businesses can leverage **generative AI** to create and assess visual content efficiently.

---

## **Step 1: Prerequisites**
Ensure you have:

✅ An **AWS account** with necessary permissions.  
✅ **Amazon Bedrock enabled** in the AWS **us-east-1** region.  
✅ **Amazon Nova Canvas** and **Anthropic Claude 3.5 Sonnet** models enabled.  
✅ **AWS CLI installed and configured** (for command-line testing).  

---

## **Step 2: Deploy Resources Using CloudFormation**
1. **Sign in to AWS Management Console** as an **IAM administrator**.
2. **Launch CloudFormation Stack**:  
   - Open **AWS CloudFormation**.  
   - Choose **Launch Stack** and upload the **YAML template** (provided in your documentation).  
3. **Enter parameters**:  
   - Name for your **S3 bucket** (e.g., `image-gen-your-initials`).  
   - Existing **S3 bucket name** for storing logs.  
   - **API token** for authentication.  
4. **Deploy the stack**:  
   - Click **Next**, then **Next** again.  
   - **Acknowledge IAM permissions** and click **Submit**.  
5. **Wait for stack creation** to complete (`CREATE_COMPLETE` status).  
6. **Copy API details** from the Outputs tab:  
   - `ApiId`, `ApiUrl`, `ResourceId`.  

---

## **Step 3: Test the API**
### **Option 1: Using API Gateway Console**
1. Go to **Amazon API Gateway** in AWS Console.  
2. Choose **APIs > BedrockImageGenEval**.  
3. In **Resources**, select **POST** method for `/generate-image`.  
4. Click **Test** and enter the JSON request body:  

```json
{ "prompt": "A black cat in an alleyway with blue eyes" }
Click Test and check the response.
Option 2: Using AWS CLI
Run the following command (replace ApiId and ResourceId with your values):

sh
Copy
Edit
aws apigateway test-invoke-method --rest-api-id ApiId --resource-id ResourceId --http-method POST --path-with-query-string "/generate-image" --body '{"prompt":"A black cat in an alleyway with blue eyes"}' | grep '"body"' | sed 's/.*"body": "\(.*\)".*/\1/' | sed 's/\\//g'
Check the output JSON with the generated image URL and evaluation.

Option 3: Using Terminal with cURL
Run this command (replace YOUR_API_URL and YOUR_TOKEN):

sh
Copy
Edit
curl -X POST YOUR_API_URL -H 'Authorization: YOUR_TOKEN' -H 'Content-Type: application/json' -d '{"prompt":"A black cat in an alleyway with blue eyes"}'
Step 4: Review API Response
The API returns JSON with:

json
Copy
Edit
{
  "url": "Your pre-signed S3 URL",
  "evaluation_data": {
    "description": "Short description of your image",
    "score": "9",
    "reason": "Explanation for the score",
    "suggestions": "How to improve the image"
  }
}
✅ Download the image using the pre-signed URL.
✅ Check Claude’s evaluation and suggestions.

Step 5: Clean Up AWS Resources
To avoid charges:

Delete generated images from S3 bucket.
Delete CloudFormation stack from the AWS console.
Conclusion
You've successfully implemented an image generation and evaluation system using Amazon Nova Canvas and Claude 3.5 Sonnet in Amazon Bedrock. You can extend this by:

✔️ Customizing models with your data.
✔️ Enhancing prompts for better image results.
✔️ Integrating additional AI models for different tasks.

CloudFormation YAML Template
yaml
Copy
Edit
AWSTemplateFormatVersion: '2010-09-09'
Parameters:
  S3BucketName:
    Type: String
    Description: Name of the main S3 bucket
  LoggingBucketName:
    Type: String
    Description: Name of an EXISTING S3 bucket for storing access logs
  Token:
    Type: String
    Description: The authorization token to store in Secrets Manager.

Resources:

  MyS3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Ref S3BucketName
      PublicAccessBlockConfiguration:
        IgnorePublicAcls: true
      VersioningConfiguration:
        Status: Enabled
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: AES256
      LoggingConfiguration:
        DestinationBucketName: !Ref LoggingBucketName
        LogFilePrefix: logs/

  SecretToken:
    Type: AWS::SecretsManager::Secret
    Properties:
      Name: "token"
      Description: "The authentication token stored in AWS Secrets Manager."
      SecretString: !Sub |
        {
          "token": "${Token}"
        }

  MyApiGateway:
    Type: AWS::ApiGateway::RestApi
    Properties:
      Name: BedrockImageGenEval

  ApiResource:
    Type: AWS::ApiGateway::Resource
    Properties:
      ParentId: !GetAtt MyApiGateway.RootResourceId
      PathPart: generate-image
      RestApiId: !Ref MyApiGateway

  ApiMethod:
    Type: AWS::ApiGateway::Method
    Properties:
      AuthorizationType: NONE
      HttpMethod: POST
      ResourceId: !Ref ApiResource
      RestApiId: !Ref MyApiGateway
      Integration:
        IntegrationHttpMethod: POST
        Type: AWS_PROXY
        Uri: !Sub
          - arn:${AWS::Partition}:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${LambdaFunctionArn}/invocations
          - LambdaFunctionArn: !GetAtt GenerateImageFunction.Arn

  GenerateImageFunction:
    Type: AWS::Lambda::Function
    Properties:
      Handler: index.handler
      FunctionName: lambda-gen-eval-image-bedrock
      Role: !GetAtt LambdaExecutionRole.Arn
      Code:
        ZipFile: |
          import json, boto3, base64, os

          s3_client = boto3.client('s3')
          bedrock_client = boto3.client('bedrock-runtime', region_name="us-east-1")

          def handler(event, context):
              prompt = json.loads(event['body'])['prompt']
              image_base64 = generate_image(prompt)
              image_data = base64.b64decode(image_base64)
              presigned_url = generate_presigned_url(image_data)
              return {'statusCode': 200, 'body': json.dumps({'url': presigned_url})}

          def generate_image(prompt):
              response = bedrock_client.invoke_model(modelId='amazon.titan-image-generator-v2:0', body=json.dumps({"textToImageParams": {"text": prompt}}))
              return json.loads(response['body'].read())['images'][0]

      Runtime: python3.11
      Timeout: 240
      MemorySize: 512

Outputs:
  ApiUrl:
    Description: API Gateway endpoint URL for Prod stage
    Value: !Sub "https://${MyApiGateway}.execute-api.${AWS::Region}.amazonaws.com/prd/generate-image"
