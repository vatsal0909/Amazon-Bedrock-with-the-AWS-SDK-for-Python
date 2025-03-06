# Building Generative AI Applications on Amazon Bedrock with Boto3

## Overview
Amazon Bedrock is a fully managed service that provides access to multiple Foundation Models (FMs) from AI leaders like AI21 Labs, Anthropic, Cohere, Meta, Mistral AI, Stability AI, and Amazon via a single API. It allows developers to:
- Experiment with different FMs.
- Customize models using fine-tuning and Retrieval Augmented Generation (RAG).
- Build generative AI applications without managing infrastructure.

This guide demonstrates how to use Amazon Bedrock with the AWS SDK for Python (Boto3) to interact with models programmatically.

---
## Solution Architecture
The implementation follows these key steps:
1. **Set up AWS Boto3 Client** to interact with Amazon Bedrock.
2. **Define the Model ID** – In this example, Anthropic’s Claude 3 Sonnet.
3. **Prepare a Prompt** to send user input to the model.
4. **Create a Payload** with parameters to control model behavior.
5. **Invoke the Model** using the `invoke_model` API.
6. **Process and Display the Response**.

---
## Prerequisites
Before running the script, ensure you have:
- **An AWS Account** with Amazon Bedrock access.
- **AWS CLI Installed & Configured**.
- **An IAM User** with permissions for Amazon Bedrock.
- **Boto3 Installed** (`pip install boto3`).
- **Python 3.8+** installed and configured.

---
## Key Components
### 1. Setting Up the Boto3 Client
```python
import boto3

bedrock_client = boto3.client(
    service_name="bedrock-runtime",
    region_name="us-east-1"
)
```
- Creates an Amazon Bedrock runtime client using Boto3.
- Specifies the AWS region (`us-east-1`).

### 2. Defining the Model ID
```python
model_id = "anthropic.claude-3-sonnet-20240229-v1:0"
```
- Specifies the model to use (Claude 3 Sonnet).

### 3. Creating the Prompt
```python
prompt = "Hello, how are you?"
```
- Defines the input query for the AI model.

### 4. Structuring the Payload
```python
payload = {
    "anthropic_version": "bedrock-2023-05-31",
    "max_tokens": 2048,
    "temperature": 0.9,
    "top_k": 250,
    "top_p": 1,
    "messages": [
        {
            "role": "user",
            "content": [
                {"type": "text", "text": prompt}
            ]
        }
    ]
}
```
- **`anthropic_version`**: Specifies Bedrock’s version.
- **`max_tokens`**: Limits response length.
- **`temperature`**: Controls creativity (higher = more creative).
- **`top_k`**: Limits the number of top choices for each token.
- **`top_p`**: Adjusts probability distribution.
- **`messages`**: Contains the user query.

### 5. Invoking the Model
```python
response = bedrock_client.invoke_model(
    modelId=model_id,
    body=json.dumps(payload)
)
```
- Sends the request to Amazon Bedrock with the defined model and payload.

### 6. Processing and Displaying the Response
```python
result = json.loads(response["body"].read())
generated_text = "".join([output["text"] for output in result["content"]])
print(f"Response: {generated_text}")
```
- Extracts the generated response from Amazon Bedrock.
- Prints the final output.

---
## Example Use Cases
Amazon Bedrock and Boto3 can be used for:
- **Text Generation** (creative writing, scripts, poetry).
- **Code Completion** (suggesting and optimizing code snippets).
- **Data Summarization** (extracting key insights from large datasets).
- **Conversational AI** (chatbots, virtual assistants).

---
## Clean Up
To avoid unnecessary charges:
- **Delete IAM users/roles** if created temporarily.
- **Monitor AWS CloudWatch logs** and remove old logs.

For pricing details, visit [Amazon Bedrock Pricing](https://aws.amazon.com/bedrock/pricing/).

---
## Conclusion
This guide demonstrated how to:
- Set up Amazon Bedrock with Boto3.
- Define and invoke an FM (Claude 3 Sonnet).
- Process responses for text generation.

Explore more FMs in Amazon Bedrock to find the best fit for your application!



# Comparing Model Responses in AWS Bedrock Using Boto3

## Overview
This document explains the steps taken to invoke **Amazon Bedrock** models using `boto3`, experiment with different prompts, and compare the responses of two different **Titan** models.

---

## 1️⃣ Connecting to Bedrock Runtime
- Established a connection with **AWS Bedrock Runtime** using `boto3`.
- Configured the client with the appropriate service name and region.

---

## 2️⃣ Invoking `amazon.titan-text-express-v1`

### **Prompt 1: General Conversation**
- Sent a basic prompt: **"Hello, how are you?"**
- Defined a payload specifying:
  - The input text.
  - Model parameters such as **max token count, temperature, and topP**.
- Received a response where the model provided a **detailed explanation of empathy**, instead of answering conversationally.

### **Prompt 2: C++ Bubble Sort Algorithm**
- Changed the prompt to **"Write me code to perform bubble sort algorithm in C++ language?"**.
- Modified the payload to:
  - Lower the **temperature** for more deterministic output.
  - Keep **topP** at 1.
- The model successfully generated **valid C++ code for the Bubble Sort algorithm**.

---

## 3️⃣ Comparing with `amazon.titan-text-lite-v1`
- Used the same prompt **"Write me code to perform bubble sort algorithm in C++ language?"** with a different model.
- Adjusted the payload by setting the **temperature to 0** for a deterministic response.
- The model **refused to generate the response**, instead returning a message indicating it could not process the request.

---

## 4️⃣ Observations & Conclusion

| Model | Prompt | Response |
|--------|-------------------------|----------------------------|
| **Titan-Express** | "Hello, how are you?" | Provided a detailed explanation of empathy. |
| **Titan-Express** | C++ Bubble Sort | Successfully generated valid C++ code. |
| **Titan-Lite** | C++ Bubble Sort | Refused to generate a response. |

### **Key Takeaways**
- **Titan-Express performed better** in both general conversation and code generation.
- **Titan-Lite appears to have stricter content policies** or **fewer capabilities**.
- Adjusting model parameters like **temperature and topP** can impact response quality.

---

## Next Steps
- Test additional models like **Claude, Llama2, and Jurassic** for a broader comparison.
- Experiment with different **temperature, topP, and max token count** values to analyze output variations.
- Consider using **Bedrock Knowledge Bases** for domain-specific, fine-tuned responses.

---

📌 **This document helps understand how different models behave with the same input.** Let me know if further refinements or experiments are needed! 🚀
