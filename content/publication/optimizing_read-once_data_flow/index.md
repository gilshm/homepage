---
title: "Optimizing Read-Once Data Flow in Big-Data Applications"

# Authors
# If you created a profile for a user (e.g. the default `admin` user), write the username (folder name) here 
# and it will be replaced with their full name and linked to their profile.
authors:
- Tomer Morad
- admin
- Mattan Erez
- Avinoam Kolodny
- Uri Weiser

# Author notes (optional)
# author_notes:
# - "Equal contribution"
# - "Equal contribution"

date: "2016-01-22T00:00:00Z"
doi: ""

# Schedule page publish date (NOT publication's date).
publishDate: "2016-01-22T00:00:00Z"

# Publication type.
# Legend: 0 = Uncategorized; 1 = Conference paper; 2 = Journal article;
# 3 = Preprint / Working Paper; 4 = Report; 5 = Book; 6 = Book section;
# 7 = Thesis; 8 = Patent
publication_types: ["2"]

# Publication name and optional abbreviated publication name.
publication: In *Computer Architecture Letters (CAL)*
publication_short: In *CAL*

abstract: Memory hierarchies in modern computing systems work well for workloads that exhibit temporal data locality. Data that is accessed frequently is brought closer to the computing cores, allowing faster access times, higher bandwidth, and reduced transmission energy. Many applications that work on big data, however, read data only once. When running these applications on modern computing systems, data that is not reused is nevertheless transmitted and copied into all memory hierarchy levels, leading to energy and bandwidth waste. In this paper we evaluate workloads dealing with read-once data and measure their energy consumption. We then modify the workloads so that data that is known to be used only once is transferred directly from storage into the CPU's last level cache, effectively bypassing DRAM and avoiding keeping unnecessary copies of the data. Our measurements on a real system show savings of up to 5 Watts in server power and up to 3.9% reduction in server energy when 160 GB of read-once data bypasses DRAM.

# Summary. An optional shortened abstract.
# summary: Lorem ipsum dolor sit amet, consectetur adipiscing elit. Duis posuere tellus ac convallis placerat. Proin tincidunt magna sed ex sollicitudin condimentum.

tags: []

# Display this page in the Featured widget?
featured: false

# Custom links (uncomment lines below)
# links:
# - name: Custom Link
#   url: http://example.org

url_pdf: 'https://ieeexplore.ieee.org/abstract/document/7390022'
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

