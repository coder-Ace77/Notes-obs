
---

```python
import openai
import os
from dotenv import load_dotenv
from openai import OpenAI

load_dotenv()
OPENAI_API_KEY=os.getenv('OPEN_AI_KEY')
client = OpenAI(api_key=OPENAI_API_KEY)
MODEL="gpt-5-nano"

def log(s):
    with open("log.txt",'a') as f:
        f.write(s)

# observe that its doing essentially completion
def get_response(req):
    res = client.chat.completions.create(
        model=MODEL,
        messages=[
            {
                "role":"user",
                "content":req,
            }
        ]
    )
    log("MODEL:"+res.model+" "+"TOKEN USED: "+str(res.usage.completion_tokens)+"\n")
    return res.choices[0].message.content

def sentiment_main():
    with open("doc.txt") as f:
        doc = f.read()
        print("OPENING DOC:")
        print(f"doc :: {doc}")
        print("SENTIMENT::"+sentiment_analysis(doc))

  

# code to store history and give responses
def main():
    history=""
    prev_question = []
    prev_answer = []
    while True:
        inp = input("\nYOU:")
        if inp.strip().lower=="exit":
            break
        for q,a in zip(prev_question,prev_answer):
            history+=f"User: {q} , Chat gpt: {a}"
        history+=f"User:{inp}"
        res = get_response(history)
        print(f"\nGpt: {res}")
        prev_question.append(inp)
        prev_answer.append(res)
```

#### Applications:

The sentiment analysis means to find the tone of a document or a piece of text. We can simply provide the string to gpt along with the request to give the response. 

To get better responses one fundamental thing we need to understand is that its gpt - which is generative pretrained transformer. Its basic task is to guess the next words best to its knowledge. So it will give better response if we give it request ending with say

`doc-- The sentiment is-`
This means it will give better sentiment analysis.

```python
def sentiment_analysis(doc):
    prompt = f"{doc} . The sentiment is."
    return client.chat.completions.create(
        model=MODEL,
        messages=[
            {
                "role":"user",
                "content":prompt
            }
        ]).choices[0].message.content
```

We can also use this trick to do summarizations as well 
Finally we can also forbid the application to generate some result by checking the response for a given word or not. 

```pseudo
res = ...
for word in res.words():
	if(word in black_List_tokens):
		// do a replace or something
```

Finally we can also compare the cosine similarity of two diffenent sentences to find if there similar or not.

```python
def cosine_similarity(vec1,vec2):
    overlap = np.dot(vec1,vec2)
    mag1 = np.linalg.norm(vec1)
    mag2 = np.linalg.norm(vec2)
    cos_sim = overlap/(mag1*mag2)
    return cos_sim

def is_relevant(response,input,threshold=0.5):
    vectorized_input = spacy_model(input).vector
    vectorized_response = spacy_model(response).vector
    similarity = cosine_similarity(vectorized_input,vectorized_response)
    return similarity>=threshold
```



