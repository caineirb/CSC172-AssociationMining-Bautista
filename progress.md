# CSC172 Association Rule Mining Project Progress Report
**Student:** Caine Ivan R. Bautista, 2022-0378
**Date:** December 18, 2025  
**Repository:** https://github.com/caineirb/CSC172-AssociationMining-Bautista

## 📊 Current Status
| Milestone | Status | Notes |
|-----------|--------|-------|
| Dataset Preparation | ✅ Completed | More than 5 million transactions processed |
| Data Preprocessing | ✅ Completed | Ensure proper memory management and saving dataframes after processing |
| EDA & Visualization | ✅ Completed | Visualized [Item support, Co-occurrence heatmap, Support vs Confidence, Support vs Lift] |
| Apriori Implementation | ✅ Completed | Done |
| Rule Evaluation | ✅ Completed | Done |


## 1. Dataset Progress
### Transaction 1:
- **Total transactions:** 5,140,889
- **Unique items:** 24 engineered items
- **Matrix size:** 5,140,889 transactions × 24 items (29.17% density)
- **Preprocessing applied:** 
  * Merging 'action_logs.csv' with 'problem_details.csv'
  * Columns filtering
  * Action filtering (keep only student-problem actions)
  * Grouping action_logs by ['assignment_log_id', 'problem_id']
  * Aggregation of grouped attempts rows per actions then droping actions resulting to NA
  * Binning applied (9 categories)
  * Classify boolean items (4 categories)
  * Basketing applied (24 items per basket)
  * One-hot encoding

- **Sample transaction preview:** 

| transaction_id | values | 
|----------------|--------|
| 1EZQH1WY9H_1R8PA6HFZW | [no_hints, few_wrongs, slow, answer_req, no_explanation, correct_after_help, algebra] |
| 12F78MMP2T_O6T088UQ | [no_hints, no_wrongs, medium, no_answer, no_explanation, correct_first_try, other] |
| 1T3X6XCIMA_MV8V6YRBG | [no_hints, no_wrongs, medium, no_answer, no_explanation, gave_up, open] |
| 1YZC5GRZGO_1RR3LLRUFT | [no_hints, no_wrongs, fast, no_answer, no_explanation, gave_up, open] |
| 2GET2L0998_1OH2WJVLRE | [no_hints, no_wrongs, fast, no_answer, no_explanation, correct_first_try, numeric] |

### Transaction 2:
- **Total transactions:** 42,244
- **Unique items:** 19 engineered items
- **Matrix size:** 42,244 transactions × 19 items (36.84% density)
- **Preprocessing applied:** 
  * Merging 'action_logs.csv' with 'training_unit_test_scores.csv' and 'assignment_relationships.csv'
  * Use the same actions in the 1st transaction analysis
  * Group the action_logs by ['assignment_log_id'] 
  * Aggregate the total of actions per assignment group then droping rows resulting to NA
  * After merging, calculate the various statistics for each merged group by ['unit_test_assignment_log_id']
  * Apply binning to all statistics gathered from aggregation results (19 categories)
  * Basketing applied (19 items per basket)
  * One-hot encoding
- **Sample transaction preview:** 

| transaction_id | values | 
|----------------|--------|
| 1KPCIEDF9V | [high_score, typical_help, low_wrongs, high_completion, med_problems, low_struggle, low_effort] |
| 21YMFN3ZCZ | [high_score, typical_help, low_wrongs, high_completion, many_problems, low_struggle, low_effort] |
| 29FO15HZJY | [low_score, high_help, low_wrongs, high_completion, few_problems, med_struggle, med_effort] |
| NVCJECPF7 | [low_score, typical_help, low_wrongs, low_completion, few_problems, low_struggle, low_effort] |
| 15DHHGWMUX | [low_score, typical_help, low_wrongs, low_completion, few_problems, med_struggle, low_effort] |

### Transaction 3:
- **Total transactions:** 205,693
- **Unique items:** 27 engineered items
- **Matrix size:** 205,693 transactions × 27 items (33.33% density)
- **Preprocessing applied:** 
  * Merging 'action_logs.csv' with 'sequence_details.csv'
  * Filter actions limiting to classified as 'asking help' ['hint_requested', 'explanation_requested', 'answer_requested']
  * Group the action_logs by ['assignment_log_id'] 
  * Aggregate the total of help actions per assignment group  then droping rows resulting to NA
  * Add metadata for each sequence found in sequence_details
  * Apply binning to all statistics gathered from aggregation results (15 categories)
  * Classify boolean items (4 categories)
  * Classify grade_level of based on 'sequence_folder_path_level_2' (4 categories)
  * Classify subject of based on 'sequence_folder_path_level_3' (5 categories)
  * Basketing applied (27 items per basket)
  * One-hot encoding
- **Sample transaction preview:**

| transaction_id | values | 
|----------------|--------|
| YKLKRCDDM | [rare_hints, low_help, no_explanation, some_answer_req, high_success, help_first, no_self_correction, elementary, unknown] |
| 1857XD6M3P | [rare_hints, low_help, no_explanation, some_answer_req, low_success, tries_before_help, no_self_correction, elementary, unknown] |
| ZEP44HUK0 | [rare_hints, low_help, no_explanation, some_answer_req, low_success, tries_before_help, self_corrects, elementary, unknown] |
| 10HKTZ5OB3 | [rare_hints, low_help, no_explanation, frequent_answer_req, med_success, help_first, self_corrects, elementary, ratios] |
| YW2HR4TPT | [rare_hints, low_help, no_explanation, some_answer_req, med_success, tries_before_help, self_corrects, elementary, unknown] |

## 2. EDA Progress
**Key Findings (so far):**
### 1. Student-problem interactions (behavior co-occurrence)
![Item Frequency Distribution](results/vis/A1_results_analysis.png)
- Top 5 items: no_explanation (99.6%), no_hints (99.0%), no_answer (88.4%),  no_wrongs (82.8%), fast (55.9%)
- Average basket size: 7.0 items

### 2. Student-unit aggregations (success/failure prediction)
![Item Frequency Distribution](results/vis/A2_results_analysis.png)
- Top 5 items: low_wrongs (95.9%), high_completion (86.9%), low_struggle (70.0%), low_effort (70.0%), typical_help (70.0%)
- Average basket size: 6.9995 items

### 3. Student-problem interactions (behavior co-occurrence)
![Item Frequency Distribution](results/vis/A1_results_analysis.png)
- Top 5 items: low_help (98.6%), rare_hints (98.2%), no_explanation (95.1%), tries_before_help (77.1%), unknown (65.2%)
- Average basket size: 9.0 items


**Current Metrics:**
| Metric | Transaction 1 | Transaction 2 | Transaction 3 |
| - | - | - | - |
| Transactions cleaned | 100% | 100% | 100% | 
| Sparsity | 70.83% | 63.16% | 66.67% |
| Top item support |  no_explanation: 0.996 |  no_explanation: 0.996 | low_wrongs: 0.959 | low_help: 0.986 | 

## 3. Challenges Encountered & Solutions
| Issue | Status | Resolution |
|-------|--------|------------|
| Memory usage due to large dataset | ✅ Fixed | Ensure proper memory management and saving dataframes after processing |
| Memory usage during Apriori | ✅ Fixed | Progressive support levels + low_memory mode |
| Large itemset generation | ✅ Fixed | Limited max_len to 3-itemsets |
| High-dimensional data | ✅ Fixed | Grouping then Binning (9 categories) + Basketing (7 patterns) |

## 4. Next Steps (Before Final Submission)
- [X] Code cleanup
- [X] Complete co-occurrence heatmap
- [X] Create rule scatter plot 
- [ ] Record 5-min demo video
- [ ] Write complete README.md 