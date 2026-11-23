# econ3916-lab18-model-evaluation
# Model Evaluation: The Curve, and What It Cannot Tell You

## Objective
I wanted to see what ROC and precision-recall curves can tell me about a classifier for a rare outcome, and what they can't tell me once I have to pick a threshold and pay for my mistakes.

## Methodology
- I simulated 8,000 cases of a rare outcome and split them into training and test rows. 11.8% of the test rows were positive.
- I fit a logistic regression and drew its ROC curve and precision-recall curve on the test rows. I recorded the ROC AUC and the average precision.
- I fit a boosted-tree model and compared its ROC AUC with the logistic regression's on the same test rows.
- To check whether the AUC gap was more than noise, I ran a paired bootstrap. I drew 500 resamples of the test rows, scored both models on each resample, took the difference and read off a 95% interval.
- I built an expected-cost curve where a miss costs ten times as much as a false alarm. I picked the cheapest threshold using only the training rows. Then I priced that threshold once on the test rows and compared it with the default threshold of 0.5.
- I repeated the threshold choice for both models at cost ratios from 2:1 to 50:1, each time choosing on the training rows, and checked whether the cheaper model changed.
- I wrote a small function, `roc_point()`, that returns the false positive rate and true positive rate at a single threshold. I used it to find where my chosen threshold sits on the ROC curve.

## Key Findings
- **The ROC curve looks better than the precision-recall curve.** The logistic regression had a ROC AUC of 0.8468 but an average precision of only 0.4651. Only 11.8% of test rows were positive, so there are many more negatives than positives. The false positive rate divides false alarms by that large pool of negatives, so many false alarms barely move the ROC curve. Precision counts them directly against the true positives, so the precision-recall curve shows the cost. A model guessing at random would have an average precision near the 11.8% positive rate. That makes 0.4651 a real improvement, just a less impressive-looking one.
- **I can't tell the two models apart on AUC.** The boosted tree scored 0.8431, so the logistic regression was ahead by +0.0038 on the full test set. The paired bootstrap gave a 95% interval of [-0.0020, +0.0094] for that difference, and the interval includes zero. On this test set I can't say that either model ranks cases better than the other.
- **The threshold mattered more than the curve.** With a miss costing ten times a false alarm, the cheapest threshold on the training rows was 0.0905. On the test rows it cost 0.4692 per case, against 0.9437 at the default 0.5, which is about half (costs count a false alarm as 1 and a miss as 10). The ROC curve was the same in both cases; only the point I chose on it changed. AUC sums up the whole curve and says nothing about where on it I should be.
- **Where my threshold sits on the curve.** At 0.0905, `roc_point()` gave a false positive rate of 0.296 and a true positive rate of 0.823 on the test rows. I catch most of the positive cases, and in return I flag about three in ten negatives. That trade makes sense when a miss costs ten times as much as a false alarm.
- **Changing the cost ratio:** the cheaper model did change. The logistic regression was cheaper at 2:1 and at every ratio from 10:1 to 50:1, and the boosted tree only at 5:1. The gaps at the low ratios were small. With AUCs this close, I would not pick a model from those differences without an interval on the cost gap at the ratio the business actually faces.

Overall, the curves tell me how well a model ranks cases. They don't tell me which threshold to use, what my mistakes will cost, or whether a small AUC gap between two models is real.
