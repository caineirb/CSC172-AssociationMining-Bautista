# CSC172 Association Rule Mining Project Progress Report
**Student:** Caine Ivan R. Bautista, 2022-0378
**Date:** December 14, 2025  
**Repository:** https://github.com/caineirb/CSC172-AssociationMining-Bautista

## 📊 Current Status
| Milestone | Status | Notes |
|-----------|--------|-------|
| Dataset Preparation | ✅ Completed | Almost 6M transactions processed |
| Data Preprocessing | ✅ In Progress | Memory Limitation still working |
| EDA & Visualization | ⏳ Not Started | Initial run tomorrow |
| Apriori Implementation | ⏳ Pending | Initial run tomorrow |
| Rule Evaluation | ⏳ Not Started | Planned for next day |


## 1. Dataset Progress
- **Total transactions:** 5,245,859
- **Unique items:** 67 → filtered to 28 (support > 0.05)
- **Matrix size:** 5,245,859 transactions × 28 items (40.0% density)
- **Preprocessing applied:** 
  * Missing values removed
  * Binning applied (7 categories)
  * Basketing applied (8 baskets)
  * One-hot encoding
  * Infrequent item filtering (support > 0.05)

**Sample transaction preview:**

Transaction 1: ['curriculum_EngageNY' 'tutoring_answer' 'action_wrong_response_once'
 'basket_requested_help' 'probtype_Number' 'timing_very_fast'
 'basket_had_correct_response' 'action_correct_response_once'] ...
Transaction 2: ['tutoring_no_tutoring' 'curriculum_EngageNY' 'attempts_limited'
 'skill_advanced' 'content_simple' 'probtype_Ungraded_Open_Respon'
 'grade_middle' 'multipart_problem'] ...
Transaction 3: ['tutoring_no_tutoring' 'curriculum_EngageNY' 'attempts_limited'
 'skill_advanced' 'content_simple' 'probtype_Ungraded_Open_Respon'
 'grade_middle' 'multipart_problem'] ...
Transaction 4: ['curriculum_EngageNY' 'tutoring_answer' 'action_wrong_response_once'
 'basket_requested_help' 'probtype_Number' 'basket_had_correct_response'
 'action_correct_response_once' 'content_simple'] ...
Transaction 5: ['tutoring_no_tutoring' 'curriculum_EngageNY' 'attempts_limited'
 'skill_advanced' 'content_simple' 'probtype_Ungraded_Open_Respon'
 'grade_middle' 'multipart_problem'] ...


## 2. EDA Progress
**Key Findings (so far):**
![Item Frequency Distribution](results/item_frequencies.png)
- Top 5 items: score_visible(99.1%), skill_advanced(97.4%), curriculum_EngageNY(85.2%), attempts_moderate(69.1%), basket_had_correct_response(68.4%)
- Average basket size: 11.5 items
- 0% transactions contain 1-3 items

**Current Metrics:**
| Metric | Value |
|--------|-------|
| Transactions cleaned | 5,245,859/5,245,859 (100.0%) |
| Sparsity reduced | 60.0% |
| Top item support | score_visible: 0.991 |

## 3. Challenges Encountered & Solutions
| Issue | Status | Resolution |
|-------|--------|------------|
| High matrix sparsity | ✅ Fixed | Filtered to top 28 items (min_support > 0.05) |
| Memory usage during Apriori | ✅ Fixed | Progressive support levels + low_memory mode |
| Large itemset generation | ✅ Fixed | Limited max_len to 3-itemsets |
| High-dimensional data | ✅ Fixed | Binning (7 categories) + Basketing (8 patterns) |
| Item filtering trade-off | ⏳ Ongoing | Tuning min_support threshold (0.05 currently) |

## 4. Next Steps (Before Final Submission)
- [ ] Code cleanup
- [ ] Complete co-occurrence heatmap
- [ ] Generate top 25 rules with metrics
- [ ] Create rule scatter plot 
- [ ] Record 5-min demo video
- [ ] Write complete README.md 