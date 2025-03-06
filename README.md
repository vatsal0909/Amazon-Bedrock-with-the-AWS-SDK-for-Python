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
