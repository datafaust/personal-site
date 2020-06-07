+++
title = "Taxi Analytics and KPIs through R Shiny"
subtitle = "Simple metrics for a complicated story:rocket:"

date = 2018-11-05T00:00:00
lastmod = 2018-11-05T00:00:00
draft = false

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ['Fausto Lopez']

tags = ["Academic"]
summary = "Simple metrics for a complicated story."

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["deep-learning"]` references 
#   `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
# projects = ["internal-project"]

# Featured image
# To use, add an image named `featured.jpg/png` to your project's folder. 
[image]
  # Caption (optional)
  caption = ""

  # Focal point (optional)
  # Options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
  focal_point = ""

  # Show image only in page previews?
  preview_only = false

# Set captions for image gallery.
+++

Taxi Industry & Indicators

The Taxi & Limousine Commission of New York City (TLC) is the regulating body for all taxi and for hire vehicle service. In order to understand the industry they regulate data collection is key and quick metrics are important for tracking and getting ahead of shifts in an industry. TLC releases [industry indicators](http://www.nyc.gov/html/tlc/html/technology/aggregated_data.shtml).

In order to make this easier for the public, I created the [tlc fast dash](https://tlcanalytics.shinyapps.io/tlc_fast_dash/), a simple shiny application meant to help visualize these metrics for rapid use. The source code is available through the app or you can [click right here](https://gitlab.com/maverick_tlc/tlc_fast_dash). You can also see the official post at the [tlc medium blog](https://medium.com/@NYCTLC/simple-to-use-visualizations-for-trip-trends-6dd35ae1f247)

This dashboard has gone through many iterations in the beginning of my shiny R career, you can see some of the iterations in snapshots from before:



Soon to come is a more comprehensive dashboard, my team is working. Stay tuned.

