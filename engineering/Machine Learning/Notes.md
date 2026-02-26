
---

Linguistic is the key of crafting effective prompts.

Best practices:

1. Give out clear instructions. 
2. Adopt persona - "Write the poem as Halena and loves the rookie style"
3. Specify the format
4. Avoid leading the answer
5. Limit the scope

Leave as less room for guess as possible

### Kinds of LLMs:

1. Generic language models: Predict the next word based on the language in the test data.
2. Instruction tuned models: They predict the response given some instruction. Eg: Give a python code ..
3. Diaologue tuned models: They take chain of dailogues and produce next response. Dialogue tuned are special case of Instruction tuned models.

Chain of thought reasoning:

It is based on philosphy that models are better getting the right answer when they first output text that explains the reason of the answer.

Tuning a model enables us to customize the model repsonse based on examples of tasks one want model to perform.
Fine tuning a model is customizing the model weights based on our use case.

Parameter efficient tuning only tunes small number of layers. 

Gemini is multimodel AI model meaning it can do more things apart from seeing the text etc.

## Bodhi 

Bodhi simplyfies the AI-ML pipelines. 
It is AI-ML platform to make your application go to scale. 

Gitops principle

Three areas - 

Bodhi core - core playform which enables to solve some problem
Functional core - some products - solution in box

Bodhi Core - complete bodhi is deployed in the kubernetes cluster. It can run on any cloud or on premise

It has - 

notebook servers for making the system
We can build ml pipelines in bodhi really easily
One can easily get the data from kafka stream.
Automatic ci/cd of models.

Automl  - already developed solution. 

Models can be treated like code (git workflow)
None 
Staged
Production

Multiple notebooks can be connected to form pipelines using drag and drop
