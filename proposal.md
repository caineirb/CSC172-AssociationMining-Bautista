# CSC172 Association Rule Mining Project Proposal
**Student:** Caine Ivan R. Bautista, 2022-0378
**Date:** December 12, 2025

## 1. Project Title 
Educational Data Mining: Student Learning Behavior Analysis using Apriori Algorithm for the EDM Cup 2023 Dataset

## 2. Problem Statement
Educational institutions often find it difficult to determine which learning behaviors and help-seeking patterns contribute to student success or failure on assessments. Understanding how these behaviors influence long-term performance has broad practical value. This project analyzes clickstream data from the ASSISTments online learning platform to uncover association rules linking student behaviors—such as hint usage, response times, and error patterns—to their unit test performance.

## 3. Objectives
- Analyze three key questions using association rule mining:
  1. Which problem-solving behaviors co-occur during student work?
  2. What combinations of actions predict success/failure on unit tests?
  3. Which help-seeking patterns are associated with mastery vs struggle?
- Preprocess educational clickstream data and perform EDA with visualizations.
- Implement Apriori algorithm to generate actionable association rules.
- Evaluate rules using support, confidence, lift, and interpret educational significance.
- Visualize patterns and derive educational insights

## 4. Dataset Plan
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

## 5. Technical Approach
- **Preprocessing:** 
  - Transform clickstream into three transaction types:
    1. Student-problem interactions (behavior co-occurrence)
    2. Student-unit aggregations (success/failure prediction)
    3. Student help-seeking patterns (mastery analysis)
  - Feature engineering: response time categories, hint rates, accuracy metrics, tutoring usage patterns
  - Handle temporal sequences and multi-file relationships
- **Algorithm:** Apriori with mlxtend library
  - Analysis 1 (Behaviors): min_support=0.05, min_confidence=0.6, min_lift=1.5
  - Analysis 2 (Prediction): min_support=0.05, min_confidence=0.6, min_lift=1.5
  - Analysis 3 (Help-seeking): min_support=0.05, min_confidence=0.6, min_lift=1.5
  - Parameters tunable based on dataset characteristics
- **Framework:** Python + pandas + mlxtend + matplotlib/seaborn
- **Environment:** Jupyter Notebook

## 6. Expected Challenges & Mitigations
- **Challenge:** Extremely large-scale clickstream data (~23.9M action logs, ~9.3M assignments)
  - **Solution:** 
    - Process data in chunks using pandas `chunksize` parameter
    - Filter to relevant assignment_log_ids early (only in-unit assignments via relationships)
    - Use memory-efficient dtypes (category, int32 instead of int64)
    - Sample subset for prototyping (~10% of data), then scale to full dataset
  
- **Challenge:** High memory consumption (~5GB RAM required)
  - **Solution:** 
    - Load only necessary columns with `usecols` parameter
    - Process analyses sequentially and clear memory between runs (`del`, `gc.collect()`)
    - Drop intermediate DataFrames after merging
  
- **Challenge:** Complex multi-file relationships across 10 CSV files
  - **Solution:** 
    - Pre-compute and cache unit test mappings (702K assignment relationships)
    - Index key columns for faster lookups (assignment_log_id, problem_id)
    - Use vectorized pandas operations instead of iterrows()
    - Merge only necessary fields
  
- **Challenge:** Sparse transaction matrix with diverse items (132K problems, 8K hints)
  - **Solution:** 
    - Aggregate similar items into categories (problem types, difficulty levels)
    - Start with higher min_support (0.05-0.1) given large transaction volume
  
- **Challenge:** Trivial or obvious rules in large-scale data
  - **Solution:** 
    - Set minimum lift threshold (≥1.5) for interesting associations
    - Filter out consequents that are too common (e.g., remove rules → "medium_performance")
    - Focus on actionable rules linking behaviors to unit test outcomes
    - Separate analyses by key question to avoid mixing contexts
    - Sort by lift and manually review top 20-30 rules for educational significance