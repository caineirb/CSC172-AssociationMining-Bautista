# Educational Data Mining: Student Learning Behavior Analysis using Apriori Algorithm for the EDM Cup 2023 Dataset
**CSC172 Data Mining and Analysis Final Project**  
*Mindanao State University - Iligan Institute of Technology*  
**Student:** Caine Ivan R. Bautista, 2022-0378
**Semester:** A.Y. 2025-2026 1st Semester

## Abstract
This project applies association rule mining using the Apriori algorithm to analyze student learning behaviors in the EDM Cup 2023 dataset from ASSISTments, an online learning platform. The analysis examines three perspectives: (1) problem-solving behavior co-occurrence patterns, (2) student-unit performance prediction from aggregated behaviors, and (3) help-seeking mastery patterns. Key findings reveal that students with low help-seeking, low wrong attempts, and high completion rates show strong associations with high unit test scores (lift=3.26), while help-first strategies without prior attempts correlate with lower performance. The analysis processed 16.25M action logs and 5.14M problem attempts to generate actionable educational insights.

## Table of Contents
- [Abstract](#abstract)
- [1. Introduction](#1-introduction)
  - [1.1 Problem Statement](#11-problem-statement)
  - [1.2 Objectives](#12-objectives)
  - [1.3 Scope and Limitations](#13-scope-and-limitations)
- [2. Dataset Description](#2-dataset-description)
  - [2.1 Source and Acquisition](#21-source-and-acquisition)
  - [2.2 Data Structure](#22-data-structure)
    - [2.2.1 Raw Action Log Format](#221-raw-action-log-format)
    - [2.2.2 Transaction Formats (Post-Processing)](#222-transaction-formats-post-processing)
  - [2.3 Sample Transactions](#23-sample-transactions)
- [3. Methodology](#3-methodology)
  - [3.1 Data Preprocessing](#31-data-preprocessing)
  - [3.2 Exploratory Data Analysis](#32-exploratory-data-analysis)
  - [3.3 Apriori Algorithm Implementation](#33-apriori-algorithm-implementation)
  - [3.4 Evaluation Metrics](#34-evaluation-metrics)
- [4. Results](#4-results)
  - [4.1 Analysis 1: Student-Problem Interactions](#41-analysis-1-student-problem-interactions)
  - [4.2 Analysis 2: Student-Unit Aggregations](#42-analysis-2-student-unit-aggregations)
  - [4.3 Analysis 3: Help-Seeking Patterns](#43-analysis-3-help-seeking-patterns)
- [5. Discussion](#5-discussion)
  - [5.1 Educational Insights](#51-educational-insights)
  - [5.2 Actionable Recommendations](#52-actionable-recommendations)
  - [5.3 Limitations](#53-limitations)
- [6. Conclusion](#6-conclusion)
- [7. Video Presentation](#7-video-presentation)
- [References](#references)
- [Appendix: Full Results](#appendix-full-results)
- [Github Pages](#github-pages)


## 1. Introduction
### 1.1 Problem Statement
Educational institutions often struggle to understand which specific learning behaviors and help-seeking patterns contribute to student success or failure on assessments. While online learning platforms capture vast amounts of clickstream data, extracting actionable insights that can guide instructional interventions remains challenging. This project addresses three critical questions:

1. Behavioral Co-occurrence: What problem-solving behaviors (hints, wrong attempts, time spent) naturally occur together during student work sessions?
2. Performance Prediction: Which combinations of aggregated behaviors across multiple assignments predict success or failure on unit tests?
3. Mastery Patterns: What help-seeking patterns distinguish students who achieve mastery from those who continue to struggle?

Understanding these relationships through association rule mining can help educators identify at-risk students early, personalize interventions, and design more effective learning pathways.

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
**Scope:**
- Dataset: EDM Cup 2023 training data (2018-2022 academic years)
- Platform: ASSISTments online learning system for K-12 mathematics
- Analysis window: Static snapshot analysis across 638K assignment sessions
- Methods: Apriori algorithm with transaction encoding for three distinct perspectives

**Limitations:**
- Causality: Association rules reveal correlation, not causal relationships between behaviors and outcomes
- Computational: Memory limitations required chunked processing for large aggregations (16GB+ dataframes)
- Feature simplification: Binned continuous variables (e.g., time spent, hint counts) may lose granular patterns
- Cold start: Cannot analyze students with fewer than 3 completed problems
- Platform-specific: Findings may not generalize to other learning management systems with different affordances

## 2. Dataset Description
### 2.1 Source and Acquisition
- Source: [EDM Cup 2023 - Kaggle Competition](https://www.kaggle.com/competitions/edm-cup-2023/overview)
- Domain: Educational clickstream data from ASSISTments online math learning platform
- Time Period: 2018-2022 academic years (September - June cycles)
- Acquisition: Direct download from Kaggle to `dataset/*.csv` (2.4GB compressed)
- Ethics: De-identified student data with teacher/class ID for privacy

**Used Dataset Files**
| File | Rows | Columns | Size | Description |
| ---- | ---- | ------- | ---- | ----------- |
| action_logs.csv | 23,932,276 |10 |1.37 GB | Timestamped student actions |
| assignment_details.csv | 9,319,676 | 9 | 921 MB | Assignment metadata | 
| problem_details.csv | 132,738 | 10 | 59 MB | Problem characteristics | 
| training_unit_test_scores.csv | 452,439 | 3 | 10 MB | Unit test outcomes | 
| assignment_relationships.csv | 702,887 | 2 | 14 MB | Unit test ↔ in-unit links |
| sequence_details.csv | 10,774 |8 | 3.8 MB | Curriculum structure | 

### 2.2 Data Structure
#### 2.2.1 Raw Action Log Format
```
assignment_log_id,timestamp,problem_id,action,hint_id,explanation_id
2QV1F2GSBZ,1599150990,I2GX4OQIE,problem_started,,
2QV1F2GSBZ,1599150997,I2GX4OQIE,wrong_response,,
2QV1F2GSBZ,1599151002,I2GX4OQIE,hint_requested,OEM5SD5F2,
```
**Key Columns Used:**

| Column | Type | Description | Example Values |
|--------|------|-------------|----------------|
| `action` | categorical | Student interaction type | `problem_started`, `wrong_response`, `hint_requested`, `correct_response` |
| `timestamp` | float | Unix epoch time | `1599150990.0` |
| `problem_id` | string | Unique problem identifier | `I2GX4OQIE` |
| `problem_type` | categorical | Question format | `Multiple Choice`, `Numeric`, `Algebraic Expression` |
| `problem_skill_code` | string | Common Core standard | `6.RP.A.3b`, `8.EE.A.1-1` |
| `score` | binary | Unit test outcome | `0` (fail), `1` (pass) |

#### 2.2.2 Transaction Formats (Post-Processing)
##### Analysis 1: Problem-Solving Behaviors
```
# One transaction per (assignment_log_id, problem_id)
transaction_id: "2QV1F2GSBZ_I2GX4OQIE"
items: ['few_wrongs', 'answer_req', 'medium_time', 'correct_after_help', 'numeric']
```
##### Analysis 2: Unit Test Prediction
```
# One transaction per unit_test_assignment_log_id
transaction_id: "1KPCIEDF9V"  
items: ['high_score', 'low_help', 'low_wrongs', 'high_completion', 'med_problems']
```
##### Analysis 3: Help Seeking Patterns:
```
# One transaction per assignment_log_id (help-seekers only)
transaction_id: "YKLKRCDDM"
items: ['rare_hints', 'low_help', 'tries_before_help', 'high_success', 'elementary']
```

### 2.3 Sample Transactions
**Analysis 1 Example (Problem Interactions):**
```python
Transaction 794213: 
  ['no_hints', 'few_wrongs', 'slow', 'answer_req', 'no_explanation', 'correct_after_help', 'algebra']

Transaction 129199: 
  ['no_hints', 'no_wrongs', 'medium', 'no_answer', 'no_explanation', 'correct_first_try', 'other']
```

**Analysis 2 Example (Unit Test Aggregations):**
```python
Transaction 8914 (High Score): 
  ['high_score', 'typical_help', 'low_wrongs', 'high_completion', 'med_problems', 'low_struggle', 'low_effort']

Transaction 36923 (Low Score): 
  ['low_score', 'typical_help', 'low_wrongs', 'low_completion', 'few_problems', 'low_struggle', 'low_effort']
```

**Analysis 3 Example (Help-Seeking):**
```python
Transaction 202669 (Successful Help-Seeker): 
  ['rare_hints', 'low_help', 'no_explanation', 'some_answer_req', 'high_success', 'help_first', 'no_self_correction', 'elementary']

Transaction 17433 (Struggling Help-Seeker): 
  ['rare_hints', 'low_help', 'no_explanation', 'some_answer_req', 'low_success', 'tries_before_help', 'no_self_correction', 'elementary']
```

## 3. Methodology

### 3.1 Data Preprocessing

#### Step 1: Action Log Filtering
- **Input:** 23,932,276 raw action logs
- **Process:**
  1. Dropped non-essential columns: max_attempts, score_viewable, continuous_score_viewable, hint_id, explanation_id
  2. Filtered to relevant actions: problem_started, wrong_response, correct_response, hint_requested, explanation_requested, answer_requested, problem_finished
  3. Removed assignment start/resume/continue actions
- **Output:** 16,252,841 action logs (67.9% retention)

#### Step 2: Problem Details Enrichment
- **Input:** 132,738 problem records
- **Process:**
  1. Filled missing boolean flags (`problem_contains_image/equation/video`) with 0
  2. Replaced missing `problem_skill_code` with "Unknown"
  3. Dropped unused columns: `problem_text_bert_pca`, `problem_multipart_id/position`, `problem_skill_description`
- **Output:** 132,738 × 6 enriched problem metadata

#### Step 3: Feature Engineering (Analysis-Specific)
1. Problem Attempt Aggregation:
```python
# Per (assignment_log_id, problem_id) group
- hint_count: sum(action == 'hint_requested')
- wrong_count: sum(action == 'wrong_response')
- answer_requested: any(action == 'answer_requested')
- time_spent: finished_timestamp - started_timestamp
- final_outcome: 'correct_first_try' | 'correct_after_help' | 'gave_up'

# Binning:
- hints: [-0.1, 0, 2, ∞] → ['no_hints', 'few_hints', 'many_hints']
- wrongs: [-0.1, 0, 2, ∞] → ['no_wrongs', 'few_wrongs', 'many_wrongs']
- time: [q33, q66] quantiles → ['fast', 'medium', 'slow']
```
- Chunking Strategy: Processed 1M rows at a time with buffer carryover to handle incomplete groups
- Output: 5,140,889 problem attempts

2. Unit Test Behavior Aggregation:
```python
# Per in-unit assignment
- completion_rate: problems_finished / problems_started
- help_per_problem: (hints + explanations + answers) / problems_started
- struggle_ratio: wrong_count / (correct_count + 1)

# Per unit test (average across linked in-unit assignments)
- avg_completion_rate, avg_help_seeking, avg_wrong_attempts, avg_struggle_ratio

# Binning:
- score: [0, 0.5, 1] → ['low_score', 'high_score']
- help: quantiles [0, .7, .9, 1] → ['typical_help', 'high_help', 'very_high_help']
```
- Balancing: Downsampled majority class (score=1) to 187,520 samples for 1:1 ratio
- Output: 42,244 unit test transactions (balanced)

3. Help-Seeking Pattern Extraction:
```python
# Filter: assignments with at least one help action (hints/explanations/answers)
- hints_per_problem: hint_count / problems_started
- wrong_before_help: first_wrong_index < first_help_index
- self_correction: any(wrong_response → correct_response sequence)

# Binning:
- hint_pattern: [0, 0.5, 2, ∞] → ['rare_hints', 'moderate_hints', 'frequent_hints']
- success_rate: [0, 0.5, 0.8, 1] → ['low_success', 'med_success', 'high_success']
```
- **Grade Categorization:** Extracted from `sequence_folder_path_level_2`:
  - Grades 1-5 → 'elementary'
  - Grades 6-8 → 'middle'
  - Algebra/Geometry/Calculus → 'high_school'
- **Output:** 204,330 help-seeking transactions (min 3 problems)

#### Step 4: Transaction Encoding
- **Method:** `mlxtend.TransactionEncoder()` for one-hot encoding
- **Process:** Convert item lists to boolean matrices
- **Output Dimensions:**
  - Analysis 1: 5,140,889 × 24 (density=29.17%)
  - Analysis 2: 42,244 × 19 (density=36.84%)
  - Analysis 3: 204,330 × 27 (density=33.33%)

**Before/After Statistics:**

| Metric | Analysis 1 | Analysis 2 | Analysis 3 |
|--------|-----------|-----------|-----------|
| Raw Actions | 16.25M | 16.25M | 16.25M |
| Transactions | 5.14M | 42.2K | 204K |
| Unique Items | 24 | 19 | 27 |
| Sparsity | 70.83% | 63.16% | 66.67% |
| Memory Usage | ~980 MB | ~6.1 MB | ~42 MB |


### 3.2 Exploratory Data Analysis

**Analysis 1 - Problem Interaction Patterns:**

**Top 5 Most Frequent Items:**
| Item | Support | Interpretation |
|------|---------|----------------|
| `no_explanation` | 99.6% | Students rarely request explanations |
| `no_hints` | 99.0% | Most problems solved without hints |
| `no_answer` | 88.4% | Answer key rarely viewed |
| `no_wrongs` | 82.8% | Majority get correct on first try |
| `fast` | 55.9% | Over half complete quickly (<33rd percentile) |

**Key Insights:**
- **Self-directed learning:** 88% complete problems without any help (no_hints ∧ no_answer ∧ no_explanation)
- **Outcome distribution:** 
  - 67.3% correct_first_try
  - 20.1% correct_after_help  
  - 12.6% gave_up
- **Problem type distribution:** 
  - 42% numeric, 28% multiple choice, 18% open response

**Analysis 2 - Unit Test Prediction Patterns:**

**Top 5 Most Frequent Items:**
| Item | Support | Interpretation |
|------|---------|----------------|
| `low_wrongs` | 95.9% | Very few wrong attempts overall |
| `high_completion` | 86.9% | Most complete assigned work |
| `low_struggle` | 70.0% | Wrong/correct ratio typically low |
| `low_effort` | 70.0% | Minimal total interactions per problem |
| `typical_help` | 70.0% | Standard help-seeking frequency |

**Key Insights:**
- **Score distribution:** 50% high_score, 50% low_score (balanced via downsampling)
- **Completion matters:** High completion (86.9%) strongly clusters with success
- **Basket size:** Mean 7.0 items per transaction (standardized encoding)

**Analysis 3 - Help-Seeking Mastery Patterns:**

**Top 5 Most Frequent Items:**
| Item | Support | Interpretation |
|------|---------|----------------|
| `low_help` | 98.6% | Help-seekers use minimal resources |
| `rare_hints` | 98.2% | <0.5 hints per problem |
| `no_explanation` | 95.1% | Explanations almost never used |
| `tries_before_help` | 77.1% | Prefer attempting first |
| `unknown` | 65.2% | Many assignments lack grade metadata |

**Key Insights:**
- **Grade distribution:** 72% elementary, 18% middle, 10% high school
- **Success patterns:** 41% high_success, 36% med_success, 23% low_success
- **Self-correction:** 48% show at least one wrong→correct sequence


### 3.3 Apriori Algorithm Implementation
**Implementation:** `mlxtend.frequent_patterns.apriori()` with `association_rules()`

**Algorithm Configuration:**
```python
from mlxtend.frequent_patterns import apriori, association_rules

# Hyperparameters
AnalysisConfig:
    min_support = 0.05      # 5% frequency threshold
    min_confidence = 0.60   # 60% rule strength threshold  
    min_lift = 1.0          # Positive association only
    top_n_rules = 10        # Report top 10 by lift
```

**Execusion Pipeline:**
```python
# Step 1: Generate frequent itemsets
frequent_itemsets = apriori(
    encoded_df,
    min_support=AnalysisConfig.min_support,
    use_colnames=True
)

# Step 2: Generate rules
rules = association_rules(
    frequent_itemsets,
    metric='confidence',
    min_threshold=AnalysisConfig.min_confidence
)

# Step 3: Filter by lift and sort
rules = rules[rules['lift'] >= AnalysisConfig.min_lift]
rules = rules.sort_values(['lift', 'confidence'], ascending=False)
```

**Computational Performance:**

| Analysis | Transactions | Items | Itemsets Found | Rules Found | Runtime | 
|-|-|-|-|-|-|
| 1 | 5,140,889 | 24 | 307 | 342 | 11 | 
| 2 | 42,244 | 19 | 608 | 418 | ~8 sec | 
| 3 | 204,330 | 27 | 2,399 | 1,847 | ~45 sec |

**Hardware:** AMD Ryzen 7 5800H (16) @ 4.46 GHz, 16GB RAM, no GPU acceleration

### 3.4 Evaluation Metrics
**Support:** Absolute frequency of itemset occurrence
$$ \text{support}(A) = \frac{|\{t \in T : A \subseteq t\}|}{|T|}​ $$
**Confidence:** Conditional probability of consequent given antecedent
$$ \text{confidence}(A \to B) = \frac{\text{support}(A \cup B)}{\text{support}(A)} $$
**Lift:** Ratio of observed to expected co-occurrence (independence baseline)
$$ \text{lift}(A \to B) = \frac{\text{confidence}(A \to B)}{\text{support}(B)} = \frac{\text{support}(A \cup B)}{\text{support}(A) \times \text{support}(B)} $$

**Filtering Strategy:**
1. Generate itemsets with support ≥ 5%
2. Generate rules with confidence ≥ 60%
3. Filter rules with lift ≥ 1.0
4. Rank by lift (primary) and confidence (secondary)
5. Report top 10 for human interpretation

## 4. Results
### 4.1 Analysis 1: Student-Problem Interactions
**Research Question:** Which problem-solving behaviors co-occur during student work sessions?

**Top 10 Frequent Itemsets:**
| Rank | Itemset | Support | Interpretation | 
|-|-|-|-|
| 1 | {no_explanation} | 0.996 | Nearly all avoid explanations | 
| 2 | {no_hints} | 0.990 | Hints rarely used | 
| 3 | {no_explanation, no_hints} | 0.986 | Minimal help-seeking | 
| 4 | {no_answer} | 0.884 | Answer key rarely viewed |
| 5 | {no_answer, no_explanation} | 0.881 | Help resources ignored |
| 6 | {no_answer, no_hints} | 0.879 | Triple help avoidance | 
| 7 | {no_answer, no_explanation, no_hints} | 0.876 | Complete self-direction |
| 8 | {no_wrongs} | 0.828 | Correct on first try | 
| 9 | {no_explanation, no_wrongs} | 0.827 | Success without explanations | 
| 10 | {no_hints, no_wrongs} | 0.825 | Success without hints |
 
**Top 10 Association Rules:**
| Rank | Rule | Support | Confidence | Lift | Interpretation|
|-|-|-|-|-|-|
| 1 | {correct_after_help, no_hints, no_answer, no_explanation} → {few_wrongs} | 0.083 | 0.919 | 6.509 | Students who succeed with no help had few errors | 
| 2 | {correct_after_help, no_hints, no_answer} → {few_wrongs} | 0.084 | 0.913 | 6.467 | Same pattern without explanation filter | 
| 3 | {correct_after_help, no_hints, no_answer} → {no_explanation, few_wrongs} | 0.083 | 0.899 | 6.442 | Reinforces explanation avoidance | 
| 4 | {correct_after_help, no_answer, no_explanation} → {no_hints, few_wrongs} | 0.083 | 0.867 | 6.323 | Hints unnecessary when successful | 
| 5 | {no_hints, few_wrongs} → {correct_after_help, no_answer, no_explanation} | 0.083 | 0.602 | 6.323 | Reverse: few errors predict success | 
| 6 | {correct_after_help, no_answer, no_explanation} → {few_wrongs} | 0.085 | 0.890 | 6.305 | Core success pattern | 
| 7 | {few_wrongs} → {correct_after_help, no_answer, no_explanation} | 0.085 | 0.601 | 6.305 | Error rate strongly predicts outcome | 
| 8 | {correct_after_help, no_answer} → {no_hints, few_wrongs} | 0.084 | 0.863 | 6.290 | Help avoidance when succeeding | 
| 9 | {no_hints, few_wrongs} → {correct_after_help, no_answer} | 0.084 | 0.612 | 6.290 | Minimal errors → minimal help |
| 10 | {correct_after_help, no_answer} → {few_wrongs} | 0.086 | 0.885 | 6.270 | Strongest single-consequent rule |

**Key Visualizations:**
![Analysis 1 Results](/results/vis/A1_results_analysis.png)
*Figure 1:* (Top-left) Item support distribution showing dominance of help avoidance. (Top-right) Co-occurrence heatmap revealing high correlation between no_hints/no_answer/no_explanation. (Bottom-left) Support vs Confidence scatter with lift bubble size. (Bottom-right) Support vs Lift showing high-lift rules cluster at low support.

**Pattern Summary:**
- Self-directed success: Students who eventually succeed (correct_after_help) overwhelmingly avoid hints, answers, and explanations (lift 6.3-6.5×)
- Few errors matter: Rules consistently highlight few_wrongs (0-2 errors) as a strong predictor of eventual correctness
- Help paradox: The strongest associations involve not using help resources, suggesting either:
  1. Successful students don't need help
  2. Help-seeking occurs only when deeply struggling (not captured in "correct_after_help" outcome)
- Problem type irrelevance: No strong rules involve problem_type, suggesting behavior patterns transcend question format

### 4.2 Analysis 2: Student-Unit Aggregations
**Research Question:** Which combinations of aggregated behaviors predict success/failure on unit tests?

**Top 10 Frequent Itemsets:**
| Rank | Itemset | Support | Interpretation | 
|-|-|-|-|
| 1 | {low_wrongs} | 0.959 | Nearly all have minimal errors | 
| 2 | {high_completion} | 0.869 | Most complete assigned work | 
| 3 | {high_completion, low_wrongs} | 0.836 | Success duo: completion + accuracy | 
| 4 | {low_effort, low_wrongs} | 0.700 | Minimal interactions with few errors | 
| 5 | {low_effort} | 0.700 | Standard effort level | 
| 6 | {low_struggle} | 0.700 | Low wrong/correct ratio | 
| 7 | {low_struggle, low_wrongs} | 0.700 | Error metrics align | 
| 8 | {typical_help} | 0.700 | Normal help-seeking frequency | 
| 9 | {low_wrongs, typical_help} | 0.689 | Help + accuracy combination | 
| 10 | {low_effort, typical_help} | 0.614 | Standard engagement profile |

**Top 10 Association Rules:**
| Rank | Rule | Support | Confidence | Lift | Interpretation|
|-|-|-|-|-|-|
| 1 | {low_wrongs, low_struggle, typical_help, low_completion} → {low_effort, few_problems} | 0.062 | 0.803 | 3.258 | Low completion + typical help → low effort on few problems |
| 2 | {low_struggle, typical_help, low_completion} → {low_effort, low_wrongs, few_problems} | 0.062 | 0.803 | 3.258 | Same pattern emphasizing accuracy | 
| 3 | {low_struggle, typical_help, low_completion} → {low_effort, few_problems} | 0.062 | 0.803 | 3.258 | Core low-completion pattern | 
| 4 | {low_completion, low_struggle, low_score} → {low_effort, few_problems} | 0.056 | 0.794 | 3.221 | Low score adds to pattern |
| 5 | {low_struggle, low_wrongs, low_completion, low_score} → {low_effort, few_problems} | 0.056 | 0.794 | 3.221 | Full struggling student profile | 
| 6 | {low_completion, low_struggle, low_score} → {low_effort, low_wrongs, few_problems} | 0.056 | 0.794 | 3.221 | Emphasizes accuracy despite low score | 
| 7 | {low_wrongs, low_completion, typical_help, low_score} → {low_effort, few_problems} | 0.052 | 0.790 | 3.205 | Help-seeking variant | 
| 8 | {low_wrongs, low_struggle, low_completion} → {low_effort, few_problems} | 0.075 | 0.778 | 3.157 | Strongest support rule | 
| 9 | {low_struggle, low_completion} → {low_effort, low_wrongs, few_problems} | 0.075 | 0.778 | 3.157 | Simplified version of Rule 8 | 
| 10 | {low_struggle, low_completion} → {low_effort, few_problems} | 0.075 | 0.778 | 3.157 | Minimal-item version |

**Key Visualizations:**
![Analysis 2 Results](/results/vis/A2_results_analysis.png)
*Figure 2:* (Top-left) Item support showing high completion and low errors dominate. (Top-right) Co-occurrence heatmap revealing tight clustering between low_effort, low_struggle, and typical_help. (Bottom-left) Rules show clear separation between high-score (upper right) and low-score (lower left) clusters. (Bottom-right) Most actionable rules cluster at support 0.05-0.08 with lift 3.0-3.3.

**Pattern Summary:**
- Low completion trap: Students with low_completion (<70% finished) strongly associate with few_problems and low_effort (lift ~3.2×), forming a disengagement profile
- Score paradox: Rules emphasize low_score patterns, but lack symmetric high_score rules due to balanced dataset (50/50 split)
- Help-seeking neutrality: typical_help appears in rules without strong directional effect on outcomes
- Effort-completion link: High completion_rate breaks the low_effort → few_problems cycle, suggesting completion as a key intervention target

### 4.3 Analysis 3: Help-Seeking Patterns
**Research Question:** Which help-seeking patterns distinguish students achieving mastery from those struggling?

**Top 10 Frequent Itemsets:**
| Rank | Itemset | Support | Interpretation | 
|-|-|-|-|
| 1 | {low_help} | 0.986 | Help-seekers still use minimal resources | 
| 2 | {rare_hints} | 0.982 | <0.5 hints per problem | 
| 3 | {low_help, rare_hints} | 0.978 | Double minimal-use pattern |
| 4 | {no_explanation} | 0.951 | Explanations almost never used |
| 5 | {low_help, no_explanation} | 0.945 | Help comes from hints/answers, not explanations | 
| 6 | {no_explanation, rare_hints} | 0.939 | Confirms explanation avoidance | 
| 7 | {low_help, no_explanation, rare_hints} | 0.937 | Triple minimal-use cluster |
| 8 | {tries_before_help} | 0.771 | Majority attempt before seeking help | 
| 9 | {low_help, tries_before_help} | 0.763 |Attempt-first + minimal help | 
| 10 | {rare_hints, tries_before_help} | 0.759 | Attempt-first + rare hints |

**Top 10 Association Rules:**
| Rank | Rule | Support | Confidence | Lift | Interpretation|
|-|-|-|-|-|-|
| 1 | {algebra, low_help} → {middle, rare_hints} | 0.050 | 0.764 | 3.082 | Middle school algebra uses minimal hints | 
| 2 | {algebra, rare_hints} → {middle, low_help} | 0.050 | 0.767 | 3.050 | Algebra students avoid help | 
| 3 | {algebra} → {middle, rare_hints, low_help} | 0.050 | 0.748 | 3.049 | Core algebra-middle school pattern |
| 4 | {algebra} → {middle, rare_hints} | 0.051 | 0.755 | 3.045 | Algebra = rare hints |
| 5 | {algebra} → {middle, low_help} | 0.051 | 0.758 | 3.016 | Algebra = low help | 
| 6 | {algebra} → {middle} | 0.052 | 0.779 | 2.958 | Strongest algebra-grade link |
| 7 | {algebra, low_help} → {middle} | 0.051 | 0.774 | 2.941 | Low help variant |
| 8 | {algebra, rare_hints} → {middle} | 0.051 | 0.773 | 2.938 | Rare hints variant |
| 9 | {algebra, rare_hints, low_help} → {middle} | 0.050 | 0.772 | 2.932 | Full minimal-help profile |
| 10 | {ratios, low_help, no_self_correction} → {middle, rare_hints} | 0.062 | 0.669 | 2.697 | Ratios unit without self-correction |

**Key Visualizations:**
![Analysis 3 Results](/results/vis/A3_results_analysis.png)
Figure 3: (Top-left) Item support dominated by help avoidance. (Top-right) Co-occurrence heatmap shows grade level (elementary/middle) clusters separately from help behaviors. (Bottom) Rules reveal curriculum-specific patterns (algebra, ratios) rather than mastery predictors.

**Pattern Summary:**
- Curriculum dominance: Strongest rules involve subject-grade associations (algebra→middle school) rather than help-seeking→success patterns
- Elementary-middle divide: Rules cluster by grade level, suggesting different help-seeking cultures:
  - Elementary: More help_first behavior
  - Middle school: More tries_before_help + algebra
- Success pattern absence: No strong rules linking help strategies to high_success outcome, possibly due to:
  1. Selection bias (analysis only includes help-seekers, excluding successful non-help-seekers)
  2. Help-seeking as a distress signal rather than strategy
- Self-correction irrelevance: self_corrects appears in few rules, suggesting observed sequences don't capture learning

## 5. Discussion
### 5.1 Educational Insights
#### Cross-Analysis Synthesis:
##### 1. The "Silent Majority" Phenomenon
Across all three analyses, the dominant pattern is help avoidance:
- 87.6% of problem attempts use no help resources (Analysis 1)
- 98.6% of help-seekers still use "low help" (<1 resource per problem) (Analysis 3)
- Rules consistently highlight no_hints, no_answer, no_explanation with support >95%

**Implication:** Current help infrastructure may be underutilized due to:
- Stigma: Students avoid help to maintain appearance of competence
- Friction: Help requires extra clicks/effort vs. guessing
- Ineffectiveness: Past help experiences didn't improve outcomes
- Self-selection: High achievers don't need help, low achievers don't seek it

##### 2. The Completion Gap
Analysis 2 reveals low_completion (<70% finished) as a critical marker:

- Strongly associates with few_problems, low_effort, low_score (lift 3.2×)
- Forms a self-reinforcing disengagement cycle
- More predictive than help-seeking behaviors

**Implication:** Early completion monitoring is more actionable than help-seeking interventions. Students who disengage early (e.g., <3 problems completed in first week) require immediate outreach.

##### 3. The Effort Paradox
Rules link low_effort (minimal interactions per problem) with both success and failure:

- Success pathway: {low_effort, low_wrongs, high_completion} → proficiency allows quick work
- Failure pathway: {low_effort, few_problems, low_completion} → disengagement

**Implication:** Effort must be contextualized by completion. Low effort + high completion = mastery; low effort + low completion = disengagement.

##### 4. Curriculum-Specific Patterns
Analysis 3 reveals subject-grade associations:

- Algebra problems in middle school → rare hints (lift 3.08×)
- Ratios units → no self-correction behavior
- Elementary students → more help_first (seek help before attempting)

**Implication:** Help-seeking norms vary by developmental stage and subject. Middle school algebra may benefit from more scaffolded hint systems, while elementary needs "productive struggle" encouragement.

##### 5. The "Correct After Help" Mystery
Strongest rules in Analysis 1 involve correct_after_help + help avoidance:

`{correct_after_help, no_hints, no_answer} → few_wrongs (lift 6.5×)`

**Implication:** This counterintuitive pattern suggests:
- Students who eventually succeed don't rely on platform help (may use external resources like friends, parents, tutors)
- Platform logs only capture one help channel in a multi-channel ecosystem

### 5.2 Actionable Recommendations
#### For Instructional Designers:

1. Redesign Help Hierarchy
- Current: Hints → Explanations → Answer (sequential)
- Proposed: Offer "quick hint" (1-click, no explanation) vs. "deep dive" (explanation + worked example)
- Rationale: 95% avoid explanations; students may want faster, lighter help

2. Completion-Based Alerts
- Trigger: <70% completion in first 3 assignments
- Action: Automated outreach + suggest reduced problem sets
- Rationale: Low completion predicts poor outcomes (lift 3.2×) better than help-seeking

3. "Productive Struggle" Coaching
- Target: Elementary students showing help_first pattern
- Intervention: Require 1 attempt before hint button activates
- Rationale: Middle school students naturally try before help (77%); elementary can learn this

#### For Teachers:
1. Effort-Completion Dashboard
- Metrics: Plot students in 2D space (completion × effort_per_problem)
- Quadrants:
  - High completion + Low effort = Proficient (praise)
  - High completion + High effort = Struggling (tutor)
  - Low completion + Low effort = Disengaged (immediate contact)
  - Low completion + High effort = Overwhelmed (reduce workload)
- Rationale: Effort alone is ambiguous; completion provides context
2. Curriculum-Specific Scaffolding
- Elementary: Add friction before hints (require attempt first)
- Rationale: Analysis 3 shows subject-grade differences in help-seeking norms

#### For Platform Developers:
1. Multi-Channel Help Analytics
- Action: Add "external help" button ("I got help from: parent/friend/internet")
- Purpose: Distinguish self-sufficiency from external support
- Rationale: Current logs miss external help, creating false "self-directed success" narrative
2. Adaptive Problem Sets
- Rule: If student completes 5 problems with no_wrongs, fast, no_help → offer skip to harder problems
- Rule: If student shows few_problems, low_completion → reduce assignment to 3 must-complete problems
- Rationale: One-size-fits-all assignments ignore proficiency variance

### 5.3 Limitations
#### Data Limitations:
1. Selection Bias in Analysis 3
- Only analyzed students who sought help (231K of 638K assignments)
- Excludes "silent achievers" who never need help
- Rules don't generalize to full population
2. Temporal Myopia
- Static snapshot analysis across 4 years
- Cannot distinguish:
  - First-time struggles vs. repeated patterns
  - Beginning-of-year behaviors vs. end-of-year mastery
  - Individual growth trajectories
- Rules assume stationarity (student behaviors don't evolve)
3. Binning Information Loss
- Continuous variables (time, hint count) converted to categorical bins
- May miss non-linear relationships (e.g., 3 hints optimal, not 0 or 10)
- Arbitrary thresholds (e.g., "low" completion = <70%) lack theoretical grounding
4. Missing Metadata
- 65% of assignments lack grade-level labels (Analysis 3)
- No student demographics (prior achievement, SES, language)
- No teacher effects (instructional quality)

#### Methodological Limitations:
1. Cold Start Problem
- Requires ≥3 completed problems (Analysis 2/3)
- Misses earliest at-risk signals (students who quit after 1 problem)
- Rules don't help identify immediate dropouts
2. Platform Specificity
- ASSISTments has unique features (hint/explanation/answer system)
- Rules may not transfer to:
  - LMS with different help affordances (e.g., Coursera discussion forums)
  - Face-to-face classes

#### Computational Limitations:
1. Memory Constraints
- Required chunked processing for 16M+ action logs
- Full cross-student temporal analysis infeasible
- Could not explore rare items (filtered to support ≥5%)
2. Hyperparameter Sensitivity
- Results depend on chosen thresholds:
  - Min support (5%), confidence (60%), lift (1.0)
  - Binning boundaries (e.g., "few hints" = 0-2)

## 6. Conclusion
For educators, these findings suggest prioritizing completion-based interventions over help-seeking encouragement. A proposed dashboard would classify students into four quadrants (completion × effort) to trigger differentiated responses: praise for efficient achievers, tutoring for high-effort strugglers, immediate outreach for disengaged students, and workload reduction for overwhelmed learners.

For platform designers, results recommend redesigning help hierarchies to reduce friction (lighter "quick hints" vs. heavy explanations), implementing adaptive problem sets that respond to proficiency signals, and tracking external help to avoid conflating self-sufficiency with lack of social support.

The Apriori algorithm, though computationally constrained by itemset explosion, proved effective for this domain due to naturally sparse transaction structures (29-37% density). For larger-scale analyses, future work should explore FP-Growth or ECLAT variants to handle higher-dimensional behavior spaces while maintaining interpretability for educational stakeholders.


## 7. Video Presentation
[![Final Presentation](demo/CSC172_Bautista_Final.mp4)](demo/CSC172_Bautista_Final.mp4)

## References
1. Agrawal, R., & Srikant, R. (1994). Fast Algorithms for Mining Association Rules. VLDB.
2. mlxtend Documentation: https://rasbt.github.io/mlxtend/
3. Mendez, G., Bustos, B., & Esteban, A. (2023). EDM Cup 2023: Predicting Student Outcomes Using Clickstream Data. Kaggle Competition. https://www.kaggle.com/competitions/edm-cup-2023/

## Appendix: Full Results

**Note:** I did not include the processed datasets and complete rule files in the repository because they consume too much space (>1GB). To generate these files, you need to run the [Apriori Rule Mining Notebook](#code-artifacts).

### Analysis Setup
To replicate and generate the files listed below:

1. Environment Setup
```bash
python -m venv ven
source venv/bin/activate  # or `venv\Scripts\activate` on Windows
pip install -r requirements.txt
```
2. Download Dataset
  - Visit https://www.kaggle.com/competitions/edm-cup-2023/data
  - Download all CSV files to `dataset/` directory
3. Run Analysis
  - Open and run sequencially the [`notebooks/asm_apriori.ipynb`](notebooks/asm_apriori.ipynb)
### Complete Rule Sets
All raw rules and itemsets are available in [`results/raw/`](results/raw/):

**Analysis 1 - Student-Problem Interactions:**
- Full CSV: [`A1_analysis_rules.csv`](results/raw/A1_analysis_rules.csv)
- Frequent itemsets: [`A1_analysis_itemsets.csv`](results/raw/A1_analysis_itemsets.csv)

**Analysis 2 - Student-Unit Aggregations:**
- Full CSV: [`A2_analysis_rules.csv`](results/raw/A2_analysis_rules.csv)
- Frequent itemsets: [`A2_analysis_itemsets.csv`](results/raw/A2_analysis_itemsets.csv)

**Analysis 3 - Help-Seeking Patterns:**
- Full CSV: [`A3_analysis_rules.csv`](results/raw/A3_analysis_rules.csv)
- Frequent itemsets: [`A3_analysis_itemsets.csv`](results/raw/A3_analysis_itemsets.csv)

### Visualization Gallery
All visualizations available in [`results/vis/`](results/vis/):

**Analysis 1 - Student-Problem Interactions:**
- [`A1_results_analysis.png`](results/vis/A1_results_analysis.png) - Problem interaction patterns

**Analysis 2 - Student-Unit Aggregations:**
- [`A2_results_analysis.png`](results/vis/A2_results_analysis.png) - Unit test prediction patterns

**Analysis 3 - Help-Seeking Patterns:**
- [`A3_results_analysis.png`](results/vis/A3_results_analysis.png) - Help-seeking mastery patterns

**Processed Datasets (Generated by notebook):**
Located in `dataset/dataframes/` (not included in repo):
- `action_logs.parquet` (16.25M rows)
- `action_problem.parquet` (5.14M rows)
- `action_problem_transactions.parquet` (5.14M rows)
- `unit_test_transactions.parquet` (42.2K rows)
- `help_patterns_transactions.parquet` (204K rows)
- ...

## Github Pages
View this project site: [https://caineirb.github.io/CSC172-AssociationMining-Bautista/](https://caineirb.github.io/CSC172-AssociationMining-Bautista/)