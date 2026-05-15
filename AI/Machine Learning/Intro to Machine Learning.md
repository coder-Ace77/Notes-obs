
---

**Machine Learning** is a field of **artificial intelligence (AI)** where computers learn patterns from **data** and make predictions or decisions **without being explicitly programmed** for each task.

**Classical ML** refers to the traditional ML algorithms that are **not deep learning**. These methods typically work on **structured/tabular data** and require **feature engineering** (manual creation of useful features).

**Deep Learning (DL)** is a subfield of **machine learning** that uses **artificial neural networks** with many layers (hence “deep”) to automatically learn patterns and representations from data.
## Classes

#### Supervised Learning

- **Definition:** The algorithm learns from **labeled data**, i.e., each input comes with a known output (target).
- **Goal:** Predict the output for new, unseen data.
- **Types:**
    - **Regression:** Predict continuous values (e.g., house prices, stock prices).
    - **Classification:** Predict discrete categories (e.g., spam/not spam, disease diagnosis).
- **Examples:** Linear Regression, Logistic Regression, Decision Trees, Random Forest, SVM, Neural Networks.

#### Unsupervised Learning

- **Definition:** The algorithm works on **unlabeled data** and tries to find patterns, structures, or relationships.
- **Goal:** Discover hidden structures in data.
- **Types:**
    - **Clustering:** Group similar data points together (e.g., K-Means, DBSCAN).
    - **Dimensionality Reduction:** Reduce features while preserving information (e.g., PCA, t-SNE).

- **Examples:** Customer segmentation, market basket analysis, anomaly detection.

### Reinforcement Learning

- **Definition:** The algorithm learns by **trial and error** through interaction with an environment.
- **Goal:** Maximize cumulative reward over time.
- **Key Concepts:** Agent, Environment, Action, Reward.
- 
- **Examples:**
    
    - Game playing (chess, Go, video games).
    - Robotics (teaching robots to walk or grasp objects).
    - Recommendation systems with feedback loops.


### Semi supervised 

- **Definition:** Uses a mix of **labeled and unlabeled data**.
- **Goal:** Improve learning when labeling data is expensive or time-consuming.
- **Example:** Medical image classification where only some images are labeled.


### Self-Supervised Learning

- **Definition:** A special type of unsupervised learning where the system **generates pseudo-labels** from the input data itself.
- **Example:** Large language models like GPT use self-supervised learning to predict missing words.


### **Regression**

- **Definition:** Regression is a type of supervised learning used to predict a **continuous numerical value** based on input data.
- **Goal:** Find the relationship between input variables (features) and a numeric output (target).
- **Example:**
    - Predicting house prices based on size, location, and number of bedrooms.
    - Forecasting temperature tomorrow based on historical data.
- **Algorithms:** Linear Regression, Polynomial Regression, Ridge/Lasso Regression, Support Vector Regression (SVR).

### **Classification**

- **Definition:** Classification is another type of **supervised learning**, but here the goal is to predict a **discrete label or category**.
- **Goal:** Assign input data to one of the predefined classes.
- **Example:**
    - Spam detection in emails → “Spam” or “Not Spam”.
    - Predicting if a tumor is “Benign” or “Malignant”.
    - Handwriting recognition → identifying digits 0–9.
- **Algorithms:** Logistic Regression, Decision Trees, Random Forest, Support Vector Machines (SVM), Neural Networks.

### **Clustering**

- **Definition:** Clustering is **unsupervised learning**, where the goal is to group data into clusters based on similarity **without predefined labels**.
- **Goal:** Find inherent structures or patterns in the data.
- **Example:**
    
    - Customer segmentation → grouping customers by buying behavior.
    - Grouping news articles by topic.
    - Image compression → grouping similar pixels.
        
- **Algorithms:** K-Means, Hierarchical Clustering, DBSCAN, Gaussian Mixture Models.

