# TakeMeter Planning

## Project Overview

For this project, I am building TakeMeter, a text classifier for career advice discourse in the `r/datasciencecareers` Reddit community. The classifier will read a data science career-related post or comment and classify it into one of four categories based on the type and quality of the advice or claim.

The goal is not to judge whether a comment is objectively true or false. The goal is to classify how the comment functions in the community: whether it gives actionable advice, shares personal experience, makes an unsupported claim, or adds low-detail noise.

---

## 1. Community

I chose `r/datasciencecareers` because it is focused on data science career discussions. People in this community discuss job searching, resumes, projects, degrees, interviews, networking, AI/ML skills, entry-level roles, and the data science job market.

This community is a good fit for a classification task because the discourse is active, text-heavy, and varied in quality. Some comments give clear steps that a job seeker can follow. Some comments describe the writer’s personal career path. Some comments make broad claims about AI, hiring, or the job market without much support. Other comments are vague replies like “network more,” “just keep applying,” or “do projects.”

These distinctions matter because people reading this community are often making real decisions about their careers. A useful classifier could help separate practical advice from personal anecdotes, unsupported market claims, and low-effort comments.

---

## 2. Labels

I will use four labels:

1. `actionable_guidance`
2. `experience_report`
3. `unsupported_claim`
4. `generic_noise`

Each comment must be assigned exactly one label.

---

### Label 1: `actionable_guidance`

Definition: A comment is `actionable_guidance` when it gives specific advice, steps, strategies, or recommendations that a reader could realistically act on.

Clear example 1:

"1300 apps with under 10 interviews usually means the problem is before the interview stage, not necessarily your degree. Data analytics is crowded, and little experience hurts, but sometimes it’s resume positioning, ATS fit, or applying too broadly. I’d focus hard on showing projects like real business outcomes, not just tools used. At this point, better targeting probably helps more than another 500 applications."

Clear example 2:

"If you have zero experience, then you need to work on some solo projects and do some freelance work that you could then list on your resume."

Uncertain example:

"Network. Job boards are just putting your name on a raffle ticket. I got my current job by cold emailing a senior partner out of nowhere."

Why this is uncertain:

This comment includes personal experience and also makes a broad claim about job boards. However, the main purpose is to recommend a concrete strategy: networking and cold outreach. I would label it `actionable_guidance`.

Decision rule:

If the comment mainly tells the reader what to do next, label it `actionable_guidance`, even if it also includes personal experience or broad claims.

---

### Label 2: `experience_report`

Definition: A comment is `experience_report` when it mainly describes the writer’s own background, job search, career path, interviews, or outcome.

Clear example 1:

"I received 5 offers after intense interviews in April and early May. I relied entirely on LinkedIn and recruiting contacts I built over years."

Clear example 2:

"I went to grad school for data science. I came in with a physics degree."

Uncertain example:

"I got my current job by cold emailing a senior partner out of nowhere. I would not have this job right now if I had not networked and shot my shot."

Why this is uncertain:

This comment is personal, but it also gives advice indirectly. I would label it `experience_report` only if the main focus is the writer describing what happened to them. If the writer is mainly using the story to push a recommendation, I would label it `actionable_guidance`.

Decision rule:

If the comment mainly says “this is what happened to me,” label it `experience_report`. If it mainly says “this is what you should do,” label it `actionable_guidance`.

---

### Label 3: `unsupported_claim`

Definition: A comment is `unsupported_claim` when it makes a broad, confident, or predictive claim without enough evidence or reasoning inside the comment.

Clear example 1:

"Anything dealing with analytics is being actively replaced with AI."

Clear example 2:

"Anyone who claims to know anything about ML has no idea what they are talking about."

Uncertain example:

"I’ll advise new grads to either do software engineering or train themselves for jobs that involve building or fine-tuning foundation models. Most companies require a PhD."

Why this is uncertain:

This comment gives career advice, but it also makes a broad claim about what most companies require. I would label it `actionable_guidance` if the main purpose is to recommend a specific career path. I would label it `unsupported_claim` if the main point is the broad claim and the comment does not give enough reasoning or next steps.

Decision rule:

A comment is not `unsupported_claim` just because I disagree with it. It is `unsupported_claim` when the main point is a broad assertion about careers, hiring, degrees, AI, or the job market, and the comment does not support that assertion with enough reasoning, examples, or evidence.

---

### Label 4: `generic_noise`

Definition: A comment is `generic_noise` when it is too vague, short, motivational, or low-detail to help the reader make a concrete decision.

Clear example 1:

"gotta network."

Clear example 2:

"ML is about inquiry, not mastery."

Uncertain example:

"Yeah, I am in the same boat. I am hoping for campus placement drives. DM me if you want to prepare together."

Why this is uncertain:

This comment has a small personal element, but it does not explain a path, result, lesson, or concrete strategy. It is not detailed enough to be `experience_report`, and it does not give enough specific advice to be `actionable_guidance`. I would label it `generic_noise`.

Decision rule:

Short comments are not automatically noise. If a short comment gives a specific action, it can be `actionable_guidance`. But if it only gives vague encouragement, agreement, a slogan, or a low-detail reply, label it `generic_noise`.

---

## 3. Hard Edge Cases

The hardest edge cases will be comments that combine personal experience with advice. For example, a person may describe how they got a job through networking and then tell others to network. That could fit both `experience_report` and `actionable_guidance`.

I will handle this by labeling based on the main function of the comment. If the comment is mainly a story about the writer’s own path, I will label it `experience_report`. If the comment mainly tells the reader what to do, I will label it `actionable_guidance`.

Another hard edge case is between `actionable_guidance` and `unsupported_claim`. Some comments give advice but support it using broad claims about the job market or AI. If the main purpose is a clear next step, I will label it `actionable_guidance`. If the main purpose is a broad claim without support, I will label it `unsupported_claim`.

A final hard edge case is `generic_noise`. Some short comments may make sense only when read with the original Reddit post. Since the model will mainly use the text field, I will label based on whether the comment itself contains enough information to be useful. If a short comment gives no clear action, explanation, or lesson, I will label it `generic_noise`.

---

## 4. Mutual Exclusivity Check

The four labels are designed so that each comment should belong to exactly one category most of the time. The main boundary is based on the comment’s primary function:

* If it tells the reader what to do, label it `actionable_guidance`.
* If it mainly tells the writer’s own story, label it `experience_report`.
* If it mainly makes a broad claim without support, label it `unsupported_claim`.
* If it is too vague or low-detail to be useful, label it `generic_noise`.

Some comments combine multiple functions, especially personal experience plus advice. In those cases, I will label based on the main purpose of the comment, not every element inside it.

---

## 5. Data Collection Plan

I will collect examples from public posts and comments in `r/datasciencecareers`. I will focus on threads about:

* Entry-level data science jobs
* Resume advice
* Data science projects and portfolios
* Networking
* Internships
* Degrees and certifications
* AI/ML career preparation
* Data science job market discussions
* Career switching into data science
* Whether data science is oversaturated

The dataset will contain at least 200 examples. I will aim for a reasonably balanced dataset, but I do not expect the four labels to appear perfectly equally in the community.

My target distribution is:

* `actionable_guidance`: around 60–75 examples
* `experience_report`: around 40–55 examples
* `unsupported_claim`: around 40–55 examples
* `generic_noise`: around 30–45 examples

I will try to keep every label at a minimum of 35–40 examples so the model has enough examples to learn each category. I expect `actionable_guidance` to be the most common label because many career threads are directly asking for advice.

I will store the dataset in a CSV file with the following columns:

* `id`
* `text`
* `label`
* `source_url`

The final CSV will be saved as:

`data/labeled_dataset.csv`

If one label is underrepresented after my first pass, I will not force incorrect labels. Instead, I will continue collecting from targeted threads where that type of comment is more likely. For example, if I need more `experience_report` examples, I will look for comments containing phrases like “I got my first job,” “I applied,” “my experience,” or “I switched from.” If I need more `unsupported_claim` examples, I will look at job market and AI replacement discussions. If I need more `generic_noise`, I will look at shorter replies lower in career advice threads.

In the final README, I will report the actual label distribution rather than pretending the dataset is perfectly balanced.

---

## 6. Evaluation Metrics

I will use accuracy, macro F1-score, per-class precision and recall, and a confusion matrix.

Accuracy is useful because it gives a simple overall measure of how often the classifier is correct. However, accuracy alone is not enough because the dataset may be slightly imbalanced. A model could get decent accuracy by overpredicting the most common label, especially `actionable_guidance`.

Macro F1-score is important because it treats each label equally. This matters because I care about performance across all four labels, not just the largest label. Per-class precision and recall will help show which labels the model handles well and which labels it confuses. For example, I expect the model may confuse `actionable_guidance` and `experience_report` because many comments contain both advice and personal stories.

The confusion matrix will help me inspect the model’s mistakes. It will show whether the model is systematically confusing certain categories, such as labeling unsupported market claims as actionable advice.

I will compare the fine-tuned model against a zero-shot Groq baseline using the same test set. This comparison matters because fine-tuning is only useful if it performs better than a strong general-purpose model with no task-specific training.

---

## 7. Definition of Success

For this project, a successful classifier should do more than memorize obvious examples. It should correctly classify new comments from the community into the four labels most of the time and should avoid collapsing everything into `actionable_guidance`.

For the class project, I would consider the model successful if the fine-tuned model reaches at least 70% overall accuracy, has a macro F1-score of at least 0.65, and performs better than the Groq zero-shot baseline on the same test set. I would also want no label to have extremely poor recall, because that would mean the model is failing to recognize one type of discourse.

For a real community tool, I would want stronger performance before deployment: around 80% accuracy, macro F1 above 0.75, and clear handling of low-confidence predictions. In a real setting, I would not want the model to automatically judge users. I would use it as a support tool to help moderators, researchers, or readers understand patterns in career advice quality.

---

## 8. AI Tool Plan

This project is not mainly a code-generation project. The most useful role for AI tools is to help test the label design, support annotation consistency, and analyze model failures after evaluation.

### Label Stress-Testing

Before labeling the full dataset, I will use an AI tool to stress-test my label definitions. I will provide the tool with my four labels, definitions, and edge case rules, then ask it to generate 5–10 realistic career advice comments that sit near the boundary between two labels.

I will especially test these boundaries:

* `actionable_guidance` vs. `experience_report`
* `actionable_guidance` vs. `unsupported_claim`
* `experience_report` vs. `generic_noise`
* `unsupported_claim` vs. `generic_noise`

If I cannot classify the generated examples cleanly using my decision rules, I will revise the label definitions before continuing annotation. The goal is not to let the AI define my labels, but to expose weak boundaries before I label 200 examples.

### Annotation Assistance

I may use an LLM to pre-label small batches of examples, but I will manually review every label myself before adding it to the final dataset. If I use AI pre-labeling, I will track that in a separate column during annotation, such as `ai_suggested_label`, while keeping my final human-reviewed label in the `label` column.

The final dataset label will always be my reviewed decision, not the AI’s automatic label. This matters because the label taxonomy depends on my project-specific definitions and edge case rules.

If time is limited, I may skip AI pre-labeling and label the dataset manually. In either case, I will disclose in the README whether AI was used during annotation.

### Failure Analysis

After training and evaluation, I will collect examples where the fine-tuned model predicted the wrong label. I will give those wrong predictions to an AI tool and ask it to identify possible error patterns.

I will look for patterns such as:

* The model confusing `actionable_guidance` with `experience_report`
* The model treating long comments as `actionable_guidance` even when they are mostly unsupported claims
* The model misclassifying short but useful comments as `generic_noise`
* The model failing to recognize unsupported market or AI replacement claims
* The model relying on surface cues like length, confidence, or first-person language

I will not accept the AI’s failure analysis automatically. I will manually check the suggested patterns against the actual wrong predictions and only include patterns in my evaluation report if they appear in multiple examples.

### Objective Success Criteria Review

My success criteria are specific enough to evaluate at the end of the project because they use measurable thresholds.

I will consider the fine-tuned model successful for the class project if it meets all of the following:

* Overall accuracy is at least 70%
* Macro F1-score is at least 0.65
* The fine-tuned model performs better than the Groq zero-shot baseline on the same test set
* No label has extremely poor recall
* The model does not collapse into predicting mostly `actionable_guidance`

I will consider the model partially successful if it beats the Groq baseline but performs poorly on one or two labels. I will consider it unsuccessful if it does not beat the baseline, has very low macro F1, or mainly predicts the most common label.

---

## 9. Planned Files

The GitHub repository will contain the files that live outside the Colab notebook:

* `planning.md`
* `README.md`
* `data/labeled_dataset.csv`
* `outputs/evaluation_results.json`
* `outputs/confusion_matrix.png`

The main model training and evaluation will be done in Google Colab. The output files will be downloaded from Colab and added to the repository after evaluation.
