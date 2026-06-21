# TakeMeter: Reddit Career Comment Classifier



**Demo Video:** https://www.loom.com/share/4f235796233f4bb9b709f1e1ebafd452


## Project Overview

TakeMeter is a text classification project that labels Reddit comments from data science and career advice discussions. The goal is to classify each comment based on what kind of information it provides to a reader.

The project uses four labels:

| Label                 | Meaning                                                                                                                      |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `actionable_guidance` | Specific advice, steps, strategies, or recommendations a reader can follow.                                                  |
| `experience_report`   | The writer describes their own background, job search, career path, interviews, education, workplace experience, or outcome. |
| `unsupported_claim`   | A broad or confident claim without enough evidence or reasoning inside the comment.                                          |
| `generic_noise`       | Vague, very short, sarcastic, promotional, random, reaction-only, or low-information content.                                |

The project compares two approaches:

1. A zero-shot Groq LLM baseline using a classification prompt.
2. A fine-tuned `distilbert-base-uncased` model trained on my labeled Reddit dataset.

---

## Dataset

The dataset contains 200 manually labeled Reddit comments.

Final label distribution:

| Label                 |   Count |
| --------------------- | ------: |
| `actionable_guidance` |      88 |
| `generic_noise`       |      49 |
| `unsupported_claim`   |      33 |
| `experience_report`   |      30 |
| **Total**             | **200** |

The data was split into train, validation, and test sets:

| Split      | Count |
| ---------- | ----: |
| Train      |   140 |
| Validation |    30 |
| Test       |    30 |

Training label distribution:

| Label                 | Count |
| --------------------- | ----: |
| `actionable_guidance` |    62 |
| `generic_noise`       |    34 |
| `unsupported_claim`   |    23 |
| `experience_report`   |    21 |

All four labels appeared in the train, validation, and test splits. The main challenge was not missing labels, but the small dataset size, moderate class imbalance, and subtle boundary cases between labels.

---

## Labeling Decisions

The most important labeling rule was to classify the **main function** of the comment.

If a comment contained personal experience but mainly told the reader what to do, I labeled it `actionable_guidance`. If the comment mainly reported what happened to the writer, I labeled it `experience_report`.

Some edge case rules:

* A short vague comment, even if it uses data science words, is usually `generic_noise`.
* A broad market claim without support is usually `unsupported_claim`.
* A comment with concrete steps like “learn SQL,” “build projects,” or “network” is usually `actionable_guidance`.
* A comment with “I applied,” “I worked,” or “I got hired” is usually `experience_report` only if the personal story is the main point.

---

## Models

### Zero-Shot Baseline

The baseline used a Groq-hosted LLM with a system prompt defining the four labels. The model received each Reddit comment and returned exactly one label.

This baseline did not train on my dataset. It tested how well a general instruction-following LLM could classify the comments using only label definitions and examples in the prompt.

### Fine-Tuned Model

The fine-tuned model used `distilbert-base-uncased` with a sequence classification head.

Initial fine-tuning with the default settings performed poorly because the model over-predicted only a few labels. To improve this, I used class-weighted training.

Final fine-tuning settings:

| Setting          | Value                        |
| ---------------- | ---------------------------- |
| Base model       | `distilbert-base-uncased`    |
| Epochs           | 10                           |
| Learning rate    | 5e-5                         |
| Train batch size | 8                            |
| Eval batch size  | 32                           |
| Loss function    | Class-weighted cross entropy |
| Trainer          | Custom `WeightedTrainer`     |

I used class-weighted training because the dataset was moderately imbalanced. `actionable_guidance` had the most training examples, while `experience_report` and `unsupported_claim` had fewer examples. The weighted loss gave minority classes more importance during training.

Class weights used in the final run:

| Label                 | Weight |
| --------------------- | -----: |
| `actionable_guidance` |  0.565 |
| `unsupported_claim`   |  1.522 |
| `generic_noise`       |  1.029 |
| `experience_report`   |  1.667 |

---

## Evaluation Results

### Overall Accuracy

| Model                          | Accuracy |
| ------------------------------ | -------: |
| Zero-shot Groq baseline        |    0.733 |
| Weighted fine-tuned DistilBERT |    0.667 |

The Groq baseline slightly outperformed the fine-tuned DistilBERT model on overall accuracy. However, accuracy is not the only important metric in this project because the labels are imbalanced and some label boundaries are harder than others.

The weighted fine-tuned model was still close to the baseline. The baseline got about 22 out of 30 test examples correct, while the fine-tuned model got about 20 out of 30 correct. Since the test set has only 30 examples, one prediction changes accuracy by about 0.033. This means the difference between the two models is only about two examples.

The weighted fine-tuned model was a major improvement over earlier fine-tuning attempts. Earlier runs failed to predict some classes well, but the final weighted model predicted all four labels and performed especially well on `experience_report`.

---

## Baseline Per-Class Metrics

| Label                 | Precision |   Recall | F1-score | Support |
| --------------------- | --------: | -------: | -------: | ------: |
| `actionable_guidance` |      0.82 |     0.69 |     0.75 |      13 |
| `unsupported_claim`   |      0.75 |     0.60 |     0.67 |       5 |
| `generic_noise`       |      1.00 |     0.75 |     0.86 |       8 |
| `experience_report`   |      0.44 |     1.00 |     0.62 |       4 |
| **Accuracy**          |           |          | **0.73** |  **30** |
| **Macro avg**         |  **0.75** | **0.76** | **0.72** |  **30** |
| **Weighted avg**      |  **0.81** | **0.73** | **0.75** |  **30** |

The baseline was strongest on `generic_noise`, with an F1-score of 0.86. Its weakest area was `experience_report` precision. It found all true `experience_report` examples, but it also over-predicted that label.

---

## Fine-Tuned Model Per-Class Metrics

| Label                 | Precision |   Recall | F1-score | Support |
| --------------------- | --------: | -------: | -------: | ------: |
| `actionable_guidance` |      0.82 |     0.69 |     0.75 |      13 |
| `unsupported_claim`   |      0.40 |     0.40 |     0.40 |       5 |
| `generic_noise`       |      0.62 |     0.62 |     0.62 |       8 |
| `experience_report`   |      0.67 |     1.00 |     0.80 |       4 |
| **Accuracy**          |           |          | **0.67** |  **30** |
| **Macro avg**         |  **0.63** | **0.68** | **0.64** |  **30** |
| **Weighted avg**      |  **0.68** | **0.67** | **0.66** |  **30** |

The weighted fine-tuned model performed best on `experience_report`, reaching 1.00 recall and 0.80 F1. It also matched the baseline F1 for `actionable_guidance`. The weakest class was `unsupported_claim`, where precision and recall were both 0.40.

---

## Fine-Tuned Model Confusion Matrix

Rows are true labels. Columns are predicted labels.

| True label ↓ / Predicted label → | actionable_guidance | unsupported_claim | generic_noise | experience_report |
| -------------------------------- | ------------------: | ----------------: | ------------: | ----------------: |
| `actionable_guidance`            |                   9 |                 3 |             0 |                 1 |
| `unsupported_claim`              |                   0 |                 2 |             3 |                 0 |
| `generic_noise`                  |                   2 |                 0 |             5 |                 1 |
| `experience_report`              |                   0 |                 0 |             0 |                 4 |

The fine-tuned model handled `experience_report` very well, correctly classifying all 4 test examples. The biggest remaining issue was `unsupported_claim`. Three of the five `unsupported_claim` examples were predicted as `generic_noise`.

The model also confused some `generic_noise` examples with `actionable_guidance` or `experience_report`. This suggests that the model sometimes mistakes low-information comments for meaningful advice or personal experience when they contain career-related or technical words.

---

## Error Analysis

The fine-tuned model got 10 out of 30 test examples wrong. After reviewing the wrong predictions, the main pattern is that the model sometimes over-relies on topic words, tone, or surface structure instead of judging the comment’s main function.

### Wrong Prediction 1

**Text:** “Grinding Kaggle until your eyes bleed is some of the best experience ever. It’s one thing to know a few algorithms but quite another to score big wins at validation. I am grinding Kaggle even though ...”
**True label:** `actionable_guidance`
**Predicted label:** `unsupported_claim`
**Confidence:** 0.65

**Analysis:** This comment was labeled `actionable_guidance` because it recommends Kaggle practice as a way to gain experience. The model likely predicted `unsupported_claim` because the wording is exaggerated and broad, especially “some of the best experience ever.” This shows that the model sometimes confuses strongly worded advice with unsupported claims.

### Wrong Prediction 2

**Text:** “Nothing. There won't be any DS jobs other than for phds”
**True label:** `unsupported_claim`
**Predicted label:** `generic_noise`
**Confidence:** 0.80

**Analysis:** This comment is an unsupported claim because it makes a broad statement about the entire data science job market without evidence. The model predicted `generic_noise`, probably because the comment is very short and low-detail. This is a difficult boundary because short unsupported claims can look like noise. The intended rule is that a short comment can still be `unsupported_claim` if it makes a broad confident claim.

### Wrong Prediction 3

**Text:** “2023 is my year of hiring based on vibes only. Context: work as part of a super stuffy team who will ask candidates how to calculate the harmonic mean, judgemental of every candidate to the point I do...”
**True label:** `generic_noise`
**Predicted label:** `actionable_guidance`
**Confidence:** 0.94

**Analysis:** This was labeled `generic_noise` because the comment is more of a vague or sarcastic discussion than a useful recommendation. The model likely predicted `actionable_guidance` because the comment contains hiring and interview-related language. This shows that the model sometimes treats career-topic language as advice even when the comment does not provide clear, actionable guidance.

### Wrong Prediction 4

**Text:** “Don’t be an antisocial, overly technical nerd—it will limit your career trajectory! Charisma is king in all things business.”
**True label:** `actionable_guidance`
**Predicted label:** `unsupported_claim`
**Confidence:** 0.95

**Analysis:** This comment does give advice: it tells the reader to improve social and communication skills rather than focusing only on technical ability. However, the wording is broad and strongly stated, especially “Charisma is king,” so the model treated it as an unsupported claim. This is a hard boundary because some advice comments also contain broad generalizations. The model confused the tone of the comment with its main function.

### Wrong Prediction 5

**Text:** “DS job market is rough right now. You should be open to non-DS roles that gets you into a large company that offers you mobility. I started my career in an inbound call center and did cold calling for...”
**True label:** `actionable_guidance`
**Predicted label:** `experience_report`
**Confidence:** 0.99

**Analysis:** This example contains both advice and personal experience. The correct label is `actionable_guidance` because the main purpose is to advise the reader to consider adjacent roles and use internal mobility. The model predicted `experience_report` because the comment includes first-person career history like “I started my career.” This shows that the model sometimes overweights first-person language, even when the comment’s main function is still giving advice.

### Error Pattern Summary

The main remaining failure modes are:

1. Short unsupported claims can be confused with `generic_noise`.
2. Advice written in a broad or exaggerated tone can be confused with `unsupported_claim`.
3. Comments that combine advice with personal history can be confused with `experience_report`.
4. `generic_noise` comments with career-related or technical words are sometimes predicted as meaningful labels.

To improve this model, I would add more examples for boundary cases: short unsupported claims, advice-plus-personal-story comments, broad-sounding advice, and noisy career-topic comments. I would also make the label rules more explicit about judging the main function of the comment, not just the words it contains.

---

## Sample Classifications

The table below shows sample outputs from the fine-tuned model. These examples show confident mistakes, ambiguous cases, and one correct prediction.

| Example text                                                                                                                                                                         | Predicted label       |                           Confidence | Notes                                                                                                                                                              |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------- | -----------------------------------: | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| “Nothing. There won't be any DS jobs other than for phds”                                                                                                                            | `generic_noise`       |                                 0.80 | Incorrect. This should be `unsupported_claim` because it makes a broad job-market claim without evidence.                                                          |
| “2023 is my year of hiring based on vibes only...”                                                                                                                                   | `actionable_guidance` |                                 0.94 | Incorrect. The model over-focused on hiring-related language and missed that the comment was low-information/sarcastic.                                            |
| “Don’t be an antisocial, overly technical nerd—it will limit your career trajectory! Charisma is king in all things business.”                                                       | `unsupported_claim`   |                                 0.95 | Incorrect. The comment is strongly worded, but its main function is still advice.                                                                                  |
| “DS job market is rough right now. You should be open to non-DS roles that gets you into a large company that offers you mobility. I started my career in an inbound call center...” | `experience_report`   |                                 0.99 | Incorrect. The model over-weighted first-person career history even though the main function was advice.                                                           |
| “Honestly I would have just stayed in management. I left management to become a data scientist and now I’m crawling back into management lol.”                                       | `experience_report`   | Not printed in final notebook output | Correct. This prediction is reasonable because the comment mainly describes the writer’s own career experience and outcome rather than giving steps to the reader. |

---

## Reflection: Intended vs. Learned Behavior

My intended labeling system was based on the function of a comment. I wanted the model to identify whether a comment gives useful advice, reports personal experience, makes an unsupported claim, or provides low-information noise.

The model captured some of this behavior well. It learned `experience_report` strongly in the final weighted model, correctly identifying all test examples from that class. It also performed reasonably on `actionable_guidance`, matching the baseline F1-score for that label.

However, the model did not fully capture the usefulness boundary behind `generic_noise`. Some noisy comments include technical words, career terms, or claim-like phrasing. The model sometimes treated those surface cues as evidence for `actionable_guidance`, `unsupported_claim`, or `experience_report`. This suggests that the learned decision boundary is partly based on topic and wording rather than deeper usefulness.

The model also struggled when a comment mixed two functions. For example, a comment can give advice while also including first-person career history. In those cases, the model sometimes predicted `experience_report` even when the main function was guidance.

Overall, the model learned meaningful patterns, but it still needs more examples of hard boundary cases to better match my intended label definitions.

---

## Spec Reflection

One way the spec helped guide my implementation was by requiring both a zero-shot baseline and a fine-tuned model. This made the evaluation more meaningful because I could compare a general LLM against a smaller model trained on my own labels.

One way my implementation diverged from the simplest setup was that I added class-weighted training. The original fine-tuned model performed worse and failed to predict some labels well. I added weighted loss because the dataset had moderate class imbalance and the model was over-favoring larger classes. This improved the fine-tuned model from earlier runs and helped it predict all four labels.

---

## AI Usage

I used AI assistance during this project in the following ways:

1. **Label definition and edge case reasoning:** I used AI to help clarify the differences between `actionable_guidance`, `experience_report`, `unsupported_claim`, and `generic_noise`. I did not accept labels blindly; I reviewed examples and made final labeling decisions myself.

2. **Baseline prompt design:** I used AI to help write the zero-shot baseline prompt so the Groq model would return exactly one valid label. I edited the prompt to include clear rules and examples for each category.

3. **Metric interpretation and debugging:** I used AI to interpret the baseline and fine-tuned evaluation results. When the first fine-tuned model predicted only two classes, AI helped identify class imbalance as a possible issue. I then added class-weighted training and verified that it improved the final model.

4. **Error analysis and README organization:** I used AI to help organize the evaluation report and identify patterns in wrong predictions. I verified the final claims myself by reviewing the confusion matrix and the misclassified examples.

---

## Demo Video Plan

The demo video will show:

1. A short overview of TakeMeter and the four labels.
2. Three to five posts classified by the fine-tuned model with predicted labels and confidence scores.
3. One correct prediction with an explanation of why the label is reasonable.
4. One incorrect prediction with an explanation of what went wrong.
5. A walkthrough of the evaluation results, including the baseline comparison, per-class metrics, confusion matrix, and failure analysis.

---

## Files

| File                      | Purpose                                                                                            |
| ------------------------- | -------------------------------------------------------------------------------------------------- |
| `planning.md`             | Working notes, label definitions, edge case rules, data collection plan, and annotation decisions. |
| `README.md`               | Final polished project report and evaluation summary.                                              |
| `evaluation_results.json` | Saved evaluation results from the notebook.                                                        |
| `confusion_matrix.png`    | Saved confusion matrix image from the fine-tuned model.                                            |

---

## Final Takeaway

The zero-shot Groq baseline achieved slightly higher accuracy than the weighted fine-tuned DistilBERT model. However, the fine-tuned model became much more useful after adding class-weighted training. It learned all four classes, performed especially well on `experience_report`, and produced diagnosable errors that point to clear next steps: more boundary-case examples, stronger rules for `generic_noise`, more short `unsupported_claim` examples, and more examples where advice and personal experience appear together.
