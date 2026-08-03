## Predicting Declining Content Pages for Refresh Prioritization

## Author: Leyla Aghayeva Course: FlyRank ML Internship Lane: Refresh / Content Opportunity Scoring Date: August 2026

## Abstract

Content teams managing large websites often face the challenge of deciding which pages should be reviewed and refreshed first. This project investigates whether machine learning can identify content pages that exhibit patterns associated with declining search performance and rank them for human review. Using the FlyRank ML Internship dataset, which contains production-scale search performance data, I developed a decision-support system that combines a transparent rule-based baseline with a Logistic Regression model trained on content freshness, visibility, ranking, and search demand features. The model outperformed the baseline on the same evaluation split, improving Precision@20 from 0.50 to 0.75 and Precision@50 from 0.46 to 0.58, demonstrating that machine learning can better prioritize potentially declining pages for review. The resulting ranked recommendation system is intended to support editorial decision-making rather than automate content refreshes, and all findings are presented as observed associations rather than causal relationships.

## 1. Introduction

## 1.1 Background

Search performance naturally changes over time as user behavior, competition, and search engine rankings evolve. As websites grow, manually monitoring every page becomes increasingly difficult. Many organizations therefore need a systematic way to identify which content pages deserve attention before significant traffic losses occur.


Traditional approaches often rely on simple rules, such as reviewing pages that have not been updated recently or pages that continue receiving search impressions despite declining performance. Although these heuristics are useful, they cannot

always capture the complex interactions between multiple search signals.

Machine learning provides an opportunity to analyze multiple content characteristics simultaneously and estimate which pages are more likely to belong to a declining group. Rather than replacing human judgment, predictive models can prioritize pages

that deserve review, allowing editorial teams to focus their efforts more efficiently.

## 1.2 Research Question

This project investigates the following research question:

Can machine learning identify content pages that exhibit signals associated with declining search performance and improve the prioritization of pages requiring review compared with a simple rule-based baseline?

Rather than predicting the exact future performance of a page, the objective is to estimate the likelihood that a page belongs to the declining class based on historical search performance and content-related features.

## 1.3 Problem Statement

Organizations with thousands of indexed pages cannot manually evaluate every page on a regular basis. As a result, declining content may remain unnoticed until search visibility has already been lost.

The objective of this project is to build a ranking system that supports content review decisions by identifying pages that appear most likely to require attention.

The system follows a straightforward workflow:

- Input: Historical search performance and content-related signals.

- Processing: A machine learning model estimates the probability that a page belongs to the declining class.

- Output: A ranked list of pages ordered from highest to lowest review priority.

- Decision: Human reviewers determine whether a content refresh or another action is appropriate.


This approach positions the model as a decision-support tool rather than an automated content management system.

## 1.4 Project Objectives

The primary objectives of this project are:

- Develop a transparent baseline method for identifying pages that may require review.

- Train and evaluate a machine learning model capable of ranking pages by their likelihood of decline.

- Compare the machine learning model against the baseline using the same evaluation methodology.

- Produce an interpretable ranked recommendation system that supports editorial decision-making.

- Present all findings using cautious language that reflects observed patterns rather than causal conclusions.

## 1.5 Scope of the Study

This research focuses exclusively on identifying patterns associated with declining search performance using publicly shareable features from the FlyRank ML Internship

dataset.

The project does not attempt to:

- explain search engine ranking algorithms,

- predict future Google algorithm updates,

- demonstrate that refreshing content directly improves rankings,

- automate editorial decisions without human review.

Instead, the model provides evidence-based prioritization that can assist content teams when deciding which pages to investigate first.

## 2. Data

## 2.1 Dataset


This project uses the FlyRank ML Internship dataset, which contains anonymized production-scale search performance data collected from multiple websites. The dataset was provided as part of the internship and includes content-level performance metrics, search visibility information, ranking signals, freshness indicators, and query-level statistics.

The objective of the dataset is to support research into search intelligence while preserving privacy. All client identifiers, content identifiers, and search-related information are anonymized, and no client names, domains, URLs, or private search queries are included in this research.

## Data Credit

Built on the FlyRank ML Internship dataset.

[https://flyrank.ai](https://flyrank.ai)

- 2.2 Data Sources

The project was completed using two complementary data sources provided during the internship.

The first was the prepared internship dataset used throughout the weekly assignments. This dataset contained engineered content-level features suitable for building baseline models and evaluating different machine learning approaches.

The second source was the complete FlyRank warehouse hosted on Hugging Face. The warehouse was queried directly using DuckDB, allowing feature engineering to be performed within SQL before exporting smaller aggregated datasets for machine learning.

Working directly with the warehouse provided experience handling large-scale search

performance data while avoiding loading millions of records into memory.

- 2.3 Warehouse Exploration

The FlyRank warehouse contains several relational tables describing clients, content, daily search performance, and query-level information.

The warehouse consisted of the following tables:


Table

Rows

dim_clients

104

dim_content

519,606

fact_content_daily_performance

78,835,65 5

fact_content_daily_performance_sa 11,694,07

mple

2

fact_content_query_90d

2,414,248

DuckDB was used to query the warehouse directly through the Hugging Face dataset release. Instead of downloading the complete dataset into memory, SQL aggregations were executed inside DuckDB and only the resulting feature tables were imported into Python for further analysis.

This approach significantly reduced memory usage while allowing efficient processing

of tens of millions of records.

- 2.4 Client History Audit

Before creating features, the available historical data for each client was examined.

The warehouse represents an unbalanced panel, meaning that different clients have different lengths of available search history.

An exploratory audit showed that only 4 clients contained at least 12 months of Google Search Console history.


This observation influenced the modeling strategy because clients with shorter historical periods may provide less stable performance estimates. It also highlights why grouped or time-aware validation is generally preferable to purely random

train-test splits when evaluating predictive models on search data.

## 2.5 Feature Engineering

Feature engineering was performed both on the prepared internship dataset and directly from the FlyRank warehouse.

Within the warehouse, DuckDB SQL was used to aggregate daily search performance into content-level features over a rolling 60-day period.

For each content page, the following metrics were calculated:

- impressions during the previous 30 days (imp_prev30)

- impressions during the most recent 30 days (imp_last30)

- clicks during the most recent 30 days (clk_last30)

- average search position during the most recent 30 days (pos_last30)

To reduce noise from pages with very limited search activity, only pages with at least 100 impressions during the previous 30-day period were retained.

This filtering produced a modeling dataset containing 111,247 content pages.

For the final machine learning model developed throughout the internship, additional

content-level features from the prepared dataset were used, including:

- days_since_last_update

- content_age_days

- word_count

- search_volume

- impressions_90d

- ctr

- avg_position

- position_tier

These features were selected because they represent information that would be available before making a prediction and therefore can realistically support future decision-making.


## 2.6 Query-Level Signals

The warehouse also provides information describing how individual content pages receive search traffic through the fact_content_query_90d table.

Several query-level characteristics were incorporated into the exploratory analysis:

- visible query count

- rare impression share

- anonymized impression share

- top query impressions

- total retained impressions

From these values, an additional feature called Top Query Share was created.

This feature measures how dependent a page is on its strongest search query. Pages whose impressions are concentrated around a single query may be more vulnerable to

ranking changes than pages receiving traffic from a broader set of search queries.

Although some pages contained missing query statistics after joining the tables, the final aggregated dataset still contained 111,247 content pages, providing sufficient

coverage for exploratory modeling.

## 2.7 Label Definition

The internship provided labels identifying declining content for the prepared modeling dataset used throughout the earlier assignments.

During the warehouse exploration, an additional experimental label was constructed for comparison.

A page was considered declining if impressions during the most recent 30-day period were less than 80% of impressions during the previous 30-day period.

This month-over-month definition allowed an exploratory benchmark model to be trained directly from warehouse-derived features while avoiding direct leakage from target variables.

For the final capstone model, the prepared internship labels were retained to ensure

consistency with the evaluation performed throughout the previous assignments.


## 2.8 Data Exclusions

Several variables were intentionally excluded from model training to prevent information leakage and preserve realistic evaluation.

The following fields were not used as predictive features:

- trend_direction

- trend_pct

These variables directly describe the target outcome and therefore would allow the model to indirectly observe the answer during training.

Similarly, client identifiers and content identifiers were excluded from the feature set because they do not represent meaningful search characteristics and could encourage

memorization instead of learning generalizable relationships.

## 2.9 Public Safety

All published results comply with the internship's public sharing requirements.

Specifically:

- no client names are disclosed;

- no website domains are included;

- no private URLs are published;

- no search queries are revealed;

- no credentials or confidential information are included;

- all identifiers remain anonymized.

Consequently, the published research demonstrates machine learning methodology and observed search performance patterns without exposing proprietary client information.

## 3. Methodology

## 3.1 Research Approach


The objective of this project was to develop a decision-support system capable of prioritizing content pages that may require review due to declining search performance. Rather than attempting to predict exact future search traffic, the project focused on ranking content pages according to their likelihood of belonging to the declining class.

The methodology followed a supervised machine learning workflow consisting of four

stages:

- 1. Data preparation and feature engineering.

- 2. Development of a transparent rule-based baseline.

- 3. Training and evaluation of a Logistic Regression classification model.

- 4. Ranking pages according to predicted probability and generating actionable recommendations.

The machine learning model was evaluated against a simple baseline to determine whether predictive modeling provided meaningful improvements for prioritizing review candidates.

## 3.2 Baseline System

Before building a machine learning model, a transparent rule-based baseline was

created.

The purpose of the baseline was to establish a realistic benchmark that could be easily understood by human reviewers.

The baseline prioritized pages using three intuitive signals:

- pages that had not been updated recently;

- pages that continued receiving meaningful search impressions;

- pages with sufficient search demand to justify review.

To improve interpretability, each recommendation was accompanied by one or more reason codes explaining why the page appeared in the review queue.

The baseline reason codes included:

- STALE_HIGH_DEMAND – the page had not been updated recently while still attracting substantial search demand.

- STALE_VISIBLE – the page remained visible in search results despite long periods without updates.


- HIGH_PRIORITY_REFRESH – multiple indicators suggested that the page should be reviewed with high priority.

Unlike the machine learning model, the baseline relied entirely on manually designed decision rules without learning relationships from historical data.

The baseline served as the reference system for all subsequent evaluations.

## 3.3 Machine Learning Model

The prediction problem was formulated as a binary classification task, where each content page was classified as either belonging or not belonging to the declining class.

Several algorithms were considered during model development. Logistic Regression was selected because it provides several advantages for this application.

First, it produces probability estimates rather than only class labels. These probabilities allow pages to be ranked from highest to lowest predicted likelihood of decline, making the model particularly suitable for prioritization tasks.

Second, Logistic Regression is highly interpretable. Unlike many complex machine learning algorithms, the learned coefficients indicate how each feature influences the prediction, making it easier to explain recommendations to content reviewers.

Finally, Logistic Regression performs well on structured tabular datasets while maintaining relatively low computational complexity.

For these reasons, Logistic Regression was chosen as the final model for the capstone

project.

## 3.4 Feature Selection

The predictive model was trained using content-level features describing freshness, search visibility, ranking position, and content characteristics.

The final feature set included:

- days_since_last_update

- content_age_days

- word_count

- impressions_90d


- search_volume

- ctr

- avg_position

- position_tier

These variables were selected because they represent information available before prediction time and therefore can realistically support future content review decisions.

In addition to the prepared internship dataset, exploratory warehouse analysis introduced several additional features derived from DuckDB aggregations, including:

- previous 30-day impressions;

- recent 30-day impressions;

- average ranking position;

- visible query count;

- rare impression share;

- anonymized impression share;

- top query share.

Although these warehouse-derived features were primarily used for exploratory modeling, they demonstrated how large-scale search data could be transformed into meaningful predictive variables.

## 3.5 Target Definition

The internship dataset provided labels identifying pages belonging to the declining class.

These labels served as the target variable for the final Logistic Regression model and ensured consistency across the weekly assignments.

As part of the warehouse exploration, an additional experimental target was created for benchmarking purposes.

A page was labeled as declining when impressions during the most recent thirty-day period were less than eighty percent of impressions during the previous thirty-day period.

This exploratory definition allowed experimentation directly on warehouse data while

avoiding information leakage from future observations.


The final capstone evaluation, however, reports results using the internship-provided labels to remain consistent with the baseline system.

## 3.6 Validation Design

Model evaluation should reflect how the system would behave when applied to previously unseen content.

To reduce overly optimistic performance estimates, grouped validation was considered throughout the project.

Grouping prevents highly similar observations from appearing in both training and testing data, reducing the likelihood that the model memorizes client-specific patterns instead of learning generalizable relationships.

The same evaluation methodology was used for both the baseline system and the Logistic Regression model, ensuring that comparisons were fair.

During warehouse exploration, an initial Random Forest benchmark was evaluated using a stratified random split. Although this provided a useful exploratory benchmark, grouped validation represents a more appropriate strategy for production deployment because search behavior often varies across different clients.

Future work should therefore prioritize grouped or time-aware validation when

evaluating search intelligence models.

## 3.7 Evaluation Metric

The primary evaluation metric was Precision@K, which measures the proportion of truly declining pages among the highest-ranked recommendations.

Precision@K was selected because the objective of the project is ranking rather than simple classification.

Editorial teams typically review only a limited number of pages during each review cycle. Consequently, identifying the most valuable recommendations at the top of the ranking is more important than maximizing overall classification accuracy.

Two evaluation thresholds were used:

- Precision@20

- Precision@50


These metrics directly measure the quality of the recommendation queue presented to reviewers.

For exploratory warehouse experiments, additional classification metrics such as accuracy, precision, recall, and F1-score were also examined to better understand model behavior.

## 3.8 Leakage Prevention

Preventing information leakage was an important consideration throughout the project.

Several potential leakage sources were identified and addressed before model

training.

## Label Leakage

Variables that directly described the target outcome were excluded from the feature set.

These included:

- trend_direction

- trend_pct

Using either variable during training would allow the model to observe information that would not be available during real-world prediction.

## Identifier Leakage

Client identifiers and content identifiers were not used as predictive features.

Although these identifiers uniquely distinguish pages, they do not represent meaningful search characteristics and could encourage memorization rather than learning transferable relationships.

Identifiers were used only for grouping during validation where appropriate.

## Temporal Leakage

Only information available before prediction time was included in the feature set.


During warehouse experiments, features were generated from historical observation windows, while the prediction target was defined using subsequent performance periods.

This separation ensured that future observations were not inadvertently incorporated into predictor variables.

## Evaluation Leakage

The same evaluation procedure was applied consistently across both the baseline and the machine learning model.

This ensured that observed performance differences reflected genuine improvements rather than inconsistencies in the validation process.

## 3.9 Implementation

The project was implemented using Python and the scientific computing ecosystem introduced throughout the internship.

The primary libraries included:

- pandas

- NumPy

- scikit-learn

- DuckDB

- Matplotlib

DuckDB was used to aggregate approximately 78.8 million daily search performance records directly from the FlyRank warehouse.

Machine learning experiments were implemented using scikit-learn, while evaluation metrics and ranked recommendation outputs were generated using custom analysis notebooks.

The complete workflow was organized across the weekly internship notebooks, culminating in a final capstone notebook that integrated data preparation, baseline evaluation, model training, prediction, and recommendation generation.

## 4. Results


## 4.1 Overview

The objective of this project was to determine whether a machine learning model could improve the prioritization of content pages requiring review compared with a transparent rule-based baseline.

Two complementary evaluations were performed throughout the project.

The primary evaluation compared the final Logistic Regression model against the baseline recommendation system using the prepared internship dataset and the Precision@K metric. This represents the main contribution of the capstone.

An additional exploratory experiment was conducted directly on the FlyRank warehouse using aggregated search performance features and a Random Forest classifier. Although this experiment was not the final model, it provided valuable insight into feature engineering and demonstrated how predictive models can be constructed directly from large-scale warehouse data.

## 4.2 Baseline Performance

Before applying machine learning, a rule-based baseline was developed to identify pages that appeared suitable for review.

The baseline prioritized pages using simple and interpretable rules based on content freshness, search demand, and continued search visibility.

Evaluation was performed using Precision@K, which measures the proportion of truly declining pages among the highest-ranked recommendations.

The baseline achieved the following performance:


These results indicate that approximately half of the pages recommended by the baseline belonged to the declining class.

Although the baseline provides understandable recommendations, it considers only manually designed rules and cannot capture more complex interactions between multiple search signals.

## 4.3 Logistic Regression Performance

The final machine learning model used Logistic Regression to estimate the probability that each content page belonged to the declining class.

Unlike the baseline, Logistic Regression combines multiple content characteristics simultaneously, allowing it to recognize patterns that are difficult to express through fixed decision rules.

The model achieved the following results:

Model

0

Rule-Based Baseline

Logistic Regression 0.75

Precision@2 Precision@5

0

0.50

0.46

0.58


Compared with the baseline, the machine learning model increased Precision@20 by 25 percentage points and Precision@50 by 12 percentage points.

This improvement demonstrates that machine learning produced a more accurate ranking of potentially declining pages, particularly among the highest-priority recommendations.


Because editorial teams typically review only a limited number of pages during each review cycle, improving the quality of the highest-ranked recommendations provides practical value for content review workflows.

- 4.4 Feature Interpretation

One advantage of Logistic Regression is that its learned coefficients provide insight into which variables contribute most strongly to predictions.

Among the strongest observed feature weights were:

## Feature Coefficie nt

days_since_last_upd 0.235 ate

word_count

impressions_90d

0.099

0.040


The largest positive coefficient was associated with days_since_last_update, suggesting that pages remaining unchanged for longer periods were more frequently associated with the declining class.

Content length also contributed positively, indicating that word count carried predictive information, although its influence was considerably smaller.

Existing search visibility, represented by impressions_90d, also contributed to prediction, demonstrating that historical performance provides useful context when estimating future decline.

These coefficients should be interpreted as indicators of association rather than evidence of causation.

## 4.5 Warehouse Experiment

To complement the prepared internship dataset, an exploratory model was developed directly from the FlyRank warehouse.

Daily search performance records were aggregated into content-level features using DuckDB, producing a modeling dataset containing 111,247 content pages.

The warehouse experiment used a Random Forest classifier trained on the following variables:


- previous 30-day impressions;

- visible query count;

- rare impression share;

- anonymized impression share;

- top query share.

For exploratory purposes, the declining label was defined as a reduction of more than 20% in impressions between consecutive thirty-day periods.

The model produced the following classification results:

Metric

Value

Accuracy

65.4%

Precision (Declining)

68.6%

Recall (Declining) 83.6%

F1-score

75.3%

The majority-class baseline achieved an accuracy of 63.3%, meaning that the Random Forest model provided a modest improvement while substantially increasing recall for declining pages.

High recall indicates that the exploratory model successfully identified most declining pages, although some false positives remained.

Because the intended application is review prioritization rather than automated decision-making, identifying more potentially declining pages is generally preferable to missing important candidates.

- 4.6 Comparison of Modeling Approaches


The project explored two complementary approaches.

The first approach focused on producing the strongest recommendation system using the prepared internship dataset and Logistic Regression.

The second approach demonstrated how large-scale warehouse data could be transformed into predictive features using SQL and DuckDB before applying machine learning.

Although the warehouse experiment produced encouraging classification performance, the Logistic Regression model remained the primary capstone model

because:

- it was evaluated consistently with the internship baseline;

- it produced probability scores directly suitable for ranking;

- it offered greater interpretability through feature coefficients;

- it aligned with the recommendation-focused objectives of the project.

Together, these experiments demonstrate that machine learning can successfully prioritize pages for review while also illustrating scalable feature engineering techniques for production search data.

## 4.7 Discussion

The results suggest that combining multiple search performance signals allows machine learning to prioritize declining pages more effectively than manually designed rules alone.

The strongest improvements occurred among the highest-ranked recommendations, which is particularly valuable because editorial teams usually review only a limited number of pages during each review cycle.

The warehouse exploration further demonstrated that production-scale search datasets can be processed efficiently using DuckDB, allowing feature engineering to be performed directly on tens of millions of daily search records without requiring excessive memory.

However, predictive performance should not be interpreted as evidence that refreshing a recommended page will necessarily improve search visibility. The model identifies statistical patterns associated with previous observations and is intended solely as a decision-support tool.


## 4.8 Summary

Overall, the results indicate that the proposed machine learning approach successfully improved the prioritization of declining content pages compared with a transparent rule-based baseline.

The combination of interpretable machine learning, scalable feature engineering, and ranked recommendation outputs provides a practical framework for supporting content review decisions while maintaining transparency regarding the model's limitations and intended use.

## 5. Limitations and Honest Framing

## 5.1 Project Scope

This project was designed as a decision-support system for prioritizing content pages that may require review. The objective was not to automate editorial decisions or predict future search engine behavior with certainty. Instead, the model identifies pages that share characteristics with historically declining content and ranks them according to their predicted likelihood of belonging to the declining class.

Accordingly, all findings presented in this paper should be interpreted as observed statistical relationships rather than evidence of causality.

## 5.2 Limitations

Several limitations should be considered when interpreting the results.

## Historical Data

The model was trained using historical observations. Search behavior, user interests, competition, and search engine algorithms continually evolve over time. Consequently, patterns learned from historical data may not remain equally informative in future periods.

## Feature Availability

The model relies exclusively on features available within the internship dataset and warehouse release. Important factors that may influence search


performance—including content quality, backlink profiles, technical SEO issues, user intent, or search engine algorithm updates—were not available and therefore could not be incorporated into the prediction process.

## Validation

Although grouped validation and leakage prevention were considered throughout the project, the exploratory warehouse experiment initially used a random train-test split. Random splits may produce slightly optimistic estimates because similar observations from the same client can appear in both the training and testing sets.

Future work should evaluate models using grouped or time-aware validation across all experiments.

## Prediction versus Causation

The model predicts whether pages resemble previously observed declining examples.

It does not demonstrate:

- why a page declined;

- whether refreshing a page will improve rankings;

- how search engine algorithms operate;

- that any individual feature directly causes performance changes.

The model identifies statistical associations only.

## Human Review

Every recommendation generated by the model should be reviewed by a human editor before any action is taken.

Content updates depend on business priorities, editorial strategy, user intent, and factors that cannot be captured by structured search data alone.

## 5.3 Ethical Considerations

All analyses were performed using anonymized data provided for educational purposes.

Throughout the project:


- no client names were disclosed;

- no domains or URLs were published;

- no search queries were revealed;

- no confidential information was included;

- only aggregated, public-safe results were reported.

The published artifacts therefore comply with the internship's public sharing guidelines while demonstrating reproducible machine learning methodology.

## 6. Ranked Recommendations (Action Playbook)

The primary output of this project is a ranked recommendation system that assists

content teams in deciding which pages should be reviewed first.

Unlike automated optimization systems, these recommendations are intended to

support human decision-making.

## Priority 1 — Review High-Risk Pages

## Action

Review pages receiving the highest predicted probability of belonging to the declining class.

## Reason

These pages most closely resemble historical examples of declining content and therefore should receive the highest review priority.

## Human Decision

Editors should determine whether the page requires updating, rewriting, restructuring, or continued monitoring.

## Priority 2 — Refresh Older Content

## Action


Prioritize pages that have not been updated for long periods.

## Reason

## The model identified days_since_last_update as one of the strongest predictive signals associated with declining content. Older content should therefore be examined for outdated information, missing topics, or changing user intent.

## Priority 3 — Focus on High-Visibility Pages

## Action

## Prioritize pages that continue receiving meaningful impressions and search demand despite showing signs of decline.

## Reason

## Refreshing high-visibility pages is generally more valuable than updating pages receiving little search traffic. These pages represent opportunities where editorial effort may produce greater business value.

## Priority 4 — Monitor Query Concentration

## Action

## Review pages whose search traffic depends heavily on a small number of search queries.

## Reason

Warehouse exploration showed that pages with concentrated query distributions may be more vulnerable to changes in ranking performance.

Diversifying search visibility through broader content coverage may improve

resilience.

## Priority 5 — Continue Monitoring Stable Pages


## Action

Pages with low predicted decline probability should remain under periodic observation rather than immediate review.

## Reason

Not every page requires intervention.

Efficient resource allocation requires focusing editorial effort where the greatest potential benefit exists.

## Human Review Policy

The recommendation system should never automatically publish content updates.

Instead, the model should serve as the first stage of a review workflow in which experienced editors evaluate each recommendation before deciding whether changes are necessary.

## 7. Reproducibility

Reproducibility was an important objective throughout this project.

The complete workflow was implemented using Python notebooks developed during the FlyRank Machine Learning Internship.

The repository includes all notebooks used for:

- exploratory data analysis;

- feature engineering;

- baseline construction;

- model training;

- model evaluation;

- warehouse exploration;

- action playbook generation;

- capstone analysis.

The final capstone notebook combines the complete workflow into a reproducible machine learning pipeline.


## Software Environment

The primary software libraries used include:

- Python

- pandas

- NumPy

- scikit-learn

- DuckDB

- Matplotlib

- Hugging Face Hub

DuckDB was used to query approximately 78.8 million warehouse records efficiently without loading the complete dataset into memory.

## Repository Structure

The repository contains:

work/

notebooks/

outputs/

submission/

paper_url.txt

README.md

requirements.txt

The deployment URL for the research paper is stored in:

submission/paper_url.txt

as required by the internship submission guidelines.

## Reproducing the Results

The complete workflow can be reproduced by following these steps:


- 1. Install the required Python dependencies.

pip install -r requirements.txt

- 2. Configure the required Hugging Face access token.

- 3. Execute the notebooks in order.

- 4. Generate the engineered features.

- 5. Train the baseline system.

- 6. Train the Logistic Regression model.

- 7. Produce the ranked recommendation queue.

- 8. Export evaluation metrics and figures.

Random seed values were fixed during model training to improve reproducibility.

## 8. Conclusion

This project investigated whether machine learning could improve the prioritization of content pages requiring review compared with a transparent rule-based baseline.

A Logistic Regression model was developed using content freshness, search visibility, ranking, and search demand features extracted from the FlyRank ML Internship dataset. The resulting model consistently outperformed the rule-based baseline, improving Precision@20 from 0.50 to 0.75 and Precision@50 from 0.46 to 0.58, demonstrating that machine learning can more effectively rank pages likely to require editorial attention.

In addition to the prepared internship dataset, exploratory analysis was performed directly on the FlyRank warehouse using DuckDB. Aggregating approximately 78.8 million daily search performance records demonstrated how production-scale search

data can be transformed into efficient machine learning features while maintaining modest memory usage.

Although the predictive models produced encouraging results, the project intentionally avoids causal claims. The recommendations generated by the system should be interpreted as evidence-based prioritization rather than proof that refreshing content will improve search performance. Human judgment remains essential when determining whether editorial intervention is appropriate.

Overall, the project demonstrates that interpretable machine learning can provide practical support for content review workflows by identifying pages most likely to

deserve investigation. Combining scalable warehouse analytics, transparent feature


engineering, and explainable predictive modeling provides a reproducible framework that can assist organizations in managing large content libraries while maintaining

responsible use of machine learning.

- 9. Acknowledgments and Data Credit

This project was completed as part of the FlyRank Machine Learning Internship.

The analysis was built using the FlyRank ML Internship dataset, which provides anonymized production-scale search performance data for educational and research purposes.

Data Credit

Built on the FlyRank ML Internship dataset.

[https://flyrank.ai](https://flyrank.ai)

I would like to thank the FlyRank team for providing the internship materials, structured learning pathway, production-scale search dataset, and practical research

environment that made this capstone project possible.

## Appendix (Optional)

## Repository

GitHub Repository:

[https://github.com/LeylaAghayeva1/ml-search-engineering.git](https://github.com/LeylaAghayeva1/ml-search-engineering.git)

Submit for Review
