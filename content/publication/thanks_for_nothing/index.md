---
title: "Thanks for Nothing: Predicting Zero-Valued Activations with Lightweight Convolutional Neural Networks"

# Authors
# If you created a profile for a user (e.g. the default `admin` user), write the username (folder name) here 
# and it will be replaced with their full name and linked to their profile.
authors:
- admin
- Ron Banner
- Moran Shkolnik
- Uri Weiser

# Author notes (optional)
# author_notes:
# - "Equal contribution"
# - "Equal contribution"

date: "2020-08-23T00:00:00Z"
doi: ""

# Schedule page publish date (NOT publication's date).
publishDate: "2020-08-23T00:00:00Z"

# Publication type.
# Legend: 0 = Uncategorized; 1 = Conference paper; 2 = Journal article;
# 3 = Preprint / Working Paper; 4 = Report; 5 = Book; 6 = Book section;
# 7 = Thesis; 8 = Patent
publication_types: ["1"]

# Publication name and optional abbreviated publication name.
publication: In *European Conference on Computer Vision (ECCV)*
publication_short: In *ECCV*

abstract: Convolutional neural networks (CNNs) introduce state-of-the-art results for various tasks with the price of high computational demands. Inspired by the observation that spatial correlation exists in CNN output feature maps (ofms), we propose a method to dynamically predict whether ofm activations are zero-valued or not according to their neighboring activation values, thereby avoiding zero-valued activations and reducing the number of convolution operations. We implement the zero activation predictor (ZAP) with a lightweight CNN, which imposes negligible overheads and is easy to deploy on existing models. ZAPs are trained by mimicking hidden layer ouputs; thereby, enabling a parallel and label-free training. Furthermore, without retraining, each ZAP can be tuned to a different operating point trading accuracy for MAC reduction.

# Summary. An optional shortened abstract.
# summary: Lorem ipsum dolor sit amet, consectetur adipiscing elit. Duis posuere tellus ac convallis placerat. Proin tincidunt magna sed ex sollicitudin condimentum.

tags: []

# Display this page in the Featured widget?
featured: false

# Custom links (uncomment lines below)
# links:
# - name: Custom Link
#   url: http://example.org

url_pdf: 'https://arxiv.org/pdf/1909.07636'
url_code: 'https://github.com/gilshm/zap'
url_dataset: ''
url_poster: ''
url_project: ''
url_slides: ''
url_source: ''
url_video: ''

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder. 
# image:
#   caption: 'Image credit: [**Unsplash**](https://unsplash.com/photos/pLCdAaMFLTE)'
#   focal_point: ""
#   preview_only: false

# Associated Projects (optional).
#   Associate this publication with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `internal-project` references `content/project/internal-project/index.md`.
#   Otherwise, set `projects: []`.
# projects:
# - example

# Slides (optional).
#   Associate this publication with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides: "example"` references `content/slides/example/index.md`.
#   Otherwise, set `slides: ""`.
# slides: example
---

