# Case study: ER injuries {#basic-case-study}



## Introduction

I've introduced you to a bunch of new concepts in the last three chapters. So to help them sink in, we'll now walk through a richer Shiny app that explores a fun dataset and pulls together many of the ideas that you've seen so far. We'll start by doing a little data analysis outside of Shiny, then turn it into an app, starting simply, then progressively layering on more detail.

In this chapter, we'll supplement Shiny with vroom (for fast file reading) and the tidyverse (for general data analysis).


```r
library(shiny)
library(vroom)
#> Warning: package 'vroom' was built under R version 3.6.3
library(tidyverse)
#> Warning: package 'tidyverse' was built under R version 3.6.3
#> Warning: package 'ggplot2' was built under R version 3.6.3
#> Warning: package 'tibble' was built under R version 3.6.3
#> Warning: package 'tidyr' was built under R version 3.6.3
#> Warning: package 'purrr' was built under R version 3.6.3
#> Warning: package 'dplyr' was built under R version 3.6.3
#> Warning: package 'stringr' was built under R version 3.6.3
#> Warning: package 'forcats' was built under R version 3.6.3
```

## The data

We're going to explore data from the National Electronic Injury Surveillance System (NEISS), collected by the Consumer Product Safety Commission. This is a long-term study that records all accidents seen in a representative sample of hospitals in the United States. It's an interesting dataset to explore because every one is already familiar with the domain, and each observation is accompanied by a short narrative that explains how the accident occurred. You can find out more about this dataset at <https://github.com/hadley/neiss>.

In this chapter, I'm going to focus on just the data from 2017. This keeps the data small enough (~10 meg) that it's easy to store in git (along with the rest of the book), which means we don't need to think about sophisticated strategies for importing the data quickly (we'll come back to those later in the book). You can see the code I used to create the extract for this chapter at <https://github.com/hadley/mastering-shiny/blob/master/neiss/data.R>.

The main dataset we'll use is `injuries`, which contains around 250,000 observations:


```r
injuries <- vroom::vroom("neiss/injuries.tsv.gz")
injuries
#> # A tibble: 255,064 x 10
#>   trmt_date    age sex   race  body_part diag  location prod_code weight
#>   <date>     <dbl> <chr> <chr> <chr>     <chr> <chr>        <dbl>  <dbl>
#> 1 2017-01-01    71 male  white Upper Tr~ Cont~ Other P~      1807   77.7
#> 2 2017-01-01    16 male  white Lower Arm Burn~ Home           676   77.7
#> 3 2017-01-01    58 male  white Upper Tr~ Cont~ Home           649   77.7
#> 4 2017-01-01    21 male  white Lower Tr~ Stra~ Home          4076   77.7
#> 5 2017-01-01    54 male  white Head      Inte~ Other P~      1807   77.7
#> 6 2017-01-01    21 male  white Hand      Frac~ Home          1884   77.7
#> # ... with 255,058 more rows, and 1 more variable: narrative <chr>
```

Each row represents a single accident with 10 variables:

* `trmt_date` is date the person was seen in the hospital (not when the 
   accident occurred).

* `age`, `sex`, and `race` give demographic information about the person 
  who experienced the accident.

* `body_part` is the location of the injury on the body (like ankle or ear); 
  `location` is the place where the accident occurred (like home or school).

* `diag` gives the basic diagnosis of the injury (like fracture or laceration).

* `prod_code` is the primary product associated with the injury.

* `weight` is statistical weight giving the estimated number of people who
  would suffer this injury if this dataset was scaled to the entire population 
  of the US.

* `narrative` is a brief story about how the accident occurred.

We'll pair it with two other data frames for additional context: `products` lets us look up the product name from the product code, and `population` tells us the total US population in 2017 for each combination of age and sex.


```r
products <- vroom::vroom("neiss/products.tsv")
products
#> # A tibble: 38 x 2
#>   prod_code title                            
#>       <dbl> <chr>                            
#> 1       464 knives, not elsewhere classified 
#> 2       474 tableware and accessories        
#> 3       604 desks, chests, bureaus or buffets
#> 4       611 bathtubs or showers              
#> 5       649 toilets                          
#> 6       676 rugs or carpets, not specified   
#> # ... with 32 more rows

population <- vroom::vroom("neiss/population.tsv")
population
#> # A tibble: 170 x 3
#>     age sex    population
#>   <dbl> <chr>       <dbl>
#> 1     0 female    1924145
#> 2     0 male      2015150
#> 3     1 female    1943534
#> 4     1 male      2031718
#> 5     2 female    1965150
#> 6     2 male      2056625
#> # ... with 164 more rows
```

## Exploration

Before we create the app, let's explore the data a little. We'll start by looking at the product associated with the most injuries: 1842, "stairs or steps". First we'll pull out the injuries associated with this product:


```r
selected <- injuries %>% filter(prod_code == 1842)
nrow(selected)
#> [1] 30647
```

Next we'll perform some basic summaries looking at the diagnosis, body part, and location where the injury occurred. Note that I weight by the `weight` variable so that the counts can be interpreted as estimated total injuries across the whole US.


```r
selected %>% count(diag, wt = weight, sort = TRUE)
#> # A tibble: 23 x 2
#>   diag                        n
#>   <chr>                   <dbl>
#> 1 Strain, Sprain        267892.
#> 2 Fracture              243082.
#> 3 Other Or Not Stated   227515.
#> 4 Contusion Or Abrasion 195172.
#> 5 Inter Organ Injury    111340.
#> 6 Laceration             89190.
#> # ... with 17 more rows

selected %>% count(body_part, wt = weight, sort = TRUE)
#> # A tibble: 25 x 2
#>   body_part         n
#>   <chr>         <dbl>
#> 1 Ankle       183470.
#> 2 Head        174725.
#> 3 Lower Trunk 150459.
#> 4 Knee        112162.
#> 5 Upper Trunk  98197.
#> 6 Face         73815.
#> # ... with 19 more rows

selected %>% count(location, wt = weight, sort = TRUE)
#> # A tibble: 8 x 2
#>   location                         n
#>   <chr>                        <dbl>
#> 1 Home                       647127.
#> 2 Unknown                    458802.
#> 3 Other Public Property       57625.
#> 4 School                      25146.
#> 5 Sports Or Recreation Place  11833.
#> 6 Street Or Highway            2148.
#> # ... with 2 more rows
```

As you might expect, steps are most often associated with sprains, strains and fractures, of the ankle, occurring at home.

We can also explore the pattern across age and sex. We have enough data here that a table is not that useful, and so I make a plot, Figure \@ref(fig:stairs-raw), that makes the patterns more obvious.


```r
summary <- selected %>% 
  count(age, sex, wt = weight)
summary
#> # A tibble: 204 x 3
#>     age sex         n
#>   <dbl> <chr>   <dbl>
#> 1     0 female  3714.
#> 2     0 male    3981.
#> 3     1 female 12155.
#> 4     1 male   12898.
#> 5     2 female  6949.
#> 6     2 male    9730.
#> # ... with 198 more rows

summary %>% 
  ggplot(aes(age, n, colour = sex)) + 
  geom_line() + 
  labs(y = "Estimated number of injuries")
```

<div class="figure" style="text-align: center">
<img src="basic-case-study_files/figure-html/stairs-raw-1.png" alt="Estimated number of injuries caused by stairs, broken down by age and sex" width="100%" />
<p class="caption">(\#fig:stairs-raw)Estimated number of injuries caused by stairs, broken down by age and sex</p>
</div>

We see a big spike when children are learning to walk, a flattening off over middle age, and then a gradual decline after age 50. Interestingly, the number of injuries is much higher for women. (Maybe this is due to high-heeled shoes?)

One problem with interpreting this pattern is that we know that there are fewer older people than younger people, so the population available to be injured is smaller. We can control for this by comparing the number of people injured with the total population and calculating an injury rate. Here I use a rate per 10,000.


```r
summary <- selected %>% 
  count(age, sex, wt = weight) %>% 
  left_join(population, by = c("age", "sex")) %>% 
  mutate(rate = n / population * 1e4)

summary
#> # A tibble: 204 x 5
#>     age sex         n population  rate
#>   <dbl> <chr>   <dbl>      <dbl> <dbl>
#> 1     0 female  3714.    1924145  19.3
#> 2     0 male    3981.    2015150  19.8
#> 3     1 female 12155.    1943534  62.5
#> 4     1 male   12898.    2031718  63.5
#> 5     2 female  6949.    1965150  35.4
#> 6     2 male    9730.    2056625  47.3
#> # ... with 198 more rows
```

Plotting the rate, Figure \@ref(fig:stairs-rate), yields a strikingly different trend after age 50: while the number of injuries decreases, the *rate* of injuries continues to increase.


```r
summary %>% 
  ggplot(aes(age, rate, colour = sex)) + 
  geom_line(na.rm = TRUE) + 
  labs(y = "Injuries per 10,000 people")
```

<div class="figure" style="text-align: center">
<img src="basic-case-study_files/figure-html/stairs-rate-1.png" alt="Estimated rate of injuries per 10,000 people, broken down by age and sex" width="100%" />
<p class="caption">(\#fig:stairs-rate)Estimated rate of injuries per 10,000 people, broken down by age and sex</p>
</div>

(Note that the rates only go up to age 80 because I couldn't find population data for ages over 80.)

Finally, we can look at some of the narratives. Browsing through these is an informal way to check our hypotheses, and generate new ideas for further exploration. Here I pull out a random sample of 10:



```r
selected %>% 
  sample_n(10) %>% 
  pull(narrative)
#>  [1] "33YOM PRESENTED TO ED AFTER ATEMPTING TO MOVE METAL STAIRS TODAY.C/O LOWER BACK PAIN.DX:BACK STRAIN"
#>  [2] "55YOF C/O FALL DOWN 17 STAIRS PTA. HIT HEAD. NO LOC DX=ACUTE SUBARACHNOID HEMORRHAGE="              
#>  [3] "48 YOF FELL DOWN 2 STEPS. C/O TOE AND WRIST PAIN DX WRIST SPRAIN, MULTIPLE CONTUSIONS"              
#>  [4] "33YOM FELL DOWN STAIRS 3 WKS AGO,LANDED ON BOTH ELBOWS; PAIN SINCEDX: LATERAL EPICONDYLITIS ELBOW"  
#>  [5] "47YOF FELL DOWN 10 CARPETED STEPS, HIT HEAD.  DX; LT WRIST PAIN /CHI"                               
#>  [6] "51YF S'D&F BWD ON ICE COVERED OUTSIDE STEPS STRIKING HEAD ONTO A STEP C+LOC>>CONCUSSION"            
#>  [7] "66 YO F S/P REPORTEDLY FELL DOWN APPROX 5 STAIRS AND HIT HER HEAD W/ PAIN TO LT HEAD DX FALL"       
#>  [8] "80YOF WITH CHEST WALL CONTUSION AFTER MISSING A STEP AND FALLING DX CONTUSION*"                     
#>  [9] "*35YOF,WALKING DOWNS STAIRS IN FLIPFLOPS SLIPPED LANDED ON BACK,BACKPAIN,DX:BACK INJURY"            
#> [10] "CONT ELBOW 44YOF FELL DOWN STAIRS STRUCK ELBOW ON DOOR KNOBDX: CONT ELBOW"
```

Having done this exploration for one product, it would be be very nice if we could easily do it for other products, without having to retype the code. So let's make a Shiny app!

## Prototype

When building a complex app, I strongly recommend starting as simple as possible, so that you can confirm the basic mechanics work before you start doing something more complicated. Here I'll start with one input (the product code), three tables, and one plot. 

When making a first prototype, the challenge is the "as simple _as possible_". There's a tension between getting the basics working quickly and planning for the future of the app. Either extreme can be bad: if you design too narrowly, you'll spend a lot of time later on reworking your app; if you design too rigorously, you'll spend a bunch of time writing code that later ends up on the cutting floor. To help get the balance right, I often do a few pencil-and-paper sketches to rapidly explore the UI and reactive graph before committing to code.

Here I decided to have one row for the inputs (accepting that I'm probably going to add more inputs before this app is done), one row for all three tables (giving each table 4 columns, 1/3 of the 12 column width), and then one row for the plot:


```r
ui <- fluidPage(
  fluidRow(
    column(6,
      selectInput("code", "Product", setNames(products$prod_code, products$title))
    )
  ),
  fluidRow(
    column(4, tableOutput("diag")),
    column(4, tableOutput("body_part")),
    column(4, tableOutput("location"))
  ),
  fluidRow(
    column(12, plotOutput("age_sex"))
  )
)
```

Note the use of `setNames()` in the `selectInput()` `choices`: this shows the product name in the UI and returns the product code to the server.

The server function is relatively straightforward. I first convert the `selected` and `summary` variables (defined above) to reactive expressions. This is a reasonable general pattern: you create variables in your data analysis to decompose the analysis into steps, and to avoid recomputing things multiple times, and reactive expressions play the same role in Shiny apps. 

Often it's a good idea to spend a little time cleaning up your analysis code before you start your Shiny app, so you can think about these problems in regular R code, before you add the additional complexity of reactivity.


```r
server <- function(input, output, session) {
  selected <- reactive(injuries %>% filter(prod_code == input$code))

  output$diag <- renderTable(
    selected() %>% count(diag, wt = weight, sort = TRUE)
  )
  output$body_part <- renderTable(
    selected() %>% count(body_part, wt = weight, sort = TRUE)
  )
  output$location <- renderTable(
    selected() %>% count(location, wt = weight, sort = TRUE)
  )

  summary <- reactive({
    selected() %>%
      count(age, sex, wt = weight) %>%
      left_join(population, by = c("age", "sex")) %>%
      mutate(rate = n / population * 1e4)
  })

  output$age_sex <- renderPlot({
    summary() %>%
      ggplot(aes(age, n, colour = sex)) +
      geom_line() +
      labs(y = "Estimated number of injuries")
  }, res = 96)
}
```

Note that creating the `summary` reactive isn't strictly necessary here, as it's only used by a single reactive consumer. But it's good practice to keep computing and plotting separate as it makes the flow of the app easier to understand, and will make it easier to generalise in the future.

A screenshot of the resulting app is shown in Figure \@ref(fig:prototype). You can find the source code at <https://github.com/hadley/mastering-shiny/tree/master/neiss/prototype.R> and try out a live version of the app at XYZ.


<div class="figure" style="text-align: center">
<img src="screenshots/basic-case-study/prototype.png" alt="First prototype of NEISS exploration app" width="100%" />
<p class="caption">(\#fig:prototype)First prototype of NEISS exploration app</p>
</div>

## Polish tables

Now that we have the basic components in place and working, we can progressively improve our app. The first problem with this app is that it shows a lot of information in the tables, where we probably just want the highlights. To fix this we need to first figure out how to truncate the tables. I've chosen to do that with a combination of forcats functions: I convert the variable to a factor, order by the frequency of the levels, and then lump together all levels after the top 5. 


```r
injuries %>%
  mutate(diag = fct_lump(fct_infreq(diag), n = 5)) %>%
  group_by(diag) %>%
  summarise(n = as.integer(sum(weight)))
#> # A tibble: 6 x 2
#>   diag                        n
#>   <fct>                   <int>
#> 1 Other Or Not Stated   1806436
#> 2 Fracture              1558961
#> 3 Laceration            1432407
#> 4 Strain, Sprain        1432556
#> 5 Contusion Or Abrasion 1451987
#> 6 Other                 1929147
```

Because I knew how to do it, I wrote a little function to automate this for any variable. The details aren't really important here; and don't worry if this looks totally foreign: you could also solve the problem via copy and paste.


```r
count_top <- function(df, var, n = 5) {
  df %>%
    mutate({{ var }} := fct_lump(fct_infreq({{ var }}), n = n)) %>%
    group_by({{ var }}) %>%
    summarise(n = as.integer(sum(weight)))
}
```

I then use this in the server function:


```r
  output$diag <- renderTable(count_top(selected(), diag), width = "100%")
  output$body_part <- renderTable(count_top(selected(), body_part), width = "100%")
  output$location <- renderTable(count_top(selected(), location), width = "100%")
```

I made one other change to improve the aesthetics of the app: I forced all tables to take up the maximum width (i.e. fill the column that they appear in). This makes the output more aesthetically pleasing because it reduces the amount of incidental variation.

A screenshot of the resulting app is shown in Figure \@ref(fig:polish-tables). You can find the source code at <https://github.com/hadley/mastering-shiny/tree/master/neiss/polish-tables.R> and try out a live version of the app at XYZ.


<div class="figure" style="text-align: center">
<img src="screenshots/basic-case-study/polish-tables.png" alt="The second iteration of the app improves the display by only showing the most frequent rows in the summary tables" width="100%" />
<p class="caption">(\#fig:polish-tables)The second iteration of the app improves the display by only showing the most frequent rows in the summary tables</p>
</div>

## Rate vs count

So far, we're displaying only a single plot, but we'd like to give the user the choice between visualising the number of injuries or the population-standardised rate. First I add a control to the UI. Here I've chosen to use a `selectInput()` because it makes both states explicit, and it would be easy to add new states in the future:


```r
  fluidRow(
    column(8,
      selectInput("code", "Product",
        choices = setNames(products$prod_code, products$title),
        width = "100%"
      )
    ),
    column(2, selectInput("y", "Y axis", c("rate", "count")))
  ),
```

(I default to `rate` because I think it's safer; you don't need to understand the population distribution in order to correctly interpret the plot.)

Then I condition on that input when generating the plot:


```r
  output$age_sex <- renderPlot({
    if (input$y == "count") {
      summary() %>%
        ggplot(aes(age, n, colour = sex)) +
        geom_line() +
        labs(y = "Estimated number of injuries")
    } else {
      summary() %>%
        ggplot(aes(age, rate, colour = sex)) +
        geom_line(na.rm = TRUE) +
        labs(y = "Injuries per 10,000 people")
    }
  }, res = 96)
```

A screenshot of the resulting app is shown in Figure \@ref(fig:rate-vs-count). You can find the source code at <https://github.com/hadley/mastering-shiny/tree/master/neiss/rate-vs-count.R> and try out a live version of the app at XYZ.


<div class="figure" style="text-align: center">
<img src="screenshots/basic-case-study/rate-vs-count.png" alt="In this iteration, we give the user the ability to switch between displaying the count or the population standardised rate on the y-axis." width="100%" />
<p class="caption">(\#fig:rate-vs-count)In this iteration, we give the user the ability to switch between displaying the count or the population standardised rate on the y-axis.</p>
</div>

## Narrative

Finally, I want to provide some way to access the narratives because they are so interesting, and they give an informal way to cross-check the hypotheses you come up with when looking at the plots. In the R code, I sample multiple narratives at once, but there's no reason to do that in an app where you can explore interactively.

There are two parts to the solution. First we add a new row to the bottom of UI. I use an action button to trigger a new story, and put the narrative in a `textOutput()`:


```r
  fluidRow(
    column(2, actionButton("story", "Tell me a story")),
    column(10, textOutput("narrative"))
  )
```

The result of an action button is an integer that increments each time it's clicked. Here I just use it to trigger a re-execution of the random selection:


```r
  output$narrative <- renderText({
    input$story
    selected() %>% pull(narrative) %>% sample(1)
  })
```

A screenshot of the resulting app is shown in Figure \@ref(fig:narrative). You can find the source code at <https://github.com/hadley/mastering-shiny/tree/master/neiss/narrative.R> and try out a live version of the app at XYZ.


<div class="figure" style="text-align: center">
<img src="screenshots/basic-case-study/narrative.png" alt="The final iteration adds the ability to pull out a random narrative from the selected rows" width="100%" />
<p class="caption">(\#fig:narrative)The final iteration adds the ability to pull out a random narrative from the selected rows</p>
</div>

## Exercises

1.  Draw the reactive graph for each app.

1.  What happens if you flip `fct_infreq()` and `fct_lump()` in the code that 
    reduces the summary tables?

1.  Add an input control that lets the user decide how many rows to show in the
    summary tables.

1.  Provide a way to step through every narrative systematically with forward
    and backward buttons. 
    
    Advanced: Make the list of narratives "circular" so that advancing 
    forward from the last narrative takes you to the first.

