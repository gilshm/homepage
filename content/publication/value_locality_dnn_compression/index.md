---
title: "Non-Blocking Simultaneous Multithreading: Embracing the Resiliency of Deep Neural Networks"

# Authors
# If you created a profile for a user (e.g. the default `admin` user), write the username (folder name) here 
# and it will be replaced with their full name and linked to their profile.
authors:
- Freddy Gabbay
- admin

# Author notes (optional)
# author_notes:
# - "Equal contribution"
# - "Equal contribution"

date: "2021-10-16T00:00:00Z"
doi: ""

# Schedule page publish date (NOT publication's date).
publishDate: "2020-10-17T00:00:00Z"

# Publication type.
# Legend: 0 = Uncategorized; 1 = Conference paper; 2 = Journal article;
# 3 = Preprint / Working Paper; 4 = Report; 5 = Book; 6 = Book section;
# 7 = Thesis; 8 = Patent
publication_types: ["2"]

# Publication name and optional abbreviated publication name.
publication: In *Multidisciplinary Digital Publishing Institute (MDPI)*
publication_short: In *MDPI*

abstract: "Convolutional Neural Networks (CNNs) are broadly used in numerous applications such as computer vision and image classification. Although CNN models deliver state-of-the-art accuracy, they require heavy computational resources that are not always affordable or available on every platform. Limited performance, system cost, and energy consumption, such as in edge devices, argue for the optimization of computations in neural networks. Toward this end, we propose herein the value-locality-based compression (VELCRO) algorithm for neural networks. VELCRO is a method to compress general-purpose neural networks that are deployed for a small subset of focused specialized tasks. Although this study focuses on CNNs, VELCRO can be used to compress any deep neural network. VELCRO relies on the property of value locality, which suggests that activation functions exhibit values in proximity through the inference process when the network is used for specialized tasks. VELCRO consists of two stages: a preprocessing stage that identifies output elements of the activation function with a high degree of value locality, and a compression stage that replaces these elements with their corresponding average arithmetic values. As a result, VELCRO not only saves the computation of the replaced activations but also avoids processing their corresponding output feature map elements. Unlike common neural network compression algorithms, which require computationally intensive training processes, VELCRO introduces significantly fewer computational requirements. An analysis of our experiments indicates that, when CNNs are used for specialized tasks, they introduce a high degree of value locality relative to the general-purpose case. In addition, the experimental results show that without any training process, VELCRO produces a compression-saving ratio in the range 13.5–30.0% with no degradation in accuracy. Finally, the experimental results indicate that, when VELCRO is used with a relatively low compression target, it significantly improves the accuracy by 2–20% for specialized CNN tasks."

# Summary. An optional shortened abstract.
# summary: Lorem ipsum dolor sit amet, consectetur adipiscing elit. Duis posuere tellus ac convallis placerat. Proin tincidunt magna sed ex sollicitudin condimentum.

tags: []

# Display this page in the Featured widget?
featured: false

# Custom links (uncomment lines below)
# links:
# - name: Custom Link
#   url: http://example.org

url_pdf: 'https://www.mdpi.com/2227-7390/9/20/2612/pdf'
url_code: ''
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

