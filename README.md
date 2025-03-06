# Amazon-Bedrock-with-the-AWS-SDK-for-Python

# step1 
importing required library

boto3 for accesing bedrock api
json to convert response return from the llm model

# step2
connecting bedrock with boto3 

# step3 
assigning model Id

# step4
Giving prompt

# step 5
Providing Payload to define model behaviour 
for example: temprature, top p , top k, stopping sequence
eaxh model have differnet payload

# step 6
Model invokation using .ivoke() function 

# step 7
extracting response from the bedrock using json.read() to get output from  the model

# step 8
changing model to compare output, and examine the perfomance in terms of time and generated output

# output analysis

while using amazon titan express,
it responded in 10-15 seconds, and generated output was incomplete

while using amazon titan lite
Response: Sorry - this model is unable to respond to this request.

