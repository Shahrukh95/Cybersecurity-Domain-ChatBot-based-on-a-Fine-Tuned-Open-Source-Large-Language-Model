# Cybersecurity Domain ChatBot based on a Fine-Tuned Open Source Large Language Model

The objective of this project was to develop an LLM powered chatbot that is fine-tuned with recent cybersecurity knowledge. Although new vulnerabilities are reported constantly and publicly by security experts, this project aims to determine the effectiveness of open-source LLMs in terms of learning the underlying cause and potential harm of such vulnerabilities if users are exposed to them. The methodology involves fine-tuning two open-source LLMs: Llama-2 and Falcon. The Falcon model is highly effective for question-answering tasks and outperforms several other models such as StableLM, RedPajama and MPT. The LLaMa-2 models have demonstrated even better performance and cost efficiency as compared to other open-source LLMs including Falcon itself, according to Meta AI's benchmarks. The models fine-tuned in this study were Falcon-7B and Llama-2-7b-chat-hf.

A dataset consisting of question-answer pairs referencing the latest publicly reported vulnerabilities in 2023 was required to fine-tune the models. Such a dataset was not found to be publicly available. Therefore, a novel database containing vulnerability information in the form of question-answer pairs was compiled using publicly available sources. These sources were the NVD and the OWASP 2023 Top 10 API and Mobile vulnerabilities. The dataset was made public and posted on HuggingFace. This was a significant contribution of this study in the LLM and cybersecurity space as the comprehensive dataset containing 19,135 rows can be used in future research involving fine-tuning strategies.

Each base model was finetuned on three different sets of hyperparameters to evaluate the performance. The evaluation was performed by comparing the model generated answers with true answers, using a GPT-3.5 Assistant. Consequently, the best performing models were then obtained for Falcon and Llama-2, from among the six fine-tuned models in total. Those models were highly effective in answering questions about the latest vulnerabilities described in the training data. Each question for evaluation was re-phrased before feeding to the model. Therefore, overfitting was not an issue in these models. The answers typically followed a certain format when prompted to describe a vulnerability in a specific domain such as Android or Windows. They started with a description followed by the scope, possible mitigation strategies and in some cases, the models were even able to give code examples of the vulnerability.

# Cybersecurity Dataset

## OWASP Top 10
The web page content of each of the [Top 10 API](https://owasp.org/API-Security/editions/2023/en/0x11-t10/) and [Mobile](https://owasp.org/www-project-mobile-top-10/2023-risks/) vulnerabilities were scraped. This content was fed into GPT-4 Turbo to convert it into question-answer pairs. A total of 273 question-answer pairs were obtained.

## NVD
This database contains data from the Common Vulnerabilities and Exposures (CVE) Program which uniquely identifies vulnerabilities with a CVE identifier (ID) and associates specific versions of code bases to those vulnerabilities. In this project, CVEs in four domains were chosen: Android, Databases, Windows and Web Servers. The chosen CVEs from the categories mentioned above were reported between October 2022 to December 2023. This ensured that the models were fine-tuned only on recent cybersecurity data. These were obtained using NVD's 2.0 API. The implementation is available in the "Data Generator" folder. 18,861 question-answer pairs were obtained.

# Validation Set
To assess the performance of the fine-tuned models, a validation set had to be created. This was constructed by randomly selecting 20% rows from each individual OWASP and NVD domains. The total number of rows in the validation set amounted to 3824. The validation set was constructed by taking question-answer pairs from the training set and subsequently rephrased using GPT-4 Turbo. This ensured that the evaluation was conducted on a dataset that the fine-tuned models had not seen during training.

# Hyperparameter Configuration
Fine-tuning LLMs requires testing different combinations of hyperparameters to achieve the best possible performance. For this project, 3 different sets of hyperparameters were tested. Each set was designed to test the performance of the model under varying training conditions. Both the Falcon and Llama-2 models were fine-tuned on each of the three sets. In total, six fine-tuned models were obtained in this study.

<img src="Images/hyperparameters.png" width="600" height="auto">

# Evaluation
The evaluation was performed in two steps. In the first step, the respective base models were assessed to determine the number of questions they can accurately answer using only their pre-trained knowledge. In the second step, the fine-tuned models were evaluated to determine the extent of information they learned from the dataset.

Each model was prompted with questions from the validation set to obtain the model generated answers or predicted answers. Model answers for all 3824 questions in the validation set were obtained for each model. The questions, true answers and the model answers were then passed to a GPT-3.5-turbo Assistant to obtain correctness (denoted by 0 or 1), confidence percentage and code generation capability. A GPT-4 model was not used for evaluation due to the sheer size of evaluations required, which was a costly endeavour. Six fine-tuned models and two base models were evaluated. Each item for evaluation comprised of the question, true answer and the model generated answer. Since the validation set contained 3824 items, the total number of evaluations performed were 30,592. A possible future work can be to evaluate the models with a better model such as GPT-4 or newer models which OpenAI will release. An example of an evaluated question-answer pair was as follows:

<img src="Images/values.png" width="700" height="auto">

All the evaluation steps including determining the validation accuracies and code generation capabilities are included in the Falcon-7B and Llama-2 folders. Each base and fine-tuned model was evaluated separately.

# Results
A comparison of the base and fine-tuned models indicated that the fine-tuned models indeed learned the information from the training dataset effectively. Evaluation between the fine-tuned Falcon and Llama-2 models demonstrated that the fine-tuned Falcon-7B model was able to generate a higher number of accurate (or technically correct) responses. However, Llama-2, specifically fine-tuned Llama-2-7b-chat-hf was able to generate a higher number of code examples for vulnerabilities. Since there was no large difference in validation accuracies between fine-tuned Falcon and Llama-2, both models look promising to be used in cybersecurity applications. For hands-on cybersecurity learning applications, this study suggests that Llama-2 could perform better than Falcon due its better code generation capability.

# Accessibility
The fine-tuned models and the dataset are made public on [HuggingFace](https://huggingface.co/shahrukh95). Feel free to access them and ask about vulnerabilities in Android, Databases, Windows and Web Servers from October 2022 to December 2023.

