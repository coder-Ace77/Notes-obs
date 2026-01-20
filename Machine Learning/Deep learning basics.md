
---

A subfield of machine learning using **artificial neural networks** with multiple layers to automatically learn **representations and patterns** from data.

### Artificial Neural Networks (ANN)

- **Definition:** The foundational architecture of deep learning, inspired by the human brain.
- **Structure:**
    - **Input layer:** Receives raw data features.
    - **Hidden layers:** Transform input features through weighted connections and activation functions.
    - **Output layer:** Produces predictions.
- **Key Components:** Neurons, weights, biases, activation functions, loss function.
- **Use Cases:** Regression, classification, basic prediction tasks.

### Recurrent Neural Networks (RNN)

- **Definition:** Neural networks specialized for **sequential data** where current output depends on previous inputs.
- **Key Feature:** Maintains a **hidden state** to remember past information.
- **Variants:**
    - LSTM (Long Short-Term Memory) → handles long-term dependencies.
    - GRU (Gated Recurrent Unit) → simpler and faster than LSTM.
- **Applications:** Time series forecasting, speech recognition, text generation.

## **Convolutional Neural Networks (CNN)**

- **Definition:** Neural networks designed for **grid-like data**, primarily images.
- **Key Feature:** Uses **convolutions and pooling** to automatically extract spatial hierarchies of features.
- **Components:** Convolution layers, pooling layers, fully connected layers.
- **Applications:** Image classification, object detection, medical image analysis.


## **Transformers & Attention Mechanism**

- **Transformers:** Neural network architecture designed for **sequence modeling** without relying on recurrence.
- **Attention Mechanism:** Allows the model to **focus on important parts of the input** when making predictions.
- **Advantages:** Parallelizable, captures long-range dependencies efficiently.
- **Applications:** NLP (translation, summarization), speech processing, multimodal tasks.

## **Foundational Models**

- **Definition:** Large-scale pre-trained models that form the backbone for various AI applications.
- **Examples:**
    
    - **GPTs (Generative Pre-trained Transformers):** Generate human-like text, perform reasoning, code generation.
    - **LLMs (Large Language Models):** Massive transformer-based models trained on huge text corpora to understand and generate natural language.


An LLM is feeded piece of text called prompt. Output of llm is called completion or response. Prompt is divided and converted into numbers and feeded to model. These numbers are called tokens and peice of code that performs this task is called tokenizer also the set of all tokens is called vocabulary of the language model.

Context window is the maximum number of tokens an llm can take at once and generate output.

LLMs are trained on the masisve datasets by self supervised learning. A type of **unsupervised learning** where the model generates its **own supervision signal** (labels) from the input data itself.

Analogy: It’s like giving a student a text with missing words and asking them to fill in the blanks—they learn the structure of language by predicting the missing parts.

LLMs (like GPT, BERT) are trained on **massive text corpora** using tasks where the **input and labels come from the text itself**.

In gpts task is to predict the next word in the model. So a piece of text can be converted to feature and label by just adding next word in feature. Now model is porvided to 
However in BERT task is to predict the randomly masked word in the sentence.

### **Fine-Tuning: Teaching Models with New Knowledge

Fine-tuning is a process in which a pre-trained model, such as GPT or BERT, is further trained on a smaller, domain-specific dataset to adapt it to a particular task or field. The base model has already learned general patterns of language from massive text corpora during its initial training phase (called pretraining). However, it may not perform optimally on specialized data, such as legal documents, scientific research, or company-specific chat logs. Fine-tuning allows developers to refine the model’s internal weights by exposing it to examples that better reflect the target use case.

During fine-tuning, the training is usually done with a lower learning rate and fewer epochs compared to pretraining. The idea is not to “relearn” language but to “adapt” the already learned patterns to a narrower domain. For example, a general GPT model might know what “heart rate” means, but fine-tuning it on medical texts would help it use the term accurately in a clinical sense. Fine-tuning can be **supervised** (using labeled data, e.g., input-output pairs for classification or question answering) or **instruction-based**, where the model learns from prompt-response examples.

The main advantage of fine-tuning is **precision** — the model becomes more accurate, consistent, and aligned with the tone or terminology of the desired context. However, it comes with **trade-offs**: it requires access to high-quality, representative data and computational resources for retraining. Moreover, fine-tuned models can become less general and may lose some flexibility if over-specialized. Despite this, fine-tuning remains a powerful approach when long-term, stable customization of an LLM is required.

---

### **RAG (Retrieval-Augmented Generation): Blending Knowledge with Context**

Retrieval-Augmented Generation (RAG) takes a different approach to improving model performance — instead of retraining or altering the model’s parameters, it **augments the model with external knowledge retrieval** at inference time. In simpler terms, when a user asks a question, RAG systems first fetch relevant documents or data from a knowledge base (like PDFs, databases, or web pages) and then feed that retrieved content to the model as context before generating a response. This allows the model to produce **up-to-date, factual, and source-grounded** answers without requiring retraining.

A RAG pipeline typically consists of two main components: a **retriever** and a **generator**. The retriever searches through a vector database (created using embeddings of text chunks) to find information relevant to the query. Then, the generator — usually a large language model — uses the retrieved text along with the prompt to generate the final response. For instance, if a user asks, “What are the safety guidelines for lithium-ion batteries?”, the retriever might pull sentences from technical manuals or internal company documents, and the LLM will synthesize them into a coherent, well-phrased answer.

The biggest strength of RAG lies in **dynamic adaptability** — it can instantly access new or updated information without retraining the model itself. This makes it ideal for enterprise use cases like document-based question answering, customer support bots, or internal knowledge assistants. However, it also introduces challenges such as ensuring the retriever fetches the most relevant and trustworthy data, managing hallucinations, and maintaining retrieval speed.