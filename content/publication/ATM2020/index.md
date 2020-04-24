---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Propensity Score Analysis for Time-Dependent Exposure"
authors: [Zhongheng Zhang, Xiuyang Li, Xiao Wu, Huixian Qiu, Hongying Shi, written on behalf of AME Big-Data Clinical Trial Collaborative Group]
date: 2020-04-22T17:00:07+08:00
doi: "10.21037/atm.2020.01.33"

# Schedule page publish date (NOT publication's date).
publishDate: 2020-04-22T17:00:07+08:00

# Publication type.
# Legend: 0 = Uncategorized; 1 = Conference paper; 2 = Journal article;
# 3 = Preprint / Working Paper; 4 = Report; 5 = Book; 6 = Book section;
# 7 = Thesis; 8 = Patent
publication_types: ["2"]

# Publication name and optional abbreviated publication name.
publication: "Annals of Translational Medicine"
publication_short: "ATM"

abstract: "Propensity score analysis (PSA) is widely used in medical literature to account for confounders. Conventionally, the propensity score (PS) is calculated by a binary logistic regression model using time-fixed covariates. In the presence of time-varying treatment or exposure, the conventional method may cause bias because subjects with early and late exposure are treated as the same. In effect, subjects who are treated latter can be different from those who are treated early. Thus, the conventional PSA must be modified to address this bias. In this paper, we illustrate how to perform analysis in the presence of time-dependent exposure. We conduct a simulation study with a known treatment effect. In the simulation study, we find the PSA method that directly adjust PS estimated by either a binary logistic regression model or a Cox regression model using time-fixed covariates still introduce significant bias. On the other hand, the time-dependent PS matching can help to achieve a result approaching the true effect. After time-dependent PS matching, the matched cohort can be analyzed with conventional Cox regression model or conditional logistic regression (CLR) model with time strata. The performance is comparable to the correctly specified Cox regression model with time-varying covariates (i.e., adjusting the exposure in a multivariable model as a time-varying covariate). We further develop a function called TDPSM() for time-dependent PS matching and it is applied to a real world dataset."

# Summary. An optional shortened abstract.
summary: ""

tags: [big data methodology]
categories: [methodology]
featured: false

# Custom links (optional).
#   Uncomment and edit lines below to show custom links.
# links:
# - name: Follow
#   url: https://twitter.com
#   icon_pack: fab
#   icon: twitter

url_pdf: http://atm.amegroups.com/article/view/36411/pdf
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
  caption: "ATM"
  focal_point: "Bottom"
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
