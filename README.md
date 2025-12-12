# Educational Data Mining: Student Learning Behavior Analysis using Apriori Algorithm for the EDM Cup 2023 Dataset
**CSC172 Data Mining and Analysis Final Project**  
*Mindanao State University - Iligan Institute of Technology*  
**Student:** Caine Ivan R. Bautista, 2022-0378
**Semester:** A.Y. 2025-2026 1st Semester

## Abstract
This project implements the Apriori algorithm for association rule mining on [dataset name] containing [X] transactions. Key findings include [top rule example: "if {bread} then {butter}" with lift=2.3]. The analysis pipeline includes data preprocessing, exploratory data analysis (EDA), rule generation, and evaluation using support, confidence, lift, and conviction metrics. Business insights and actionable recommendations are derived from the strongest rules.

## Table of Contents
- [Abstract](#abstract)
- [1. Introduction](#1-introduction)
  - [1.1 Problem Statement](#11-problem-statement)
  - [1.2 Objectives](#12-objectives)
  - [1.3 Scope and Limitations](#13-scope-and-limitations)
- [2. Dataset Description](#2-dataset-description)
  - [2.1 Source and Acquisition](#21-source-and-acquisition)
  - [2.2 Data Structure](#22-data-structure)
  - [2.3 Sample Transactions](#23-sample-transactions)
- [3. Methodology](#3-methodology)
  - [3.1 Data Preprocessing](#31-data-preprocessing)
  - [3.2 Exploratory Data Analysis](#32-exploratory-data-analysis)
  - [3.3 Apriori Algorithm Implementation](#33-apriori-algorithm-implementation)
  - [3.4 Evaluation Metrics](#34-evaluation-metrics)
- [4. Results](#4-results)
  - [4.1 Top Association Rules](#41-top-association-rules)
  - [4.2 Key Visualizations](#42-key-visualizations)
  - [4.3 Performance Metrics](#43-performance-metrics)
- [5. Discussion](#5-discussion)
  - [5.1 Business Insights](#51-business-insights)
  - [5.2 Actionable Recommendations](#52-actionable-recommendations)
  - [5.3 Limitations](#53-limitations)
- [6. Conclusion](#6-conclusion)
- [7. Video Presentation](#7-video-presentation)
- [References](#references)
- [Appendix: Full Results](#appendix-full-results)



## 1. Introduction
### 1.1 Problem Statement
Educational institutions often find it difficult to determine which learning behaviors and help-seeking patterns contribute to student success or failure on assessments. Understanding how these behaviors influence long-term performance has broad practical value. This project analyzes clickstream data from the ASSISTments online learning platform to uncover association rules linking student behaviors—such as hint usage, response times, and error patterns—to their unit test performance.

### 1.2 Objectives
- Analyze three key questions using association rule mining:
  1. Which problem-solving behaviors co-occur during student work?
  2. What combinations of actions predict success/failure on unit tests?
  3. Which help-seeking patterns are associated with mastery vs struggle?
- Preprocess educational clickstream data and perform EDA with visualizations.
- Implement Apriori algorithm to generate actionable association rules.
- Evaluate rules using support, confidence, lift, and interpret educational significance.
- Visualize patterns and derive educational insights

### 1.3 Scope and Limitations
**Scope:** Single snapshot analysis of grocery purchase patterns using Apriori algorithm  
**Limitations:** No temporal analysis, static customer behavior, computational constraints on full itemset space

## 2. Dataset Description
### 2.1 Source and Acquisition
- Source: [EDM Cup 2023 - Kaggle Competition](https://www.kaggle.com/competitions/edm-cup-2023/overview)
  ```
  Dimensions:
  action_logs.csv = (23932276, 10)
  assignment_details.csv = (9319676, 9)
  assignment_relationships.csv = (702887, 2)
  evaluation_unit_test_scores.csv = (124455, 4)
  sequence_relationships.csv = (13108, 2)
  training_unit_test_scores.csv = (452439, 3)
  problem_details.csv = (132738, 10)
  sequence_details.csv = (10774, 8)
  hint_details.csv = (8381, 7)
  explanation_details.csv = explanation_details.shape=(4132, 6)
  ```
- Domain: Educational Transaction
- Acquisition: Direct Kaggle download to `dataset/*.csv`
**Format:**
```
=== action_logs Columns ===
['assignment_log_id', 'timestamp', 'problem_id', 'max_attempts', 'available_core_tutoring', 'score_viewable', 'continuous_score_viewable', 'action', 'hint_id', 'explanation_id']

=== assignment_details Columns ===
['assignment_log_id', 'teacher_id', 'class_id', 'student_id', 'sequence_id', 'assignment_release_date', 'assignment_due_date', 'assignment_start_time', 'assignment_end_time']

=== assignment_relationships Columns ===
['unit_test_assignment_log_id', 'in_unit_assignment_log_id']

=== evaluation_scores Columns ===
['id', 'assignment_log_id', 'problem_id', 'score']

=== sequence_relationships Columns ===
['unit_test_sequence_id', 'in_unit_sequence_id']

=== training_scores Columns ===
['assignment_log_id', 'problem_id', 'score']

=== problem_details Columns ===
['problem_id', 'problem_multipart_id', 'problem_multipart_position', 'problem_type', 'problem_skill_code', 'problem_skill_description', 'problem_contains_image', 'problem_contains_equation', 'problem_contains_video', 'problem_text_bert_pca']

=== sequence_details Columns ===
['sequence_id', 'sequence_folder_path_level_1', 'sequence_folder_path_level_2', 'sequence_folder_path_level_3', 'sequence_folder_path_level_4', 'sequence_folder_path_level_5', 'sequence_name', 'sequence_problem_ids']

=== hint_details Columns ===
['hint_id', 'hint_creator_id', 'hint_position', 'hint_contains_image', 'hint_contains_equation', 'hint_contains_video', 'hint_text_bert_pca']

=== explanation_details Columns ===
['explanation_id', 'explanation_creator_id', 'explanation_contains_image', 'explanation_contains_equation', 'explanation_contains_video', 'explanation_text_bert_pca']
```

### 2.2 Data Structure
Raw format (one row per item):
member_id,date,product
1,2023-01-01,whole milk
1,2023-01-01,other vegetables
2,2023-01-01,yogurt

Transaction format (one row per basket):
[['whole milk', 'other vegetables'], ['yogurt', 'whole milk']]


### 2.3 Sample Transactions
Transaction 1: ['whole milk', 'other vegetables', 'root vegetables']
Transaction 2: ['yogurt', 'whole milk', 'rolls/buns']
Transaction 3: ['sausage', 'frankfurter', 'soda']


## 3. Methodology

### 3.1 Data Preprocessing
1. **Missing Value Handling:** Removed 127 incomplete transactions (1.3%)
2. **One-Hot Encoding:** Converted to 9,708 × 169 binary transaction matrix
3. **Item Filtering:** Retained top 50 items (support > 0.01) → 9,708 × 50 matrix
4. **Final Dataset:** 9,708 transactions × 50 items (98.7% sparsity reduced to manageable size)

**Before/After Statistics:**
| Metric | Raw Data | Processed Data |
|--------|----------|----------------|
| Transactions | 9,835 | 9,708 |
| Unique Items | 169 | 50 |
| Density | 0.12% | 2.1% |

### 3.2 Exploratory Data Analysis
- **Top 10 Items:** whole milk (25.3%), other vegetables (19.1%), rolls/buns (17.4%)
- **Basket Size:** Mean=2.4 items, 68% transactions contain 1-3 items
- **Co-occurrence:** whole milk appears with 89% of top 20 items

### 3.3 Apriori Algorithm Implementation
**Implementation:** mlxtend.frequent_patterns.apriori() with association_rules()

### 3.4 Evaluation Metrics
- **Support:** \( \frac{\text{support}(A \cup B)}{N} \) - Absolute frequency
- **Confidence:** \( \frac{\text{support}(A \cup B)}{\text{support}(A)} \) - Rule strength
- **Lift:** \( \frac{\text{confidence}(A \to B)}{\text{support}(B)} \) - Rule interestingness (>1 = positive association)


## 4. Results
### 4.1 Top Association Rules

| Rank | Antecedents | Consequents | Support | Confidence | Lift | Conviction | Leverage |
|------|-------------|-------------|---------|------------|------|------------|----------|
| 1 | {other vegetables} | {root vegetables} | 0.023 | 0.74 | 3.15 | 3.42 | 0.017 |
| 2 | {yogurt} | {whole milk} | 0.028 | 0.68 | 2.12 | 2.31 | 0.015 |
| 3 | {rolls/buns} | {whole milk} | 0.032 | 0.62 | 1.98 | 2.01 | 0.016 |
| 4 | {sausage} | {frankfurter} | 0.015 | 0.81 | 4.23 | 4.67 | 0.012 |
| 5 | {tropical fruit} | {other vegetables} | 0.021 | 0.65 | 2.34 | 2.41 | 0.014 |

### 4.2 Key Visualizations
![Item Frequency Distribution](results/item_frequencies.png) 

### 4.3 Performance Metrics
Runtime: Preprocessing=42s, Apriori=18s, Rules=3s (Total: 63s)
Scalability: Handles 10K+ transactions on standard laptop


## 5. Discussion

### 5.1 Business Insights
1. **Dairy Clustering:** whole milk as "hub item" (89% co-occurrence)
2. **Vegetable Pairing:** root vegetables strongly associated with other vegetables
3. **Breakfast Bundle:** yogurt + whole milk + rolls/buns (lift=2.1)

### 5.2 Actionable Recommendations
1. **Shelf Placement:** Place root vegetables near other vegetables
2. **Bundling:** Promote "Breakfast Pack" (yogurt + milk + rolls)
3. **Cross-promotion:** sausage → frankfurter discount coupons
4. **Inventory:** Stock 25% more whole milk based on pairing frequency

### 5.3 Limitations
- Single time period (no seasonality)
- No customer demographics
- Binary presence/absence (no quantities)

## 6. Conclusion
The Apriori algorithm successfully identified 25 actionable association rules from 9,708 grocery transactions. Strongest patterns reveal natural product groupings (lift > 3.0) suitable for retail optimization. Future work includes temporal analysis, customer segmentation, and FP-Growth comparison.


## 7. Video Presentation
[![Final Presentation](demo/CSC172_[LastName]_Final.mp4)](demo/CSC172_[LastName]_Final.mp4)  
*5-minute demo: Problem → Dataset → Methods → Key Findings → Business Insights*

## References
1. Agrawal, R., & Srikant, R. (1994). Fast Algorithms for Mining Association Rules. VLDB.
2. mlxtend Documentation: https://rasbt.github.io/mlxtend/
3. Groceries Dataset: https://www.kaggle.com/datasets/heeraldedhia/groceries-dataset

## Appendix: Full Results
**Complete rules CSV:** [results/rules_top25.csv](results/rules_top25.csv)  

