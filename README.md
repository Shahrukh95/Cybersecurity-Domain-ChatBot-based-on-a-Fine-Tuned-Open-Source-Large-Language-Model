# Cybersecurity Domain ChatBot based on a Fine-Tuned Open Source Large Language Model
LLaMA 2-7B and Falcon-7B were fine-tuned on a cybersecurity-specific dataset to evaluate whether these models can effectively answer questions in the cybersecurity domain. The dataset was manually curated from Common Vulnerabilities and Exposures (CVE) records in the National Vulnerability Database (NVD) and resources from OWASP. The fine-tuning process utilized QLoRA with 4-bit quantization, optimizing the models for efficient inference while maintaining accuracy.

# Quick Use
To quickly run the fine-tuned models available on HuggingFace, you can load the model locally or on Google Colab and start making inferences. The following example demonstrates how to load one of the models with QLoRA:

```python
from transformers import AutoTokenizer, AutoModelForCausalLM, BitsAndBytesConfig
from peft import PeftModel, PeftConfig
import torch

# Define model path
PEFT_MODEL = "shahrukh95/falcon-7b-Set-3-cybersecurity-layered-config"

# Load PEFT configuration
config = PeftConfig.from_pretrained(PEFT_MODEL)

# Define quantization configuration for QLoRA
bnb_config = BitsAndBytesConfig(
    load_in_4bit=True,  # Load model weights in 4-bit precision (saves memory)
    bnb_4bit_use_double_quant=True,  # Enable double quantization (extra memory savings)
    bnb_4bit_quant_type="nf4",  # Use Normalized Float 4 (better for language models)
    bnb_4bit_compute_dtype=torch.float16  # Perform computations in 16-bit precision for accuracy
)

# Load the base model with quantization
peft_base_model = AutoModelForCausalLM.from_pretrained(
    config.base_model_name_or_path,  # Load the base model (Falcon-7B)
    return_dict=True,
    quantization_config=bnb_config,  # Apply quantization config
    device_map="auto", # Automatically map model layers to available devices (i.e. GPU)
    trust_remote_code=True # Enable custom model architectures
)
```
Load the QLoRA adapter and tokenizer

```python
# Load the PEFT (QLoRA) adapter
model = PeftModel.from_pretrained(peft_base_model, PEFT_MODEL)

# Load the tokenizer
tokenizer = AutoTokenizer.from_pretrained(PEFT_MODEL)
# Set pad_token_id to eos_token_id
if tokenizer.pad_token_id is None:
    tokenizer.pad_token_id = tokenizer.eos_token_id

# Move the model to GPU
device = "cuda" if torch.cuda.is_available() else "cpu"
model = model.to(device)
```

Make a pipeline for generating response

```python
# Response generator
def generate_response(prompt, max_length=256):
    inputs = tokenizer(prompt, return_tensors="pt").to(device)
    with torch.no_grad():
        outputs = model.generate(
          **inputs,
          max_new_tokens=256,  # Limit the output length for QA tasks
          eos_token_id=tokenizer.eos_token_id,  # Explicitly set the stopping point
          pad_token_id=tokenizer.pad_token_id,  # Explicitly include pad token
          do_sample=False  # Use deterministic greedy decoding for QA tasks. No need for temp, top_k and top_p.
      )

    response = tokenizer.decode(outputs[0], skip_special_tokens=True)
    return response
```

Ask a cybersecurity related question preferably in the domains mentioned in the **Dataset** section below.

```python
question = "What are the potential consequences of exploiting CVE-2023-29351?"
print("Question:", question)
answer = generate_response(question)
print("Answer:", answer)
```

# System Requirements
- **GPU VRAM:** 6 GB
- **System RAM:** 4.1 GB
- **Disk Usage:** 45.7 GB


# Dataset
The NVD (National Vulnerability Database) provides an API to collect CVEs (Common Vulnerabilities and Exposures). The **Data Generator** folder contains the code to collect CVEs within a specific range of dates (October 2022 - December 2023). Vulnerabilities from OWASP were added to the dataset manually because there are only 10 vulnerabilities listed for a given domain.

The raw text of all collected vulnerabilities was converted into a question-answer pair format using GPT-4 Turbo.

The training dataset is available in the file: ```Datasets/Training Set.xlsx```
Its structure is as follows:

# Validation Set
The validation set was created by sampling 20% of the question-answer pairs from each category of the training set (NVD + OWASP). These pairs were then rephrased using GPT-3.5 Turbo to create semantically identical but syntactically different question-answer pairs. The validation set was used to evaluate the fine-tuned models through the LLM-as-a-judge method.

The validation dataset is available at: ```Datasets/Validation Set.json```



# Hyperparameter Configuration
The hyperparameters were configured as follows:

## Hyperparameter Configuration

| Parameter                     | Set 1 (Baseline Configuration) | Set 2 (Gradual Fine-tuning) | Set 3 (Aggressive Learning) |
|--------------------------------|-------------------------------|-----------------------------|-----------------------------|
| num_train_epochs              | 15                            | 20                          | 12                          |
| per_device_train_batch_size   | 8                             | 6                           | 10                          |
| learning_rate                 | 3e-4                          | 2.5e-4                      | 5e-4                        |
| weight_decay                  | 0                             | 0.0005                      | 0.0001                      |
| max_grad_norm                 | 1                             | 1                           | 0.8                         |
| lr_scheduler_type             | constant                      | linear                      | cosine                      |

Both models used the same configurations, resulting in a total of six fine-tuned models with slight variations in performance.


# Evaluation
The evaluation was performed in two steps. First, the respective base models were assessed to determine how many questions they could accurately answer using only their pre-trained knowledge. Second, the fine-tuned models were evaluated to measure the extent of information learned from the dataset. The evaluation utilized the LLM-as-a-judge method with GPT-3.5 Turbo, which compared the similarity of answers from the validation set to those generated by the model.

```json
{
    "Question": "What are the potential consequences of exploiting CVE-2023-29351?",
    "Model Answer": "The exploitation of CVE-2023-29351 could lead to an attacker gaining elevated privileges on a system by exploiting a flaw in the Windows Group Policy. This may result in unauthorized actions being performed, access to sensitive information, or the ability to enforce unwanted system policies, leading to a variety of malicious activities. It is important for administrators to apply security updates promptly to mitigate the risks associated with this vulnerability.\n\nReference:\n- https://msrc.microsoft.com/update-guide/vulnerability/CVE-2023-29351",
    "True Answer": "The exploitation of CVE-2023-29351 could result in an attacker gaining elevated privileges on a system. This might allow them to execute arbitrary code, access sensitive data, install programs, or create new accounts with full user rights, leading to a variety of malicious activities.",
    "GPT-3.5 Validation": "1",
    "Confidence Percentage": "100",
    "Asked for Code in Question": "No",
    "Code in Answer": "No"
}
```

# Usage
Fine-tuning and evaluation were performed in the same Jupyter notebook files:

```
Falcon-7B\Base Falcon 7-B\Falcon-7B-base-inference.ipynb
Falcon-7B\Fine-tuning Falcon-7B\Baseline Configuration (Hyperparameter Set 1)\main.ipynb
Falcon-7B\Fine-tuning Falcon-7B\Gradual Fine-tuning (Hyperparameter Set 2)\main.ipynb
Falcon-7B\Fine-tuning Falcon-7B\Aggressive Learning (Hyperparameter Set 3)\main.ipynb
```

```
Llama-2-7B\Base Llama-2-7B\Llama-2-7B-base-inference.ipynb
Llama-2-7B\Fine-tuning Llama-2-7B\Baseline Configuration (Hyperparameter Set 1)\llama-2.ipynb
Llama-2-7B\Fine-tuning Llama-2-7B\Gradual Fine-tuning (Hyperparameter Set 2)\llama-2.ipynb
Llama-2-7B\Fine-tuning Llama-2-7B\Aggressive Learning (Hyperparameter Set 3)\llama-2.ipynb
```


# Results
A decent improvement in the knowledge of the fine-tuned models was observed. However, the models exhibited frequent hallucinations and answers occasionally failed to end at logical stopping points even with deterministic token decoding (as opposed to random sampling). The models often continued generating text until the token limit was reached.

# Personal Recommendation
A RAG (Retrieval-Augmented Generation) based approach may be more suitable for this use case. Fine-tuning to increase the base knowledge of a model would require a very large and high quality dataset, especially since CVEs are reported continuously. A strong pre-trained model with RAG-based access to recent vulnerabilities would likely be more effective.

# Accessibility
The fine-tuned models and datasets are publicly available on [HuggingFace](https://huggingface.co/shahrukh95). Feel free to explore them and ask about vulnerabilities in Android, Databases, Windows, and Web Servers from October 2022 to December 2023.
