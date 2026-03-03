
---

Defineitons:

1. Model - It is the algorithm used as the black box to get the output provided any input.
2. Features are the **input variables** or attributes that are used by a machine learning model to make predictions.
3. Labels are the **output variable** or the **target** that the model is trying to predict.

Mathematically a model is just a function or a transformation which takes one input and gives out one output
we can write 

`y = f(x)` 

Example is single instance of data and can be of two types Labelled and unlabbeled(label on present)

**Training a model** is the process where a machine learning algorithm **learns patterns from data** so that it can make predictions on new, unseen data.

- In supervised learning: The model learns the relationship between **features (inputs)** and **labels (outputs)**.
- In unsupervised learning: The model learns patterns or structures in the data (like clusters).

In training process we have a model which has a algorithm aka a function and also has a set of weigths which are the variables.

A model takes input and weights to give output

```
y = f(x,wt) 
```

Weights are held by the model usually and can be updated. 

In training, model starts with random label and generates an output. The output is then compared with the label. To figure out how good the answer is we define `loss`. A **loss function** (also called cost function) measures **how well the model's predictions match the true labels**. Once a loss is find out model weigths are adjusted to reduce loss. And this is done for some iterations and is called ipoch.

Accuracy is the **fraction of correct predictions** made by the model.

Gradients are computed on the loss to figure out in which direction weights have to updated and by what amount. Once that is done we again repeat this cycle.

If there are very small number of examples or the model is not flexible enough to retain the data we get to something called underfitting. **Overfitting** occurs when a machine learning model **learns the training data too well**, including its **noise and random fluctuations**, instead of learning the underlying pattern.

**Inference** (also called **prediction**) is the process of using a **trained machine learning model** to make predictions on **new, unseen data**.

## Linear regression

**Linear Regression** is a **supervised machine learning algorithm** used to predict a **continuous numerical value** based on one or more input features.

- It assumes a **linear relationship** between the input(s) and the output.
- Formula (for one feature / simple linear regression):
		y=b0+b1⋅xy = b_0 + b_1 \cdot xy=b0​+b1​⋅x
- **Training data:** Collect pairs of features (x) and labels (y).
- **Find the best-fitting line:**
    - Choose b0b_0b0​ and b1b_1b1​ that **minimize the difference** between predicted y and actual y.
    - Usually done using **Mean Squared Error (MSE)**.
- **Prediction:** Use the learned line equation to predict y for new x values.

Linear regression uses **Mean Squared Error (MSE)** as the loss function:

```
mse= 1/n sum(y - y_pred)
```

**Semi-supervised learning** is a method sitting between supervised and unsupervised learning. The idea here is to run a clustering algorithm on an unlabeled dataset that will create few clusters as labels, clusters #1, #2, etc. Then propagate the labels to all the instances in the same cluster. And now, we have a labeled dataset that can be used for training a model in supervised learning.

## Reinforcement learning

**Reinforcement Learning** is a type of machine learning where an **agent learns by interacting with an environment** and receiving **feedback in the form of rewards or penalties**.

- Unlike supervised learning, RL does **not have labeled input-output pairs**.
- The agent learns **what actions lead to the best long-term outcome**.

### **Key Concepts in RL**

1. **Agent:** The learner or decision-maker.
    - Example: A robot, a game-playing AI.
2. **Environment:** Everything the agent interacts with.
    - Example: Game board, maze, or real-world environment.
3. **State (s):** A snapshot of the environment at a given time.
    - Example: Position of the robot in a maze.
4. **Action (a):** What the agent can do in a state.
    - Example: Move left, right, forward, pick up an object.
5. **Reward (r):** Feedback signal after taking an action.
    - Positive reward → good action
    - Negative reward → bad action
6. **Policy (π):** A strategy that the agent uses to decide actions based on states.
7. **Value Function:** Measures **how good a state or action is**, in terms of expected future rewards.

