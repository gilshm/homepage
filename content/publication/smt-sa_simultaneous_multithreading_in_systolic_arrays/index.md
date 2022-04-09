---
title: "SMT-SA: Simultaneous Multithreading in Systolic Arrays"

# Authors
# If you created a profile for a user (e.g. the default `admin` user), write the username (folder name) here 
# and it will be replaced with their full name and linked to their profile.
authors:
- admin
- Tal Horowitz
- Uri Weiser

# Author notes (optional)
# author_notes:
# - "Equal contribution"
# - "Equal contribution"

date: "2019-06-20T00:00:00Z"
doi: ""

# Schedule page publish date (NOT publication's date).
publishDate: "2019-06-20T00:00:00Z"

# Publication type.
# Legend: 0 = Uncategorized; 1 = Conference paper; 2 = Journal article;
# 3 = Preprint / Working Paper; 4 = Report; 5 = Book; 6 = Book section;
# 7 = Thesis; 8 = Patent
publication_types: ["2"]

# Publication name and optional abbreviated publication name.
publication: In *Computer Architecture Letters (CAL)*
publication_short: In *CAL*

abstract: Systolic arrays (SAs) are highly parallel pipelined structures capable of executing various tasks such as matrix multiplication and convolution. They comprise a grid of usually homogeneous processing units (PUs) that are responsible for the multiply-accumulate (MAC) operations in the case of matrix multiplication. It is not rare for a PU input to be zero-valued, in which case the PU becomes idle and the array becomes underutilized. In this paper we consider a solution to employ the underutilized PUs via simultaneous multithreading (SMT). We explore the design space of a SMT-SA variant and evaluate its performance, area efficiency, and energy consumption. In addition, we suggest a tiling method to reduce area overheads. Our evaluation shows that a 4-thread FP16-based SMT-SA achieves speedups of up to 3.6x as compared to conventional SA, with 1.7x area overhead and negligible energy overhead.

# Summary. An optional shortened abstract.
# summary: Lorem ipsum dolor sit amet, consectetur adipiscing elit. Duis posuere tellus ac convallis placerat. Proin tincidunt magna sed ex sollicitudin condimentum.

tags: []

# Display this page in the Featured widget?
featured: false

# Custom links (uncomment lines below)
# links:
# - name: Custom Link
#   url: http://example.org

url_pdf: 'https://arxiv.org/pdf/2105.11010.pdf'
url_code: 'https://github.com/gilshm/sparq'
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

