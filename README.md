# Fintech AI Projects Portfolio

This portfolio contains two applied AI projects in the financial domain: a fraud detection system and an auto-financing chatbot.

## 1) Fraud Detection with Machine Learning

This project focuses on detecting fraudulent bank transactions using a highly imbalanced dataset.

Key models used:
- Logistic Regression
- Random Forest
- XGBoost
- Balanced Random Forest
- Soft Voting Ensemble (LR + RF + XGB)

Key technologies used:
- Python
- Jupyter Notebook
- pandas, NumPy
- scikit-learn
- XGBoost
- Flask REST API

Important highlights:
- Performed data cleaning, feature engineering, outlier analysis, and scaling
- Addressed class imbalance using business-aware modeling and evaluation
- Compared multiple models using Precision, Recall, F1-score, and ROC-AUC
- Selected an ensemble model to balance fraud detection performance and false positives
- Deployed the final model as a REST API for real-world use


## 2) Auto Loan Finance Chatbot

This project is a conversational banking assistant that helps customers complete auto-financing applications and answer financing-related questions.

Key models and technologies used:
- Llama 3
- Open-source LLM deployment
- Python
- Local GPU infrastructure
- Conversational UI / chatbot workflow
- Business-rule-driven application logic

Important highlights:
- Guides customers through new vs. used vehicle financing flows
- Collects required information based on product type and validation rules
- Uses business documentation to answer common financing questions
- Includes cross-sell logic for HGS product offers
- Built to support a realistic fintech customer journey
