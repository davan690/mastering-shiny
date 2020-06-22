# Preface



## What is Shiny?

If you've never used Shiny before, welcome! Shiny is an R package that allows you to easily create rich, interactive web apps. Shiny allows you to take your work in R and expose it via a web browser so that anyone can use it. Shiny makes you look awesome by making it easy to produce polished web apps with a minimum amount of pain.

In the past, creating web apps was hard for most R users because:

* You need a deep knowledge of web technologies like HTML, CSS and JavaScript. 

* Making complex interactive apps requires careful analysis of interaction
  flows to make sure that when an input changes, only the related outputs are 
  updated.

Shiny makes it significantly easier for the R programmer to create web apps by:

* Providing a carefully curated set of user interface (**UI** for short)
  functions that generate the HTML, CSS, and JavaScript that you need for 
  common tasks. This means that you don't need to know the details of 
  HTML/CSS/JS until you want to go beyond the basics that Shiny provides 
  for you.
  
* Introducing a new style of programming called __reactive programming__ which 
  automatically tracks the dependencies of a code chunk. This means that 
  whenever an input changes, Shiny can automatically figure out how to do the
  smallest amount of work to update all the related outputs.

People use Shiny to:

* Create dashboards that track important performance indicators, and facilitate
  drill down into surprising results.

* Replace hundreds of pages of PDFs with interactive apps that allow the 
  user to jump to the exact slice of the results that they care about.
  
* Communicate complex models to a non-technical audience with informative
  visualisations and interactive sensitivity analysis.

* Provide self-service data analysis for common workflows where replacing 
  email requests with a Shiny app that allows people to upload their own
  data and perform standard analyses.
  
* Create interactive demos for teaching statistics and data science concepts 
  that allow learners to tweak inputs and observe the downstream effects of 
  these changes in an analysis.
  
* Leverage automation processes by making them usable by users with no programming skills.  

* Create prototype of web application, as the development in R is often much faster but difficult to scale.

In short, Shiny gives you the ability to pass on some of your R superpowers to anyone who can use a web browser. 

## Who should read this book?

This book is aimed at two main audiences:

* R users who are interested in learning about Shiny in order to turn their
  analyses into interactive web apps. To get the most out of this book,
  you should be comfortable using R to do data analysis, and should have
  written at least a few functions.

* Existing Shiny users who want to improve their knowledge of the theory
  underlying Shiny in order to write higher-quality apps faster and more
  easily. You should find this book particularly helpful if your apps are
  starting to get bigger and you're starting to have problems managing the
  complexity.

## What will you learn?

The book is divided into five parts:

1.  In "Getting started", you'll learn the basics of Shiny so you can get up 
    and running as quickly as possible. You'll learn about the basics of app 
    structure, useful UI components, and the foundations of reactive 
    programming.
    
1.  "Shiny in action" builds on the basics to help you solve common problems
    including giving feedback to the user, uploading and downloading data,
    generating UI with code, reducing code duplication, and using Shiny to
    program the tidyverse.
    
1.  "Mastering UI" dives into the details of the user interface. You'll learn 
    packages that help you create other types of UI like dashboards and 
    RStudio gadgets, then learn the basics of HTML and CSS so you can customise 
    Shiny to precisely meet your needs.
    
1.  In "Mastering reactivity", you'll go deep into the theory and practice of 
    reactive programming, the programming paradigm that underlies Shiny. If 
    you're an existing Shiny user, you'll get the most value out of this 
    chapter as it will give you a solid theoretical underpinning that will 
    allow you to create new tools specifically tailored for your problems.

1.  Finally, in "Taming Shiny" we'll finish up with a survey of useful 
    techniques for making your Shiny apps work well in production. You'll learn 
    how to measure and improve performance, debug problems when they go wrong, 
    and manage your app's dependencies.

## What won't you learn?

The focus of this book is making effective Shiny apps and understanding the underlying theory of reactivity. I'll do my best to show case best practices for data science, R programming, and software engineering, but you'll need other references to master these important skillsets. If you enjoy my writing in this book, you might enjoy my other books on these topics: [R for data science](http://r4ds.had.co.nz/), [Advanced R](http://adv-r.hadley.nz/), and [R packages](http://r-pkgs.org/).

## Prerequisites {#prerequisites}

Before we continue, make sure you have all the software you need for this book:

*   __R__:  If you don't have R installed already, you may be reading the 
    wrong book; I assume a basic familiarity with R throughout this book.
    If you'd like to learn how to use R, I'd recommend my 
    [_R for Data Science_](https://r4ds.had.co.nz/) which is designed to get
    you up and running with R with minimum of fuss.

*   __RStudio__: RStudio is a free and open source integrated development 
    environment (IDE) for R. While you can write and use Shiny apps with any R 
    environment (including R GUI and [ESS](http://ess.r-project.org)), RStudio 
    has some nice features specifically for authoring, debugging, and deploying 
    Shiny apps. We recommend giving it a try, but it's not required to be 
    successful with Shiny or with this book. You can download RStudio Desktop
    from <https://www.rstudio.com/products/rstudio/download>

*   __R packages__: This book uses a bunch of R packages. You can install them 
    all at once by running:

    
    
    
    ```r
    install.packages(c(
      "gapminder", "ggforce", "openintro", "shiny", "shinycssloaders", 
      "shinyFeedback", "shinythemes", "thematic", "tidyverse", "vroom", 
      "waiter", "zeallot" 
    ))
    ```

## Acknowledgments

This book was written in the open and chapters were advertised on twitter when complete. It is truly a community effort: many people read drafts, fixed typos, suggested improvements, and contributed content. Without those contributors, the book wouldn't be nearly as good as it is, and I'm deeply grateful for their help. 



A big thank you to all 42 people who contributed specific improvements via GitHub pull requests (in alphabetical order by username): Adi Sarid (\@adisarid), Betsy Rosalen (\@betsyrosalen), Michael Beigelmacher (\@brooklynbagel), c1au6io_hh (\@c1au6i0), \@canovasjm, Chris Beeley (\@ChrisBeeley), \@chsafouane, Chuliang Xiao (\@ChuliangXiao), \@d-edison, DanielDavid521 (\@Danieldavid521), David Granjon (\@DivadNojnarg), Emilio (\@emilopezcano), Federico Marini (\@federicomarini), Frederik Kok Hansen (\@fkoh111), Hedley (\@heds1), James Pooley (\@jamespooley), Joe Cheng (\@jcheng5), Jim Hester (\@jimhester), Joachim Gassen (\@joachim-gassen), Jon Calder (\@jonmcalder), \@jyuu, Karandeep Singh (\@kdpsingh), Robert Kirk DeLisle (\@KirkDCO), Malcolm Barrett (\@malcolmbarrett), Marly Gotti (\@marlycormar), Matthew T. Warkentin (\@mattwarkentin), Matthew Berginski (\@mbergins), Mine Cetinkaya-Rundel (\@mine-cetinkaya-rundel), Maria Paula Caldas (\@mpaulacaldas), Pietro Monticone (\@pitmonticone), psychometrician (\@psychometrician), Tom Palmer (\@remlapmot), Matthew Sedaghatfar (\@sedaghatfar), Shixiang Wang (\@ShixiangWang), SÃ©bastien Rochette (\@statnmap), \@stevensbr, Tanner Stauss (\@tmstauss), Tony Fujs (\@tonyfujs), Jeff Allen (\@trestletech), Albrecht (\@Tungurahua), é»„æ¹˜äº‘ (\@XiangyunHuang), gXcloud (\@xwydq).

## Colophon

This book was written in [RStudio](http://www.rstudio.com/ide/) using [bookdown](http://bookdown.org/). The [website](http://mastering-shiny.org/) is hosted with [netlify](http://netlify.com/), and automatically updated after every commit by [travis-ci](https://travis-ci.org/). The complete source is available from [GitHub](https://github.com/hadley/mastering-shiny). 

This version of the book was built with R version 3.6.2 (2019-12-12) and the following packages:


|package         |version |source         |
|:---------------|:-------|:--------------|
|gapminder       |0.3.0   |CRAN (R 3.6.3) |
|ggforce         |0.3.1   |CRAN (R 3.6.1) |
|openintro       |NA      |NA             |
|shiny           |1.4.0   |CRAN (R 3.6.2) |
|shinycssloaders |0.3     |CRAN (R 3.6.2) |
|shinyFeedback   |NA      |NA             |
|shinythemes     |1.1.2   |CRAN (R 3.6.1) |
|thematic        |NA      |NA             |
|tidyverse       |1.3.0   |CRAN (R 3.6.1) |
|vroom           |NA      |NA             |
|waiter          |NA      |NA             |
|zeallot         |0.1.0   |CRAN (R 3.6.1) |



