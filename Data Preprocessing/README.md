# Methodology

The validation set was created by taking 20% of question-answer pairs from each category of the training set (NVD + OWASP).
These pairs were then re-phrased by gpt-3.5-turbo to create same pairs but worded differently.
These pairs will be used to evaluate the fine-tuned models through LLM-as-a-judge method.