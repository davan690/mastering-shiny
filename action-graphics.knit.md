# Graphics {#action-graphics}



We talked briefly about `renderPlot()` in Chapter \@ref(basic-ui); it's a powerful tool for displaying graphics in your app. This chapter will show you how to use it to its full extent to create interactive plots, plots that respond to mouse events. You'll also learn about two important related functions: `renderCachedPlot()`, which speeds up your app by caching frequently used plots, and `renderImage()`, which allows you to display existing images.

In this chapter, we'll need ggplot2 as well as Shiny, since that's what I'll use for the majority of the graphics.


```r
library(shiny)
library(ggplot2)
#> Warning: package 'ggplot2' was built under R version 3.6.3
```

## Interactivity

One of the coolest things about `plotOutput()` is that as well as being an output that displays plots, it can also be an input that responds to pointer events. That allows you to create interactive graphics where the user interacts directly with the data on the plot. Interactive graphics are powerful tool, with a wide range of applications. I don't have space to show you all the possibilities, so here I'll focus on the basics, then point you towards resources to learn more.

### Basics

A plot can respond to four different mouse[^1] events: `click`, `dblClick` (double click), `hover` (when the mouse stays in the same place for a little while), and `brush` (a rectangular selection tool). To turn these events into Shiny inputs, you supply a string to the corresponding `plotOutut()` argument, e.g. `plotOutput("plot", click = "plot_click")`. This creates an `input$plot_click` that you can use to handle mouse clicks on the plot.

[^1]: Shiny didn't support touch events when I wrote this chapter, but it might by the time you read this.

Here's a very simple example of handling a mouse click. We register the `plot_click` input, and then use that to update an output with the coordinates of the mouse click:


```r
ui <- basicPage(
  plotOutput("plot", click = "plot_click"),
  verbatimTextOutput("info")
)

server <- function(input, output) {
  output$plot <- renderPlot({
    plot(mtcars$wt, mtcars$mpg)
  }, res = 96)

  output$info <- renderPrint({
    req(input$plot_click)
    x <- round(input$plot_click$x, 2)
    y <- round(input$plot_click$y, 2)
    cat("[", x, ", ", y, "]", sep = "")
  })
}
```

(Note the use of `req()`, to make sure the app doesn't do anything before the first click, and that the coordinates are in terms of the underlying `wt` and `mpg` variables.)

The following sections describe the events in more details. We'll start with the click events, then briefly discuss the closely related `dblClick` and `hover`. Then you'll learn about the `brush` event, which provides a rectangular "brush" defined by its four sides (`xmin`, `xmax`, `ymin`, and `ymax`). I'll then give a couple of examples of updating the plot with the results of the action, and then discuss some of the limitations of interactive graphics in Shiny.

### Clicking

The point events return a relatively rich list containing a lot of information. The most important components are `x` and `y`, which give the location of the event in data coordinates. But I'm not going to talk about this data structure, since you'll only need in relatively rare situations (If you do want the details, use [this app](https://gallery.shinyapps.io/095-plot-interaction-advanced/) in the Shiny gallery). Instead, you'll use the `nearPoints()` helper, which finds data points near the event, taking care of a bunch of fiddly details.

Here's a simple example of `nearPoints()` in action, showing a table of data about the points near the event:


```r
ui <- fluidPage(
  plotOutput("plot", click = clickOpts("click")),
  tableOutput("data")
)
server <- function(input, output, session) {
  output$plot <- renderPlot({
    plot(mtcars$wt, mtcars$mpg)
  }, res = 96)
  
  output$data <- renderTable({
    nearPoints(mtcars, input$click, xvar = "wt", yvar = "mpg")
  })
}
```

Here we give `nearPoints()` four arguments: the data frame that underlies the plot, the input event, and the names of the variables on the axes. If you use ggplot2, you only need to provide the first two arguments since `xvar` and `yvar` can be automatically imputed from the plot data structure. For that reason, I'll use ggplot2 throughout the rest of the chapter. Here's that previous example reimplemented with ggplot2:


```r
ui <- fluidPage(
  plotOutput("plot", click = "plot_click"),
  tableOutput("data")
)
server <- function(input, output, session) {
  output$plot <- renderPlot({
    ggplot(mtcars, aes(wt, mpg)) + geom_point()
  }, res = 96)
  
  output$data <- renderTable({
    nearPoints(mtcars, input$plot_click)
  })
}
```

Another way to use `nearPoints()` is with `allRows = TRUE` and `addDist = TRUE`. That will return the original data frame with two new columns:

-   `dist_` gives the distance between the row and the event (in pixels).
-   `selected_` says whether or not it should be selected (i.e. the logical vector that's returned.

We'll see an example of that a little later.

### Other point events

The same approach works equally well with `click`, `dblClick`, and `hover`: just change the name of the argument. If needed, you can get additional control over the events by supplying `clickOpts()`, `dblclickOpts()`, or `hoverOpts()` instead of a string giving the input id. These are rarely needed, so I won't discuss them here; see the documentation for details.

You can use multiple interactions types on one plot. Just make sure to explain to the user what they can do: one downside of using mouse events to interact with an app is that they're not immediately discoverable[^2].

[^2]: As a general rule, adding explanatory text suggests that your interface is too complex, so is best avoided, where possible. This is the key idea behind "affordances", the idea that an object should suggest naturally how to interact with it as introduced by Don Norman in the *"Design of Everyday Things"*.

### Brushing

Another way of selecting points on a plot is to use a **brush**, a rectangular selection defined by four edges. In Shiny, using a brush is straightforward once you've mastered `click` and `nearPoints()`: you just switch to `hover` argument and the `brushedPoints()` helper.

Here's another simple example that shows which points have been selected by the brush:


```r
ui <- fluidPage(
  plotOutput("plot", brush = "plot_brush"),
  tableOutput("data")
)
server <- function(input, output, session) {
  output$plot <- renderPlot({
    ggplot(mtcars, aes(wt, mpg)) + geom_point()
  }, res = 96)
  
  output$data <- renderTable({
    brushedPoints(mtcars, input$plot_brush)
  })
}
```

Use `brushOpts()` to control the colour (`fill` and `stroke`), or restrict brushing to a single dimension with `direction = "x"` or `"y"` (useful, e.g., for brushing time series).

### Modifying the plot

So far we've displayed the results of the interaction in another output. But the true beauty of interactivity comes when you display the changes in the same plot you're interacting with. Unfortunately this requires an advanced reactivity technique that you have yet learned about: `reactiveVal()`. We'll come back to `reactiveVal()` in Chapter \@ref(reactivity-components), but I wanted to show it here because it's such a useful technique. You'll probably need to re-read this section after you've read that chapter, but hopefully even without all the theory you'll get a sense of the potential applications.

As you might guess from the name, `reactiveVal()` is rather similar to `reactive()`. You create a reactive value by calling `reactiveVal()` with its initial value, and retrieve that value in the same way as a reactive:


```r
val <- reactiveVal(10)
val()
#> [1] 10
```

The big difference is that you can also **update** reactive values, and all reactive consumers that refer to it will recompute. A reactive value uses a special syntax for updating --- you call it like a function with the first argument being the new value:


```r
val(20)
val()
#> [1] 20
```

That means updating a reactive value using its current value looks something like this:


```r
val(val() + 1)
val()
#> [1] 21
```

Unfortunately if you actually try to run this code in the console you'll get an error because it has to be run in an reactive environment. That makes experimentation and debugging more challenging because you'll need to `browser()` or similar to pause execution within the call to `shinyApp()`. This is one of the challenges we'll come back to later in Chapter \@ref(reactivity-components).

For now, let's put the challenges of learning `reactiveVal()` aside, and show you why you might bother. Imagine that you want to visualise the distance between a click and the points on the plot. In the app below, we start by creating a reactive value to store those distances, initialising it with a constant that will be used before we click anything. Then we use `observeEvent()` to update the reactive value when the mouse is clicked, and a ggplot that visualises the distance with point size. All up, this looks something like:


```r
df <- data.frame(x = rnorm(100), y = rnorm(100))

ui <- fluidPage(
  plotOutput("plot", click = "plot_click")
)
server <- function(input, output, session) {
  dist <- reactiveVal(rep(1, nrow(df)))
  observeEvent(input$plot_click,
    dist(nearPoints(df, input$plot_click, allRows = TRUE, addDist = TRUE)$dist_)  
  )
  
  output$plot <- renderPlot({
    df$dist <- dist()
    ggplot(df, aes(x, y, size = dist)) + 
      geom_point() + 
      scale_size_area(limits = c(0, 1000), max_size = 10, guide = NULL)
  })
}
```

There are two important ggplot2 techniques to note here:

-   I add the distances to the data frame before plotting. I think it's good practice to put related variables together in a data frame before visualising it.
-   I set the `limits` to `scale_size_area()` to ensure that sizes are comparable across clicks. To find the correct range I did a little interactive experimentation, but you can work out the exact details if needed (see the exercises at the end of the chapter).

Here's a more complicated idea. I want to use a brush to select (and deselect) points on a plot. Here I display the selection using different colours, but you could imagine many other applications. To make this work, I initialise the `reactiveVal()` to a vector of `FALSE`s, then use `brushedPoints()` and `ifelse()` toggle their values: if they were previously excluded they'll be included; if they were previously included, they'll be excluded.


```r
ui <- fluidPage(
  plotOutput("plot", brush = "plot_brush"),
  tableOutput("data")
)
server <- function(input, output, session) {
  selected <- reactiveVal(rep(TRUE, nrow(mtcars)))

  observeEvent(input$plot_brush, {
    brushed <- brushedPoints(mtcars, input$plot_brush, allRows = TRUE)$selected_
    selected(ifelse(brushed, !selected(), selected()))
  })

  output$plot <- renderPlot({
    mtcars$sel <- selected()
    ggplot(mtcars, aes(wt, mpg)) + 
      geom_point(aes(colour = sel)) +
      scale_colour_discrete(limits = c("TRUE", "FALSE"))
  }, res = 96)
 
}
```

Again, I set the limits of the scale to ensure that the legend (and colours) don't change after the first click.

### Interactivity limitations

Before we move on, it's important to understand the basic data flow in interactive plots in order to understand their limitations. The basic flow is something like this:

1.  Javascript captures the mouse event.
2.  Shiny sends the javascript mouse event back to R, invalidating the input.
3.  Downstream reactive consumers are recomputed.
4.  `plotOutput()` generates a new PNG and sends it to the browser.

For local apps, the bottleneck tends to be the time taken to draw the plot. Depending on how complex the plot is, this may take a signficant fraction of a second. But for hosted apps, you also have to take into account the time needed to transmit the event from the browser to the R, and then the rendered plot back from R to the browser.

In general, this means that it's not possible to create Shiny apps where action and response is percieved as instanteous (i.e. the plot appears to update simultaneously with your action upon it). If you need that level of speed, you'll have to perform more computation in javascript. One way to do this is to use an R package that wraps a JS graphics library. Right now, as I write this book, I think you'll get the best experience with the plotly package, as documented in the book *[Interactive web-based data visualization with R, plotly, and shiny](https://plotly-r.com)*, by Carson Sievert.

## Theming

If you've heavily customised the style of your app, you may want to also customise your plots to match. Fortunately, this is very easy thanks to the [thematic](https://rstudio.github.io/thematic/) package by Carson Sievert. There are two main ways to use it. Firstly, you can explicitly set a theme defined by foreground, background, and accent colours (and font if desired):












