# Cybersecurity Domain ChatBot based on a Fine-Tuned Open Source Large Language Model

The objective of this project was to develop an LLM powered chatbot that is fine-tuned with recent cybersecurity knowledge. Although new vulnerabilities are reported constantly and publicly by security experts f.e. as documented in the National Vulnerability Database (NVD), this project aims to determine the effectiveness of open-source LLMs in terms of learning the underlying cause and potential harm of such vulnerabilities if users are exposed to them.

The methodology involves fine-tuning two open-source LLMs: Llama-2 and Falcon. The Falcon model is highly effective for question-answering tasks and outperforms several other models such as StableLM, RedPajama and MPT. The LLaMa-2 models have demonstrated even better performance and cost efficiency as compared to other opensource LLMs including Falcon itself, according to Meta AI's benchmarks. The models fine-tuned in this study were Falcon-7B and Llama-2-7b-chat-hf.

A dataset consisting of question-answer pairs referencing the latest publicly reported vulnerabilities in 2023 was required to fine-tune the models. Such a dataset was not found to be publicly available. Therefore, a novel database containing vulnerability information in the form of question-answer pairs was compiled using publicly available sources. These sources were the NVD and the OWASP 2023 Top 10 API and Mobile vulnerabilities. The dataset was made public and posted on HuggingFace. This was a significant contribution of this study in the LLM and cybersecurity space as the comprehensive dataset containing 19,135 rows can be used in future research involving fine-tuning strategies.

Each base model was finetuned on three different sets of hyperparameters to evaluate the performance. The evaluation was performed by comparing the model generated answers with true answers, using a GPT-3.5 Assistant. Consequently, the best performing models were then obtained for Falcon and Llama-2, from among the six fine-tuned models in total. Those models were highly effective in answering questions about the latest vulnerabilities described in the training data. Each question for evaluation was re-phrased before feeding to the model. Therefore, overfitting was not an issue in these models. The answers typically followed a certain format when prompted to describe a vulnerability in a specific domain such as Android or Windows. They started with a description followed by the scope, possible mitigation strategies and in some cases, the models were even able to give code examples of the vulnerability.

# Cybersecurity Training Dataset

## OWASP Top 10
THe web page content of Top 10 API and Mobile vulnerabilities were scraped. The raw data was fed into GPT-4 Turbo to convert the raw content into question-answer pairs. The following vulnerabilites were collected:

<img src="Images/API Vulnerabilities.png" width="500" height="auto">

<img src="Images/Mobile Vulnerabilities.png" width="500" height="auto">
