# Fraud-Detection-using-Neo4j-Graph-Data-Science

Dataset: https://drive.google.com/drive/folders/1LaNFObKnZb1Ty8T7kPLCYlXDUlHU7FGa

<br/><br/>
## Part 1: Exploring Connected Fraud Data
### **Dataset Overview**

This dataset is an anonymized sample of user accounts and transactions from a real-world Peer-to-Peer (P2P) platform. Before being imported into the graph database, original identification numbers were removed, and categorical values were masked to ensure privacy. Each user account is assigned a unique 128-bit identifier, while other nodes—representing unique credit cards, devices, and IP addresses—are assigned randomly generated UUIDs. These identifiers are stored under the `guid` property in the graph schema.



### **Exploring Fraud Patterns with Community Detection**

Given the uncertainty around fully capturing fraud through simple chargeback logic and limited connectivity, there's a possibility that not all fraudulent activity has been labeled. Fraud patterns are often complex, and fraudsters may use multiple user accounts, devices, and identifiers, making it difficult to detect their activities through single connections alone. While it’s tempting to label any account that shares a device or credit card with a flagged user as fraudulent, this approach might misclassify innocent users whose devices or cards were temporarily used by fraudsters.

In graph analysis, we can attempt to identify these fragmented, suspicious identities using **Community Detection**. Community Detection encompasses various techniques aimed at partitioning a graph into well-connected clusters, or "communities," where connections within communities are stronger than those outside. This approach helps reveal patterns of shared attributes that could indicate potential fraud networks.

### **Louvain Community Detection**

To explore fraud patterns, we can use **Louvain Community Detection**, a popular method in Neo4j’s Graph Data Science (GDS) library. Louvain uses modularity scoring to split a graph into hierarchical clusters, making it valuable for exploratory analysis. It does not require a precise theory of the graph structure, allowing us to investigate communities that might hold useful insights into potential fraud patterns.

For this analysis, we’ll use **P2P transactions, credit cards, and devices** to run Louvain, excluding IP addresses for now due to their “super node” issues (highly connected nodes that could skew results). By ordering communities based on the number of flagged users, we can prioritize examination of the communities with higher fraud concentrations. This approach helps us identify clusters that may contain previously undetected fraudulent activity.

Below are example queries for running Louvain in Neo4j GDS and aggregating community statistics, followed by ordering communities by the count of flagged users to identify high-risk clusters for further investigation.


<br/><br/>
## Part 2: Resolving Fraud Communities using Entity Resolution and Community Detection
Identifying community structures that may represent underlying groups of individuals is a crucial step in fraud detection. After an initial exploration with Louvain, we will now formalize our approach using Entity Resolution (ER). ER helps define connections between users likely belonging to the same hidden group or community, allowing us to create more structured partitions.

To accomplish this, we’ll establish ER business rules that create relationships between users, reflecting likely connections. With these relationships in place, we’ll use the Weakly Connected Components (WCC) algorithm to detect communities based on these new relationships. Finally, any user within a community containing flagged accounts will be labeled as a potential fraud risk.

### **Entity Resolution Business Rules**
To identify likely groups of individuals represented by multiple accounts, we’ll define ER rules. The rules will be used to create new relationships between users who share characteristics typical of fraudulent clusters. Here’s the logic we’ll apply: Shared Transactions and Credit Cards: If a user has sent money to another user who shares the same credit card, we’ll link these accounts.


Shared Identifiers with Thresholds: If two users share:
1. A credit card or device linked to 10 or fewer total accounts
2. At least two other identifiers of type credit card, device, or IP address
...then we’ll connect them, as this likely reflects an underlying association.


These rules are only examples and can be adjusted to fit real-world conditions. 

### **Weakly Connected Components (WCC) for Community Detection**

The Weakly Connected Components (WCC) algorithm is a scalable and explainable method for community detection. WCC identifies communities as sets of nodes connected by specific relationships, making it an excellent choice for fraud detection, where clear and deterministic groupings are valuable.

In our case, WCC will help resolve communities based on the Entity Resolution (ER) relationships we created. This approach allows us to assign formal community labels to users, grouping them into connected clusters that may reveal hidden fraud patterns. Users within the same WCC cluster can be considered part of the same community, which is helpful in identifying and examining groups likely to involve fraud.


<br/><br/>
## Part 3: Recommending Suspicious Accounts With Centrality & Node Similarity
In parts 1 and 2, we explored the graph to identify high-risk fraud communities. Now, we can go a step further by automatically identifying users that are suspiciously similar to the fraud risks already flagged. By expanding our analysis to include centrality and similarity algorithms, we can triage and recommend additional suspect users quickly and efficiently. Neo4j and Graph Data Science (GDS) provide an easy way to achieve this in seconds.

### **Using Weighted Degree Centrality to Recommend Potential High-Risk Accounts**
One approach to identifying suspicious users is to use Weighted Degree Centrality. This algorithm ranks users based on their connectedness in the graph, considering the importance of their relationships with key identifiers (e.g., Devices, Cards, and IP addresses). We can enhance the centrality calculation by weighting the degree centrality with the fraudRiskRatios we created in part 2. This will help prioritize users that are more likely to be connected to known fraud risks.


<br/><br/>
## **Part 4: Predicting Fraud Risk Accounts with Machine Learning**

In real-world fraud detection scenarios, it's often difficult to identify fraudulent user accounts ahead of time. In some cases, like with this dataset, accounts are flagged for fraud based on **business rules**—such as chargeback history—or through **user reporting mechanisms.** However, as we observed in Parts 1 and 2, these flags alone may not provide a complete picture. While **community detection** and **recommendation techniques** have helped uncover hidden fraud patterns, there are still instances where additional fraudulent accounts might go undetected by business rules alone.

Supervised **Machine Learning (ML)** can be highly beneficial in these situations. It allows us to predict fraud risk by using labeled data to train models that can recognize patterns of fraud and identify new fraud risks proactively. Here’s why integrating supervised ML is valuable:

### 1. **Proactive Detection**
   - By training a machine learning model, we can **anticipate fraud** before traditional flags such as chargebacks are triggered or new reports come in. This allows us to identify fraudulent actors even if they are not yet linked to known fraudulent accounts.
   - The model can detect **emerging fraud trends** or new communities of fraudsters that aren’t connected to older fraud patterns, providing more comprehensive coverage for fraud prevention.

### 2. **Measurable Performance**
   - Unlike heuristic or rule-based approaches, **supervised learning** models produce **clear performance metrics**—such as accuracy, precision, recall, and F1-score—that allow us to evaluate how well the model is identifying fraud risk.
   - These metrics give us a **benchmark** to assess the model’s effectiveness and adjust it as needed, whether through **hyperparameter tuning**, using more data, or experimenting with different features.

### 3. **Automation**
   - **Supervised Machine Learning** automates the **process of predicting fraud risk** accounts. Once a model is trained, it can score new user accounts in real-time, reducing the reliance on manual intervention or rule-based decision-making. This enhances **operational efficiency** and allows the fraud detection system to scale.
   - The automation also ensures that the fraud detection system **remains consistent** and free of human bias, helping to maintain fair decision-making processes.

### The Role of Supervised ML in Fraud Detection

Using labeled fraud data (fraudulent or not), we can train supervised machine learning models to recognize patterns associated with fraud. This involves:

1. **Feature Engineering:** Creating features such as transaction behavior, user activity, network centrality, and other graph-based features that can help the model identify fraudulent patterns.
2. **Training:** Feeding the labeled data into machine learning models like decision trees, logistic regression, or more advanced methods like Random Forests, Gradient Boosting, or neural networks.
3. **Evaluation:** Evaluating the model using metrics like **confusion matrix**, **ROC-AUC**, and **precision-recall curve** to understand how well the model is identifying fraud and non-fraud accounts.
4. **Prediction:** Once trained, the model can predict whether new, unseen accounts are likely to be fraudulent, allowing fraud teams to take preventative action.

By combining machine learning with the techniques we’ve used in previous sections—such as **community detection** and **entity resolution**—we can create a more robust fraud detection system that **automatically** adapts to new fraud patterns and helps prioritize fraud investigation efforts.

