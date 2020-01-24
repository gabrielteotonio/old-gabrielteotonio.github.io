+++
title = "Web application development with R Shiny: how to scale data products quickly and reliably"
date = 2020-01-24T15:28:00  
draft = false

# Talk start and end times.
#   End time can optionally be hidden by prefixing the line with `#`.
time_start = 2019-10-11T11:10:00
time_end = 2019-10-11T12:00:00

authors = ["Gabriel Teotonio"]

# Abstract and optional shortened version.
abstract = "R is a programming language developed initially to deal with statistical computing and graphics. However, over the years, many features and packages have been added, offering users more tools to create entire environments. An example of this is the Shiny package: a framework for building web applications using R. Shiny applications are a powerful and easy way to share and communicate your analysis and allow people to interact with it. At In Loco, we have a series of applications built using such a framework, both for external and internal customers. In this lecture we will share the challenges and solutions applied during development."

# Name of event and optional event URL.
event = "The Developer's Conference"
event_url = "https://thedevconf.com/tdc/2020/index.html"

# Location of event
location = "Universidade Católica de Pernambuco, Recife - Brazil"

# Is this a selected talk? (true/false)
selected = true


# Tags (optional).
tags = ["R","Shiny", "Data", "Data Products", "Production"]

# Slides (optional).
#   Associate this talk with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references 
#   `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
slides = ""

# Links (optional).
url_pdf = ""
url_slides = "https://docs.google.com/presentation/d/1CZ0b_JN0GOvzNn5ojPhPeCQveqDf4U1a1_izvOdc3vQ/edit?usp=sharing"
url_video = ""
url_code = ""

# Does the content use math formatting?
math = false

# Featured image
[image]
  caption = "The Developer's Conference 2019 - Recife"

  # Focal point (optional)
  # Options: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight
  focal_point = "Center"
+++

At In Loco, we often need to create data validation applications very quickly for the validation process of the customers and teams involved. And because of this speed, these data pass only through the hands of our data analysts and scientists before being presented or exposed. About 16 TB of data are processed daily and creating architectures that support that amount is essential. One of the challenges is to create dashboards and applications that can be robust in various aspects such as scalability, security and performance in a short period of time and give the freedom for data scientists to transmit their analysis and development in a transparent manner.  
The lecture will initially address the Shiny framework, exploring its characteristics and exemplifying how it can be an outlet for sharing analysis and data modeling. In addition, it will be shown the possibility of using Shiny in the construction of data products without losing scalability and gaining speed in product development. We will now show a real use case for Shiny at In Loco: OOH Planner, a platform for analyzing the performance of offline media campaigns, and we will share the challenges we had in implementing it. Soon after, time will be spent explaining how to structure the backend and deploy these applications. At first, focusing on the choice of the database and good practices in determining the schema of the tables, in particular. Also used are tools used to build queries, perform performance and responsiveness tests, such as dplyr and shinyloadtest. In a second step, the deployment structure using docker. How to create containers for applications in Shiny and management of dependencies in R. Use of plumber APIs as good practice for code reproducibility, given that, possibly, applications will be processing classification and prediction models in their backend.
