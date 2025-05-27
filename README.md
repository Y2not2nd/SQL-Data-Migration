# End-to-End Azure Data Engineering Project: On-Premises to Power BI

🎉 **Special thanks to Luke J Byrne** for his excellent insights!  
Check out his tutorial here:  
[End-to-End Azure Data Engineering Project](https://www.google.com/search?q=https://www.youtube.com/watch%3Fv%3D2e5XQ8K7G5A)

---

## 📖 What’s This Repo About?

This repository contains everything you need to build an **end-to-end data engineering pipeline on Microsoft Azure**, moving data from an on-premises SQL Server all the way to a live Power BI dashboard.

You will:

✅ Extract data from on-prem SQL databases  
✅ Load raw data into Azure Data Lake Gen2 (Bronze layer)  
✅ Transform and clean data in Azure Databricks (Silver/Gold layers)  
✅ Query refined data in Azure Synapse Analytics  
✅ Visualize insights with Power BI  
✅ Automate and secure the whole pipeline using Azure-native tools

---

### 📂 Where to Find Full Instructions

👉 **Full, detailed steps are provided in** [`document.md`](document.md)  
That file contains every command, configuration, and code block you’ll need — from environment setup to automation and visualization.

This README gives you a **high-level overview**.

---

## 🔧 What You’ll Set Up

- **SQL Server Express + AdventureWorksLT Sample Database** (on your local machine)  
- **Azure Data Lake Gen2** with structured containers (Bronze/Silver/Gold)  
- **Azure Data Factory** for orchestrating pipelines  
- **Azure Databricks** for scalable data transformation  
- **Azure Synapse Analytics** for serverless querying  
- **Power BI** for dashboard visualization  
- **Azure Key Vault + Microsoft Entra ID (RBAC)** for secure, governed access

---

## 🚀 Project Stages

### 1️⃣ On-Prem Setup  
Set up SQL Server, restore sample data, and create the necessary SQL user.

### 2️⃣ Azure Setup  
Provision storage, compute, and analytics services in Azure.

### 3️⃣ Data Ingestion (ADF)  
Configure pipelines to pull data from on-prem and land it in the cloud.

### 4️⃣ Data Transformation (Databricks)  
Run Spark notebooks to clean and enrich data for analytics.

### 5️⃣ Query Layer (Synapse)  
Expose transformed data to SQL queries and reporting tools.

### 6️⃣ Visualization (Power BI)  
Connect Power BI to Synapse, design meaningful dashboards, and publish.

### 7️⃣ Automation + Security  
Schedule pipeline runs and apply proper access control and key management.

---

## ✅ Prerequisites

To follow this project, you’ll need:

- ✅ **Azure Subscription** (a free trial works fine)  
- ✅ **Windows Machine** (for running SQL Server Express + Power BI Desktop)  
- ✅ **Basic Knowledge** of Azure services and data workflows  
- ✅ **Patience and persistence** — this is a full-stack cloud pipeline with many moving parts!

> **Pro Tip:** You don’t need to be an expert!  
> `document.md` provides **step-by-step** instructions for beginners.

---

## 📂 Recommended Folder Structure


/project-root
│
├── data/
│ └── AdventureWorksLT2022.bak
├── notebooks/
│ ├── BronzeToSilver.ipynb
│ └── SilverToGold.ipynb
├── pipelines/
│ └── CopyAllTablesPipeline.json
├── synapse/
│ ├── CreateSQLServerlessView_gold.sql
│ └── SynapsePipeline.json
├── powerbi/
│ └── AdventureWorksLT.pbix
├── document.md <-- 🛠 FULL STEP-BY-STEP GUIDE
└── README.md




---

## 💡 Useful Links

- [Azure Data Factory Docs](https://learn.microsoft.com/en-us/azure/data-factory/)  
- [Azure Databricks Docs](https://learn.microsoft.com/en-us/azure/databricks/)  
- [Azure Synapse Analytics Docs](https://learn.microsoft.com/en-us/azure/synapse-analytics/)  
- [Power BI Docs](https://learn.microsoft.com/en-us/power-bi/)  
- [AdventureWorks Sample Databases](https://learn.microsoft.com/en-us/sql/samples/adventureworks-install-configure)

---

## 🛡️ License & Acknowledgements

This project is for **educational purposes only**.  
If you share or adapt this repository, please **credit Luke J Byrne** for the original concept and tutorial inspiration.

---

✉ **Questions? Ideas? Found something to improve?**  
Feel free to open an issue or submit a pull request!  
Let’s make this project even better together 🚀

---

