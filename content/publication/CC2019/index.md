---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Machine learning for the prediction of volume responsiveness in patients with oliguric acute kidney injury in critical care"
authors: [Zhongheng Zhang; Kwok M. Ho; Yucai Hong ]
date: 2020-04-22T17:00:07+08:00
doi: "10.1186/s13054-019-2411-z"

# Schedule page publish date (NOT publication's date).
publishDate: 2020-04-22T17:00:07+08:00

# Publication type.
# Legend: 0 = Uncategorized; 1 = Conference paper; 2 = Journal article;
# 3 = Preprint / Working Paper; 4 = Report; 5 = Book; 6 = Book section;
# 7 = Thesis; 8 = Patent
publication_types: ["2"]

# Publication name and optional abbreviated publication name.
publication: "Critical Care"
publication_short: "CC"

abstract: "Background and objectives
Excess fluid balance in acute kidney injury (AKI) may be harmful, and conversely, some patients may respond to fluid challenges. This study aimed to develop a prediction model that can be used to differentiate between volume-responsive (VR) and volume-unresponsive (VU) AKI.

Methods
AKI patients with urine output < 0.5 ml/kg/h for the first 6 h after ICU admission and fluid intake > 5 l in the following 6 h in the US-based critical care database (Medical Information Mart for Intensive Care (MIMIC-III)) were considered. Patients who received diuretics and renal replacement on day 1 were excluded. Two predictive models, using either machine learning extreme gradient boosting (XGBoost) or logistic regression, were developed to predict urine output > 0.65 ml/kg/h during 18 h succeeding the initial 6 h for assessing oliguria. Established models were assessed by using out-of-sample validation. The whole sample was split into training and testing samples by the ratio of 3:1.

Main results
Of the 6682 patients included in the analysis, 2456 (36.8%) patients were volume responsive with an increase in urine output after receiving > 5 l fluid. Urinary creatinine, blood urea nitrogen (BUN), age, and albumin were the important predictors of VR. The machine learning XGBoost model outperformed the traditional logistic regression model in differentiating between the VR and VU groups (AU-ROC, 0.860; 95% CI, 0.842 to 0.878 vs. 0.728; 95% CI 0.703 to 0.753, respectively).

Conclusions
The XGBoost model was able to differentiate between patients who would and would not respond to fluid intake in urine output better than a traditional logistic regression model. This result suggests that machine learning techniques have the potential to improve the development and validation of predictive modeling in critical care research."

# Summary. An optional shortened abstract.
summary: ""

tags: []
categories: []
featured: false

# Custom links (optional).
#   Uncomment and edit lines below to show custom links.
# links:
# - name: Follow
#   url: https://twitter.com
#   icon_pack: fab
#   icon: twitter

url_pdf: https://ccforum.biomedcentral.com/track/pdf/10.1186/s13054-019-2411-z
url_code:
url_dataset:
url_poster:
url_project:
url_slides:
url_source:
url_video:

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder. 
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: "Oliguria.png"
  focal_point: "Smart"
  preview_only: false

# Associated Projects (optional).
#   Associate this publication with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `internal-project` references `content/project/internal-project/index.md`.
#   Otherwise, set `projects: []`.
projects: []

# Slides (optional).
#   Associate this publication with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides: "example"` references `content/slides/example/index.md`.
#   Otherwise, set `slides: ""`.
slides: ""
---
