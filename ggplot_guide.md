-   [A Layer](#a-layer)
    -   [(Aes)thetic mappings](#aesthetic-mappings)
    -   [Geoms & Stats](#geoms-stats)
    -   [Position](#position)
-   [A Layout](#a-layout)
    -   [Scales](#scales)
    -   [Facets](#facets)
    -   [Coordinate Systems](#coordinate-systems)
    -   [Themes](#themes)
-   [Rendering](#rendering)
    -   [ggsave](#ggsave)
    -   [Grid](#grid)
    -   [LaTeX integration](#latex-integration)
-   [Extensions](#extensions)
-   [Some DoPlots To Tittilate](#some-doplots-to-tittilate)
    -   [Segments & Border Whiskers](#segments-border-whiskers)
    -   [Heatmap & PCs](#heatmap-pcs)

This is an incomplete lesson, and an even worse reference meant to get
someone on their feet & give them a survey of the powers of ggplot2. For
a more complete guide see [the
book](https://github.com/hadley/ggplot2-book) and for a more complete
reference see [the
documentation](http://ggplot2.tidyverse.org/reference/).

To get started, the only thing your data needs to be is a) be, b) a
dataframe, c) "long", and d) properly classed. By properly classed I
don't mean it needs to be the diamonds dataset, I mean that each column
in your dataframe should be explicitly cast as the datatype you intend
it to be - as.factor() your factors and as.numeric() your numbers.

But we are gonna use diamonds because we are properly classed.

    library(ggplot2)
    head(diamonds)

    ## # A tibble: 6 x 10
    ##   carat       cut color clarity depth table price     x     y     z
    ##   <dbl>     <ord> <ord>   <ord> <dbl> <dbl> <int> <dbl> <dbl> <dbl>
    ## 1  0.23     Ideal     E     SI2  61.5    55   326  3.95  3.98  2.43
    ## 2  0.21   Premium     E     SI1  59.8    61   326  3.89  3.84  2.31
    ## 3  0.23      Good     E     VS1  56.9    65   327  4.05  4.07  2.31
    ## 4  0.29   Premium     I     VS2  62.4    58   334  4.20  4.23  2.63
    ## 5  0.31      Good     J     SI2  63.3    58   335  4.34  4.35  2.75
    ## 6  0.24 Very Good     J    VVS2  62.8    57   336  3.94  3.96  2.48

    # But we don't want to plot 50000 points every time so
    diamond_sub <- diamonds[sample(1:nrow(diamonds), 500, replace=FALSE),]

A Layer
=======

What is a statistical graphic? The good ole boys
[Wilkinson](http://www.springer.com/us/book/9780387245447) and
[Wickham](http://www.tandfonline.com/doi/abs/10.1198/jcgs.2009.07098)
tell us that they are

-   **Mappings** from data to aesthetic attributes (colour, shape, size)
-   Of **geometric objects** (points, lines bars)
-   On a particular **coordinate system**
-   Perhaps after a statistical **transformation**

We'll follow their lead. In ggplot, that combination of things forms a
layer. The terminology in ggplot is

-   **aes** - aesthetic mappings
-   **geom** - geometric objects
-   **scale** - ...scales
-   **stat** - statistical transformations

Reordered, the flow of information in ggplot2 is: \* **data** is
attached to the ggplot call, \* mapped by **aes**, and \* transformed by
**stat** before being passed to a \* **geom**, \* which is placed,
sized, and colored according to its relevant **scales** \* then ta-da!
rendered plot.

(Aes)thetic mappings
--------------------

Lets get ahead of ourselves and make a plot because we haven't made one
yet and our console is whining in the backseat.

    g <- ggplot(diamond_sub, aes(x=carat, y=price))+
      geom_point()
    g

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-2-1.png)

Nice. You are now better qualified to make graphics than 90% of business
dudes who still plot with excel for some reason. What did we do there?

We made a ggplot object named "g" that is attached to the dataframe
"diamond\_sub" `g <- ggplot(diamond_sub,` and has the aesthetic
parameters "x" and "y" mapped to the columns "carat" and "price,"
respectively `aes(x=carat, y=price))`. Then we added `+` another element
to our plot, `geom_point()` which tells us that we want our aesthetic
mapping represented geometrically as points. Note that we have to add
parentheses to tell R that we want that function evaluated even if we
don't pass any arguments. We then rendered the plot by calling the
variable name `g`.

Each geom (or more precisely, each stat, but we'll get to that in a bit)
has a certain subset of aes's that apply to it (don't we all).

You can list all aes's with

    ggplot2:::.all_aesthetics

    ##  [1] "adj"        "alpha"      "angle"      "bg"         "cex"       
    ##  [6] "col"        "color"      "colour"     "fg"         "fill"      
    ## [11] "group"      "hjust"      "label"      "linetype"   "lower"     
    ## [16] "lty"        "lwd"        "max"        "middle"     "min"       
    ## [21] "pch"        "radius"     "sample"     "shape"      "size"      
    ## [26] "srt"        "upper"      "vjust"      "weight"     "width"     
    ## [31] "x"          "xend"       "xmax"       "xmin"       "xintercept"
    ## [36] "y"          "yend"       "ymax"       "ymin"       "yintercept"
    ## [41] "z"

or by doing the same on the proto version of any geom, like

    ggplot2:::GeomPoint

    ## <ggproto object: Class GeomPoint, Geom>
    ##     aesthetics: function
    ##     default_aes: uneval
    ##     draw_group: function
    ##     draw_key: function
    ##     draw_layer: function
    ##     draw_panel: function
    ##     extra_params: na.rm
    ##     handle_na: function
    ##     non_missing_aes: size shape colour
    ##     optional_aes: 
    ##     parameters: function
    ##     required_aes: x y
    ##     setup_data: function
    ##     use_defaults: function
    ##     super:  <ggproto object: Class Geom>

Note the "required\_aes" and "non\_missing\_aes" for required and
optional aes, respectively.

In general, you'll always be able to map color (or colour, color is just
an alias), x, and y. Commonly you'll also be able to map fill for things
that take up area.

Aesthetics declared in the original call to `ggplot`, unless otherwise
specified, apply to each added element. Within a given element, you can
add, override, or remove aesthetics, like so:

    # Add coloring by ... color ... to the points
    ggplot(diamond_sub, aes(x=carat, y=price))+
      geom_line()+
      geom_point(aes(color=color))

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-5-1.png)

    # You can make the same plot (except for order) in a worse way by specifying color in the base ggplot call and removing it with NULL from the lines. 
    ggplot(diamond_sub, aes(x=carat, y=price, color=color))+
      geom_point()+
      geom_line(aes(color=NULL))

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-6-1.png)

Damn those are some ugly graphs, but this guide is about "can" not
"should." Notice how the order in which you add elements matters - the
plints are 'on top of' the lines in the first plot because the
`geom_point` call came second, and vice-versa in the second plot.

You can also perform operations within the aes(), say if you want to
fatten out a ribbon without it being a big goofy line

    # The economics dataset with an added 'epoch' factor to recolor every 100 points.
    economics_epoch <- economics
    economics_epoch$epoch <- as.factor(c(rep(1,100),
                                         rep(2,100),
                                         rep(3,100),
                                         rep(4,274)))
    ggplot(economics_epoch, aes(x=date,color=epoch))+
      geom_line(aes(y=unemploy), size=10)

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-7-1.png)

So goofy. Compared to...

    ggplot(economics_epoch, aes(x=date,fill=epoch))+
      geom_ribbon(aes(ymax=unemploy,ymin=unemploy-1000))

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-8-1.png)

Also, it's common to confuse "fill" and "color." You can see the
difference:

    ggplot(economics_epoch, aes(x=date,color=epoch))+
      geom_ribbon(aes(ymax=unemploy,ymin=unemploy-1000))

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-9-1.png)

And if you wanted to use both for different reasons because you're a
maniac

    ggplot(economics_epoch, aes(x=date,color=epoch, fill=rev(epoch)))+
      geom_ribbon(aes(ymax=unemploy,ymin=unemploy-1000))

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-10-1.png)

You can also assign single values to an aes, though as we'll see, it's
usually better to do that in a scale. You can give colors using [color
names](http://www.stat.columbia.edu/~tzheng/files/Rcolor.pdf) or [RGB
hex values](https://coolors.co) passed outside the aes argument -
remember the aes is a mapping from the data to the geom! Assigning a
`fill` within the `aes` argument assumes you mean that your hex value is
a level in a factor (and who could blame it), but assigning it outside
the `aes` argument means that you are sending that color to the geom
directly.

    ggplot(economics_epoch, aes(x=date))+
      geom_ribbon(aes(ymax=unemploy,ymin=unemploy-1000), fill="#F0A125")

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-11-1.png)

ggplot will do its best to figure out how to split your data when
combining multiple aesthetics, but here is one extra trick:

Say you want to split your data by two factors but don't want to make a
new column in your dataframe or color them. You can use the
`interaction()` function like (ignore the use of theme(), we'll get to
that):

    # we have tow factors, cut and color, with the levels...
    levels(diamond_sub$cut)

    ## [1] "Fair"      "Good"      "Very Good" "Premium"   "Ideal"

    levels(diamond_sub$color)

    ## [1] "D" "E" "F" "G" "H" "I" "J"

    # and we want to make a plot with all their permutations:
    ggplot(diamond_sub, aes(x=interaction(cut,color)))+
      geom_bar()+
      theme(
        axis.text.x = element_text(angle=90)
      )

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-12-1.png)

I'll give ya a list of all the aesthetics by shapes once we know what a
stat is.

Geoms & Stats
-------------

I've already thrown a lot of geoms your way, but there are a lot more
where that came from. I'm not going to do a complete runthrough of all
of them here because most of their use is relatively obvious once you
know how to manipulate aesthetics, but if you're hungry for it someone
put together a nice graphical summary of the available geoms:

![](ggplot_guide_files/figure-html/ggplot2_geoms.png)

This section is more concerned with the relationship between geoms and
stats. Behind every great geom there's a great stat. Stats act as
intermediary functions between the incoming data and the geom - put
another way, they relate the aes to the geom. Most common geoms just use
the `stat_identity` that returns the number, but many can accept a
`stat="stat_as_string"` argument. Below is a table of geoms, their
default stats, and their related aesthetic options (from page 57 of the
ggplot book)

![](ggplot_guide_files/figure-html/geoms_stat_aes.png)

A few examples of how to represent frequency data by manipulating stat:

A classic histogram

    ggplot(diamond_sub, aes(x=price))+
      geom_histogram(color="white", bins = 20)

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-13-1.png)

This is equivalent to the call

    ggplot(diamond_sub, aes(x=price))+
      stat_bin(geom="bar", color="white", bins = 20)

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-14-1.png)

If instead one wants the scale as a density,

    ggplot(diamond_sub, aes(x=price))+
      geom_histogram(aes(y=..density..), color="white", bins = 20)

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-15-1.png)

Other stats can be accessed by calling another geom that uses it as
their default. We can estimate a density kernel like:

    ggplot(diamond_sub, aes(x=price))+
      geom_histogram(aes(y=..density..), color="white", bins = 20)+
      geom_density()

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-16-1.png)

Yikes, our kernel is a little soft. We can adjust the bandwidth of the
kernel (as its standard deviation)...

    ggplot(diamond_sub, aes(x=price))+
      geom_histogram(aes(y=..density..), color="white", bins = 20)+
      geom_density(bw=300)

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-17-1.png)

How about some two dimensional data? I need lines. thankfully we can
directly perform linear models in ggplot.

    ggplot(diamond_sub, aes(x=table, y=depth))+
      geom_point(alpha=0.2)+
      geom_smooth(method="lm", color="red", se=FALSE)

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-18-1.png)

And if you want your confidence intervals right there with ya too

    ggplot(diamond_sub, aes(x=depth, y=table))+
      geom_point(alpha=0.2)+
      geom_smooth(color="red", method="lm", level=0.99)

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-19-1.png)

We can also perform a few other smoothing/estimation operations, for
example

    ggplot(diamond_sub, aes(x=table, y=depth))+
      geom_point(alpha=0.2)+
      geom_smooth(method="loess", color="red")

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-20-1.png)

And with a bit of convincing, ggplot will even do logistic regression
for us

    ggplot(midwest, aes(x=percbelowpoverty, y=inmetro))+
      geom_point(alpha=0.1)+
      geom_smooth(method="glm", method.args=list(family="binomial"), color="red")

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-21-1.png)

As we saw before, most geoms can be controlled directly by calling their
related stat. When doing so, we can select which geom we'd like to plot
our error bars with.

    ggplot(economics, aes(x=date, y=unemploy))+
      geom_line(color="gray")+
      stat_smooth(geom="linerange", size = 0.5, method="loess", level=0.99)

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-22-1.png)

How about some denser data. Remember our very first plot? There's a
whole lot more data that we randomly ignored. What if we weren't some
fraudulators and didn't conveniently "lose our data?"

    ggplot(diamonds, aes(x=carat, y=price))+
      geom_point(alpha=0.02)

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-23-1.png)

Wow that's cool and smoky and all, but is it a good representation of
the data?. If we summarize it with some hexagons ...

    ggplot(diamonds, aes(x=carat, y=price))+
      geom_hex(bins=50)

    ## Warning: Computation failed in `stat_binhex()`:
    ## Package `hexbin` required for `stat_binhex`.
    ## Please install and try again.

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-24-1.png)

Turns out it's really easy to misrepresent your data if it's not
presented in a quantitatively meaningful way. Most of the density of our
plot is in the lowest left section. Let's try one more visualization
method. Notice the `..density..` notation. Since density is a computed
function, it is wrapped in a sorta weird half ellipse to make sure it
doesn't conflict with the dataframe in case it has a variable called
density.

    ggplot(diamonds, aes(x=carat, y=price))+
      stat_density_2d(geom = "point", aes(size = ..density.., color= ..density..), n=50, contour = F)

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-25-1.png)

Position
--------

Is also important but I'm not covering it right now. ve most deel vis
it.

Until I do, if your bar charts aren't looking right just monkey around
on google with the phrase `dodge`

    ggplot(diamond_sub, aes(x=price, fill=cut))+
      geom_histogram()

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-26-1.png)

    ggplot(diamond_sub, aes(x=price, fill=cut))+
      geom_histogram(position="dodge")

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-27-1.png)

and maybe jitter

    ggplot(diamond_sub, aes(x=cut, y=price))+
      geom_point()

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-28-1.png)

    ggplot(diamond_sub, aes(x=cut, y=price))+
      geom_point(position=position_jitter(width=0.25))

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-29-1.png)

and maybe one more with `fill`

    ggplot(diamonds, aes(x=price, fill=cut))+
      geom_histogram(bins=50)

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-30-1.png)

    ggplot(diamonds, aes(x=price, fill=cut))+
      geom_histogram(bins=50, position="fill")

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-31-1.png)

A Layout
========

We have led our little geometric lambs to water, but what good is it if
we don't own the land?

Scales
------

Every aesthetic component, including axes, is a scale. There are two
types of scales, continuous and discrete. With scales, we control things
like

-   **Color**
-   **Breaks** - if it's an axis, where do we have ticks, if it's a
    legend, what is marked in the legend?
-   **Labels** - what do we call our breaks?
-   **Limits**
-   **Expand** - how much extra room we leave on the edges
-   **Transformations** - log scales & arbitrary transformations
-   **Position** - If an axis, reposition top, bottom, left, or right
    depending on whether it's an x or y axis.

The general syntax to modify your scale is `scale_aesthetic_continuous`
or `scale_aesthetic_discrete`, for example `scale_x_continuous` or
`scale_fill_discrete`.

For now, not going to spend much time on this because they are
relatively self explanatory, but a few specific useful examples

### Color Brewer

If you don't want to manually input colors and want to just follow best
principles, color brewer is a godsend. Visit the [color brewer
website](http://colorbrewer2.org) to see available color palettes. Or we
can use a tool from RColorBrewer (to be covered in extensions):

    brewplot <- RColorBrewer::display.brewer.all()

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-32-1.png)

To simply drop it in to any plot:

    ggplot(economics_epoch, aes(x=date,fill=epoch))+
      geom_ribbon(aes(ymax=unemploy,ymin=unemploy-1000))+
      scale_fill_brewer(palette="Set1")

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-33-1.png)

And the continuous variant is called "Distiller"

    ggplot(diamond_sub, aes(x=carat,y=price,color=price))+
      geom_point()+
      scale_color_distiller(palette="RdPu")+
      theme_dark()

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-34-1.png)

### Manual Colors

To use your own colors, use `scale_color_manual` or `scale_fill_manual`
(todo, trying to get some actually cool stuff in here)

Facets
------

(Skipping detailed description for now... but)

    ggplot(diamonds, aes(x=carat, y=price))+
      stat_density_2d(geom = "point", aes(size = ..density.., color = ..density..), 
                      n=10, contour = F)+
      scale_color_distiller(palette="RdPu")+
      facet_grid(cut~color)

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-35-1.png)

Coordinate Systems
------------------

(skipping for now)

Themes
------

The `theme()` command allows you fine control over each plot element.
There are a few general types of plot elements:

-   line
-   rect
-   text

That are grouped by where they are used, for example in

-   axis
-   legend
-   panel - individual plotting areas, for example each facet in a facet
    grid
-   plot - the entire plotting area
-   strip - titles on facets

Settings are inherited hierarchically. For example, one can set the size
of all text, the size of all axis titles, or just the x axis title.
Reference to each is straightforward:

`text = element_text(),` `axis.title = element_text(),`
`axis.title.x = element_text()`

ggplot comes with a few built-in themes:

    ggplot(diamond_sub, aes(x=carat,y=price,color=price))+
      geom_point()+
      theme_minimal()

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-36-1.png)

    ggplot(diamond_sub, aes(x=carat,y=price,color=price))+
      geom_point()+
      theme_void()

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-37-1.png)

### Removing elements

Sometimes you want to be all draped in pearls and roses, and other times
you just need your black turtleneck. To remove items, replace them with
a blank element:

    ggplot(diamond_sub, aes(x=carat,y=price,color=price))+
      geom_point()+
      theme(
        panel.background = element_blank(),
        panel.grid       = element_blank(),
        axis.text        = element_blank(),
        axis.ticks       = element_blank(),
        axis.title       = element_blank(),
        axis.line        = element_line(),
        legend.position  = "none"
      )

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-38-1.png)

And sometimes you just want to make a mockery of your audience's eyes
like you've got your hands in the MS paint bucket again. Notice how
since we are in a "g" rammar of "g" raphics ("plot") ("2") things work
as we expect them to

    ggplot(diamond_sub, aes(x=carat,y=price))+
      geom_point(color="pink")+
      theme(
        panel.background= element_rect(fill="yellow"),
        panel.grid      = element_blank(),
        axis.text       = element_blank(),
        axis.ticks      = element_blank(),
        axis.title      = element_blank(),
        axis.line       = element_line(),
        legend.position = "none"
      )

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-39-1.png)

A useful tool here is the direct specification of sizes with units. The
general syntax for units are unit(number, "unit\_type"), for example if
you are trying to shout about how big those numbers are and dont
understand what axis ticks are for

    ggplot(diamond_sub, aes(x=carat,y=price))+
      geom_point()+
      theme(
        panel.background= element_blank(),
        panel.grid      = element_blank(),
        axis.text       = element_text(size=unit(24,"pt")),
        axis.ticks      = element_line(arrow=arrow(type="closed")),
        axis.title      = element_blank(),
        axis.line       = element_line(),
        legend.position = "none"
      )

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-40-1.png)

Uh oh, who is screwing with our medicine? Our text is now hopelessly
buried beneath our arrows. We can fix that by adjusting our margins. The
margins are set in the order top, right, bottom, left. Since that
doesn't really make sense, they advise you to remember the phrase
"trouble" and that actually works. Notice the inheritance. Even though
we make settings for the x and y axis text specifically, it still
retains our 24 point mistake. Margins can also be used with the
plot.background to eliminate excess space, and can be set to negative
numbers.

    ggplot(diamond_sub, aes(x=carat,y=price))+
      geom_point()+
      theme(
        panel.background= element_blank(),
        panel.grid      = element_blank(),
        axis.text       = element_text(size=unit(24,"pt")),
        axis.text.x    = element_text(margin= margin(20,0,0,0, unit="pt")),
        axis.text.y     = element_text(margin= margin(0,20,0,0, unit="pt")),
        axis.ticks      = element_line(arrow=arrow(type="closed")),
        axis.title      = element_blank(),
        axis.line       = element_line(),
        legend.position = "none"
      )

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-41-1.png)

Rendering
=========

Having plots in your R environment is one thing, but getting them out
into the world or stuffed into your papers is quite another. Without
explicitly specifying certain parameters like sizes and positions, your
figure will end up with text crowded across the screen or nonevident
without microscopy. It's always a good idea to make those specifications
explicit with the theme() function, but some things require that ggplot
has knowledge about the rendering environment.

ggsave
------

What happens when we just try to export our figure window?

    # Scales lets us change the y axis numbers from scientific notation to decimal notation
    library(scales)

    g.price_density <- ggplot(diamond_sub, aes(x=price, y=..density.., fill=cut))+
      geom_density(position="stack")+
      scale_fill_brewer(palette="Set1", guide=guide_legend(title="Cut"))+
      scale_y_continuous(name="Density", labels=comma)+
      scale_x_continuous(name="Price")+
      theme_minimal()+
      theme(
        panel.grid.major = element_blank(),
        legend.position  = c(.8,.65)
      )

    # Saving figure by clicking "export" from figure window

![](ggplot_guide_files/figure-html/Rplot01.png)

Good god what have you done to the boy. Since the rendering is dependeng
on the size of the plotting environment ggplot has rendered it into, our
plot comes out looking very sad indeed.

The basic tool for exporting ggplot figures is the function ggsave. It
allows us to explicitly specify the size, resolution, rendering engine,
etc. Ggplot has many saving "devices," or rendering engines, the basic
categories of which are raster engines (like png) - which convert your
plot into a series of pixels and color values, and vector engines (like
pdf or svg) that leave the plot as a set of logical instructions. You
should preserve your plots as vectors as long as you possibly can, as
they are lossless representations of the image, and allow you to edit it
by element in another program.

    ggsave("price_density_plot.png", g.price_density,
           path="ggplot_guide_files/figure-html", # you can use relative or absolute paths
           device="png",
           width=unit(6,"in"), height=unit(4,"in"),
           dpi=300
    )

![](ggplot_guide_files/figure-html/price_density_plot.png) Much better.

Grid
----

Ok, we might know what a statistical plot is, but what is a ggplot?
Ggplot follows the [grid graphics
model](https://www.stat.auckland.ac.nz/~paul/RGraphics/chapter5.pdf),
which is a set of primitive graphical elements like circles, lines,
rectangles, that have some notion of their position within a "viewport"
(ggplot's panels). GRaphical OBjects in grid are called "grobs," and we
can convert our plot to be able to directly manipulate its grobs.

    g.grob <- ggplotGrob(g.price_density)

Your plot's layout information is in its `layout`, `heights`, and
`widths` attributes. Each of the numbers in the layout matrix
corresponds to a width (for the "t"(op), and "b"(ottom) columns), or
height (for the "l" and "r"). Each grob has its outer extent specified
by these values, so the background has its top at position 1, left at
position 1, bottom at 10 and right at 7 - the largest values of each,
predictably. By default, heights and widths of undeclared elements are
"null" or "grobheight" units that depend on the size of the viewport.
You can set them explicitly as you might expect with
`g.grob$heights[5] <- unit(5,"in")` or whatever. This becomes critical
for ensuring axes across multiple plots are the same size.

The "children" or "grobs" of the main grob object contain each of the
literal graphical objects, and they behave just like the parent.

    g.grob$layout

    ##    t l  b r  z clip       name
    ## 18 1 1 10 7  0   on background
    ## 1  5 3  5 3  5  off     spacer
    ## 2  6 3  6 3  7  off     axis-l
    ## 3  7 3  7 3  3  off     spacer
    ## 4  5 4  5 4  6  off     axis-t
    ## 5  6 4  6 4  1   on      panel
    ## 6  7 4  7 4  9  off     axis-b
    ## 7  5 5  5 5  4  off     spacer
    ## 8  6 5  6 5  8  off     axis-r
    ## 9  7 5  7 5  2  off     spacer
    ## 10 4 4  4 4 10  off     xlab-t
    ## 11 8 4  8 4 11  off     xlab-b
    ## 12 6 2  6 2 12  off     ylab-l
    ## 13 6 6  6 6 13  off     ylab-r
    ## 14 6 4  6 4 14  off  guide-box
    ## 15 3 4  3 4 15  off   subtitle
    ## 16 2 4  2 4 16  off      title
    ## 17 9 4  9 4 17  off    caption

    g.grob$heights

    ##  [1] 5.5pt                    0cm                     
    ##  [3] 0cm                      0cm                     
    ##  [5] 0cm                      1null                   
    ##  [7] sum(2.75pt, 1grobheight) 1grobheight             
    ##  [9] 0cm                      5.5pt

    g.grob$widths

    ## [1] 5.5pt                   1grobwidth              sum(1grobwidth, 2.75pt)
    ## [4] 1null                   0cm                     0cm                    
    ## [7] 5.5pt

    g.grob$grobs

    ## [[1]]
    ## zeroGrob[plot.background..zeroGrob.2847] 
    ## 
    ## [[2]]
    ## zeroGrob[NULL] 
    ## 
    ## [[3]]
    ## absoluteGrob[GRID.absoluteGrob.2820] 
    ## 
    ## [[4]]
    ## zeroGrob[NULL] 
    ## 
    ## [[5]]
    ## zeroGrob[NULL] 
    ## 
    ## [[6]]
    ## gTree[panel-1.gTree.2802] 
    ## 
    ## [[7]]
    ## absoluteGrob[GRID.absoluteGrob.2814] 
    ## 
    ## [[8]]
    ## zeroGrob[NULL] 
    ## 
    ## [[9]]
    ## zeroGrob[NULL] 
    ## 
    ## [[10]]
    ## zeroGrob[NULL] 
    ## 
    ## [[11]]
    ## zeroGrob[NULL] 
    ## 
    ## [[12]]
    ## titleGrob[axis.title.x..titleGrob.2805] 
    ## 
    ## [[13]]
    ## titleGrob[axis.title.y..titleGrob.2808] 
    ## 
    ## [[14]]
    ## zeroGrob[NULL] 
    ## 
    ## [[15]]
    ## TableGrob (3 x 3) "guide-box": 2 grobs
    ##                                     z     cells                  name
    ## 99_281d9cc4e523da50b6c36763eedff87c 1 (2-2,2-2)                guides
    ##                                     0 (1-3,1-3) legend.box.background
    ##                                               grob
    ## 99_281d9cc4e523da50b6c36763eedff87c gtable[layout]
    ##                                     zeroGrob[NULL]
    ## 
    ## [[16]]
    ## zeroGrob[plot.subtitle..zeroGrob.2845] 
    ## 
    ## [[17]]
    ## zeroGrob[plot.title..zeroGrob.2844] 
    ## 
    ## [[18]]
    ## zeroGrob[plot.caption..zeroGrob.2846]

We can see that deep down at the core of our plot is just a bunch of
polygons. I arrived at these numbers by just listing them recursively
(calling `g.grob$grobs`, seeing which one I needed, etc.). We can
directly edit the graphical properties of these polygons (`gp`) as you
might expect. We then

    # We have to load the grid library to do some commands, might as well load it now
    library(grid)

    g.grob$grobs[[6]]$children[[3]]$children

    ## (polygon[geom_ribbon.polygon.2780], polygon[geom_ribbon.polygon.2782], polygon[geom_ribbon.polygon.2784], polygon[geom_ribbon.polygon.2786], polygon[geom_ribbon.polygon.2788])

    g.grob$grobs[[6]]$children[[3]]$children[[1]]$gp$lwd <- 10
    g.grob$grobs[[6]]$children[[3]]$children[[1]]$gp$fill <- "#FF51FD"
    g.grob$grobs[[6]]$children[[3]]$children[[1]]$gp$col <- "yellow"

    # grid.newpage() clears the current viewport (so your plots don't just keep stacking up)
    # grid.draw() redraws our grob in the current viewport

    grid.newpage()
    grid.draw(g.grob)

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-46-1.png)

I have no idea why you keep wanting to do this to your plots. Lets try
to use grid to arrange two plots that we couldn't using ggplot alone.
Let's try to get density plots next to our scatterplot

    # We'll need "gtable" to tread the grids like a table
    library(gtable)

    g.scatter <- ggplot(diamond_sub, aes(x=carat, y=price, color=cut))+
      geom_point(size=0.5)+
      scale_color_brewer(palette="Set1", guide=guide_legend(title="Cut"))+
      scale_y_continuous(name="Price")+
      scale_x_continuous(name="Carat")+
      theme_minimal()+
      theme(
        legend.position=c(0.15,0.8),
        legend.background = element_rect(fill="#EFEFEF")
      )

    g.density_x <- ggplot(diamond_sub, aes(x=carat, y=..density.., fill=cut))+
      geom_density(position="stack")+
      scale_fill_brewer(palette="Set1")+
      #scale_x_continuous(expand=c(0,0))+
      theme_void()+
      theme(
        legend.position="none"
      )

    g.density_y <- ggplot(diamond_sub, aes(x=price, y=..density.., fill=cut))+
      geom_density(position="stack")+
      scale_fill_brewer(palette="Set1")+
      #scale_x_continuous(expand=c(0,0))+
      coord_flip()+ # Flip the coordinates so x is y, y i x
      theme_void()+
      theme(
        legend.position="none"
      )

    # Make our grobs
    g.scatter_grob   <- ggplotGrob(g.scatter)
    g.density_x_grob <- ggplotGrob(g.density_x)
    g.density_y_grob <- ggplotGrob(g.density_y)

    # Add a row and a column to the scatterplot where we'll add our density plots
    g.scatter_grob <- gtable_add_rows(g.scatter_grob,
                                      unit(0.1,"npc"), # npc is a unit that's defined by the viewport
                                      0) # position - we add to the top
    g.scatter_grob <- gtable_add_cols(g.scatter_grob,
                                      unit(0.1,"npc"), # npc is a unit that's defined by the viewport
                                      -1) # position - we add on the right

    # Let's check our layout
    g.scatter_grob$layout

    ##     t l  b r  z clip       name
    ## 18  2 1 11 7  0   on background
    ## 1   6 3  6 3  5  off     spacer
    ## 2   7 3  7 3  7  off     axis-l
    ## 3   8 3  8 3  3  off     spacer
    ## 4   6 4  6 4  6  off     axis-t
    ## 5   7 4  7 4  1   on      panel
    ## 6   8 4  8 4  9  off     axis-b
    ## 7   6 5  6 5  4  off     spacer
    ## 8   7 5  7 5  8  off     axis-r
    ## 9   8 5  8 5  2  off     spacer
    ## 10  5 4  5 4 10  off     xlab-t
    ## 11  9 4  9 4 11  off     xlab-b
    ## 12  7 2  7 2 12  off     ylab-l
    ## 13  7 6  7 6 13  off     ylab-r
    ## 14  7 4  7 4 14  off  guide-box
    ## 15  4 4  4 4 15  off   subtitle
    ## 16  3 4  3 4 16  off      title
    ## 17 10 4 10 4 17  off    caption

    # Now we add the density grobs
    g.scatter_grob <- gtable_add_grob(g.scatter_grob, g.density_x_grob, 
                                      t = 1, l = 4, b = 1, r = 7)
    g.scatter_grob <- gtable_add_grob(g.scatter_grob, g.density_y_grob, 
                                      t = 4, l = 8, b = 7, r = 8)

    grid.newpage()
    grid.draw(g.scatter_grob)

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-47-1.png)

Uh oh, we're a little misaligned. We can expand our x-axes to make the
density plots fill the entire space

    # We'll need "gtable" to tread the grids like a table

    g.scatter <- ggplot(diamond_sub, aes(x=carat, y=price, color=cut))+
      geom_point(size=0.5)+
      scale_color_brewer(palette="Set1", guide=guide_legend(title="Cut"))+
      scale_y_continuous(name="Price")+
      scale_x_continuous(name="Carat")+
      theme_minimal()+
      theme(
        legend.position=c(0.15,0.8),
        legend.background = element_rect(fill="#EFEFEF")
      )

    g.density_x <- ggplot(diamond_sub, aes(x=carat, y=..density.., fill=cut))+
      geom_density(position="stack")+
      scale_fill_brewer(palette="Set1")+
      scale_x_continuous(expand=c(0,0))+
      theme_void()+
      theme(
        legend.position="none"
      )

    g.density_y <- ggplot(diamond_sub, aes(x=price, y=..density.., fill=cut))+
      geom_density(position="stack")+
      scale_fill_brewer(palette="Set1")+
      scale_x_continuous(expand=c(0,0))+
      coord_flip()+ # Flip the coordinates so x is y, y i x
      theme_void()+
      theme(
        legend.position="none"
      )

    # Make our grobs
    g.scatter_grob   <- ggplotGrob(g.scatter)
    g.density_x_grob <- ggplotGrob(g.density_x)
    g.density_y_grob <- ggplotGrob(g.density_y)

    # Add a row and a column to the scatterplot where we'll add our density plots
    g.scatter_grob <- gtable_add_rows(g.scatter_grob,
                                      unit(0.1,"npc"), # npc is a unit that's defined by the viewport
                                      0) # position - we add to the top
    g.scatter_grob <- gtable_add_cols(g.scatter_grob,
                                      unit(0.1,"npc"), # npc is a unit that's defined by the viewport
                                      -1) # position - we add on the right

    # Let's check our layout
    g.scatter_grob$layout

    ##     t l  b r  z clip       name
    ## 18  2 1 11 7  0   on background
    ## 1   6 3  6 3  5  off     spacer
    ## 2   7 3  7 3  7  off     axis-l
    ## 3   8 3  8 3  3  off     spacer
    ## 4   6 4  6 4  6  off     axis-t
    ## 5   7 4  7 4  1   on      panel
    ## 6   8 4  8 4  9  off     axis-b
    ## 7   6 5  6 5  4  off     spacer
    ## 8   7 5  7 5  8  off     axis-r
    ## 9   8 5  8 5  2  off     spacer
    ## 10  5 4  5 4 10  off     xlab-t
    ## 11  9 4  9 4 11  off     xlab-b
    ## 12  7 2  7 2 12  off     ylab-l
    ## 13  7 6  7 6 13  off     ylab-r
    ## 14  7 4  7 4 14  off  guide-box
    ## 15  4 4  4 4 15  off   subtitle
    ## 16  3 4  3 4 16  off      title
    ## 17 10 4 10 4 17  off    caption

    # Now we add the density grobs
    g.scatter_grob <- gtable_add_grob(g.scatter_grob, g.density_x_grob, 
                                      t = 1, l = 4, b = 1, r = 7)
    g.scatter_grob <- gtable_add_grob(g.scatter_grob, g.density_y_grob, 
                                      t = 4, l = 8, b = 7, r = 8)

    grid.newpage()
    grid.draw(g.scatter_grob)

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-48-1.png)

Todo: adding raw grobs

LaTeX integration
-----------------

Not gonna make it here today... The gist of what I was going for here
was that using knitr/sweave with LaTeX does particularly bad with
ggplot's viewports. With explicit specification of the height and width
of our plots using gtable() we can overcome that problem.

Extensions
==========

> Cowplot

Some DoPlots To Tittilate
=========================

Here we see the plot in its native habitat, with its code fragmented
from hours of drinking at your desk while trying to figure out what the
hell a null unit is.

Segments & Border Whiskers
--------------------------

    library(grid)
    library(gtable)
    library(gridExtra)

    load("data/gendat_mouse_rs.R", .GlobalEnv)
    load("data/gendat_mouse.R", .GlobalEnv)

    # Set some colors
    scat_cols <- c("GT2_LINE"="#A54601","GT3_LINE"="#196B52")

    # Generalization Type Scatter/Regression plot
    g.genscat_scat <- ggplot(gendat.mouse.rs,aes(x=gt1)) + 
      geom_point(aes(y=gt2),col="#FF6B02",alpha=0.8,size=2) +
      geom_smooth(aes(y=gt2,colour="GT2_LINE"),method=lm,se=TRUE,size=0.75) +
      geom_point(aes(y=gt3),col="#1b9e77",alpha=0.8,size=2) +
      geom_smooth(aes(y=gt3,colour="GT3_LINE"),method=lm,se=TRUE,size=0.75) +
      scale_x_continuous(expand = c(0, 0),
                         breaks=c(0.65,0.7,0.75,0.8,0.85),labels=c("65","70","75","80","85")) + 
      scale_y_continuous(expand = c(0, 0),
                         breaks=c(0.5,0.55,0.6,0.65,0.7,0.75),labels=c("50","55","60","65","70","75")) + 
      expand_limits(x=c(0.62,0.86),y=c(0.5,0.80))+
      ylab("Generalization Accuracy (%)") +
      xlab("Learned Token Accuracy (%)") +
      scale_colour_manual(values=scat_cols,
                          labels=c(expression(paste("Type 2 -  ",beta,": 0.52,  ", R^{2},":0.70")), 
                                   expression(paste("Type 3 -  ",beta,": 0.65,  ", R^{2},":0.58"))))+
      guides(colour=guide_legend(override.aes=list(size=5)))+
      theme(panel.background  = element_rect(fill="#F5F5F5"), 
            panel.grid.major  = element_blank(),
            panel.grid.minor  = element_blank(),
            plot.background   = element_rect(fill="transparent",colour=NA),
            axis.title.y      = element_text(size=rel(1),margin=margin(2,7,2,10,unit="pt")),
            axis.text.y       = element_text(size=rel(1),margin=margin(1,3,1,1,unit="pt")),
            axis.text.x       = element_text(size=rel(1),margin=margin(7,1,1,1,unit="pt")),
            axis.title.x      = element_text(size=rel(1),margin=margin(7,2,10,2,unit="pt")),
            legend.position   = c(.7,.2),
            legend.text       = element_text(size=rel(.8)),
            legend.title      = element_blank(),
            legend.background = element_blank(),
            legend.key        = element_blank(),
            plot.margin       = unit(c(0,0,0,0),"lines"))

    g.genscat_boxhorz <- ggplot(gendat.mouse.rs,aes(x=factor(1),y=gt1))+
      geom_boxplot()+
      geom_point(size=1) + 
      scale_y_continuous(expand = c(0, 0)) + 
      #expand_limits(y=c(0.62,0.86))+
      coord_flip()+
      theme(panel.background = element_blank(), 
            panel.grid.major = element_blank(),
            panel.grid.minor = element_blank(),
            plot.background  = element_rect(fill="transparent",colour=NA),
            axis.title.y     = element_blank(),
            axis.text.y      = element_blank(),
            axis.title.x     = element_blank(),
            axis.text.x      = element_blank(),
            axis.ticks.x     = element_blank(),
            axis.ticks.y     = element_blank(),
            legend.position  = "none",
            plot.margin      = unit(c(1,0,0,0),"lines"))

    g.genscat_boxvert <- ggplot(gendat.mouse.rs,aes(x=factor(1))) +
      geom_boxplot(aes(y=gt2),col="#A54601") +
      geom_point(aes(y=gt2),col="#FF6B02",size=1) +
      geom_boxplot(aes(x=factor(2),y=gt3),col="#196B52") +
      geom_point(aes(x=factor(2),y=gt3),col="#1b9e77",size=1) +
      scale_y_continuous(expand = c(0, 0)) + 
      expand_limits(y=c(0.45,0.80))+
      theme(panel.background = element_blank(), 
            panel.grid.major = element_blank(),
            panel.grid.minor = element_blank(),
            plot.background  = element_rect(fill="transparent",colour=NA),
            axis.title.y     = element_blank(),
            axis.text.y      = element_blank(),
            axis.title.x     = element_blank(),
            axis.text.x      = element_blank(),
            axis.ticks.x     = element_blank(),
            axis.ticks.y     = element_blank(),
            legend.position  = "none",
            plot.margin = unit(c(0,0,0,0),"lines"))

    # Combine plots, see https://www.r-bloggers.com/scatterplot-with-marginal-boxplots/
    g.genscat_gtabscat <- ggplotGrob(g.genscat_scat)
    g.genscat_gtabhorz <- ggplotGrob(g.genscat_boxhorz)
    g.genscat_gtabvert <- ggplotGrob(g.genscat_boxvert)

    maxW <- unit.pmax(g.genscat_gtabscat$widths[2:3],g.genscat_gtabhorz$widths[2:3])
    maxH <- unit.pmax(g.genscat_gtabscat$heights[4:5],g.genscat_gtabvert$heights[4:5])
    g.genscat_gtabscat$widths[2:3] <- as.list(maxW)
    g.genscat_gtabhorz$widths[2:3] <- as.list(maxW)
    g.genscat_gtabscat$heights[4:5] <- as.list(maxH)
    g.genscat_gtabvert$heights[4:5] <- as.list(maxH)

    g.genscat_gtab <- gtable(widths=unit(c(7,1),"null"),height=unit(c(1,7),"null"))
    g.genscat_gtab <- gtable_add_grob(g.genscat_gtab,g.genscat_gtabscat,2,1)
    g.genscat_gtab <- gtable_add_grob(g.genscat_gtab,g.genscat_gtabhorz,1,1)
    g.genscat_gtab <- gtable_add_grob(g.genscat_gtab,g.genscat_gtabvert,2,2)

    # Generalization type scatter/line combined plot
    g.gensl <- ggplot(gendat.mouse,aes(x=as.factor(gentype),y=meancx,group=as.factor(mouse)))+
      geom_point(size=1)+
      geom_line(size=0.3)+
      xlab("Generalization Type")+
      ylab("Mean Accuracy (%)")+
      scale_x_discrete(expand=c(0.1,0.1))+
      scale_y_continuous(expand=c(0,0),
                         limits=c(0.5,0.86),
                         breaks=c(0.5,0.55,0.6,0.65,0.7,0.75,0.8,0.85),
                         labels=c("50","55","60","65","70","75","80","85"))+
      theme(panel.background = element_blank(), 
            panel.grid.major = element_blank(),
            panel.grid.minor = element_blank(),
            axis.text.x      = element_text(size=rel(1),margin=margin(7,1,1,1,unit="pt")),
            axis.title.x     = element_text(size=rel(1),margin=margin(7,2,10,2,unit="pt")),
            plot.background  = element_rect(fill="transparent",colour=NA),
            legend.position  = "none",
            plot.margin      = unit(c(1,0,0,0),"lines"))

    #Combination plot of above two
    g.gensl.gt <- ggplotGrob(g.gensl)

    ## Warning: Removed 1 rows containing missing values (geom_point).

    ## Warning: Removed 1 rows containing missing values (geom_path).

    g.combo <- grid.arrange(g.gensl.gt,g.genscat_gtab,widths=c(2,5))

    grid.newpage()
    grid.draw(g.combo)

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-49-1.png)

Heatmap & PCs
-------------

    load("data/gendat_vowmus.R", .GlobalEnv)
    load("data/klust_wseg.R", .GlobalEnv)
    load("data/pmc_melt.R", .GlobalEnv)
    load("data/pmc_x.R", .GlobalEnv)

      # Make dendrogram
      g.dendro <- ggplot(klust.w_seg)+
        geom_segment(aes(x=x,y=y,xend=xend,yend=yend,color=cols))+
        scale_color_manual(values=c("#000000","#E21D1D","#3B4aEF","#FC7A00"))+
        scale_y_reverse(expand=c(0,0))+
        scale_x_discrete(expand=c(0.04,0))+
        theme_void()+
        theme(legend.position = "none",
              panel.background  = element_blank(), 
              panel.grid.major  = element_blank(),
              panel.grid.minor  = element_blank(),
              plot.background   = element_blank(),
              plot.margin       = margin(0,0,0,0,unit="npc"),
              axis.text         = element_blank(),
              axis.title.x      = element_blank(),
              axis.title.y      = element_blank())
        
      
      # Split into /g/ and /b/
      gendat.vowmus_g <- gendat.vowmus[gendat.vowmus$consonant==1,] 
      gendat.vowmus_b <- gendat.vowmus[gendat.vowmus$consonant==2,]
      
      # Factor & reorder so the vowels are in the right order...
      gendat.vowmus_g$vowel <- factor(gendat.vowmus_g$vowel,ordered=TRUE)
      gendat.vowmus_g$vowel <- factor(gendat.vowmus_g$vowel,levels(gendat.vowmus_g$vowel)[c(6,5,4,3,2,1)])
      gendat.vowmus_b$vowel <- factor(gendat.vowmus_b$vowel,ordered=TRUE)
      gendat.vowmus_b$vowel <- factor(gendat.vowmus_b$vowel,levels(gendat.vowmus_b$vowel)[c(6,5,4,3,2,1)])
      
      # Split (& set ID's separately so the x-axis is right)
      pmc.melt_g <- pmc.melt[pmc.melt$consonant==1,]
      pmc.melt_g$ID <- factor(paste(pmc.melt_g$consonant,pmc.melt_g$speaker,pmc.melt_g$vowel),ordered=TRUE)
      pmc.melt_b <- pmc.melt[pmc.melt$consonant==2,]
      pmc.melt_b$ID <- factor(paste(pmc.melt_b$consonant,pmc.melt_b$speaker,pmc.melt_b$vowel),ordered=TRUE)
      
      
      g.tok_g <- ggplot(gendat.vowmus_g,aes(x=mouse,y=vowel,fill=meancx)) + 
        geom_tile() + 
        ylab("Vowel") + 
        scale_y_discrete(labels=c("/u/","/e/","/ae/","/a/","/o/","/I/"),
                         position="right") + # reverse order to make the plot work
        scale_fill_distiller(type   = "div", palette="RdGy", limits=c(0,1),
                             breaks = c(0,0.5,1), labels=c("0%","50%","100%"),
                             name   = "Accuracy")+
        facet_grid(speaker ~ .,scales="free_y",space="free_y") +
        theme(panel.spacing     = unit(0,"npc"),
              panel.background  = element_blank(), 
              panel.grid.major  = element_blank(),
              panel.grid.minor  = element_blank(),
              plot.background   = element_blank(),
              axis.line         = element_blank(),
              axis.title.x      = element_blank(),
              axis.text.x       = element_blank(),
              axis.ticks.x      = element_blank(),
              axis.title.y      = element_blank(),
              axis.text.y       = element_text(size=rel(0.5)),
              legend.position   = "none",
              legend.text       = element_text(size=rel(0.5)),
              legend.title      = element_blank(),
              legend.key.height = unit(0.01,"npc"),
              legend.background = element_blank(),
              strip.text        = element_text(size=rel(0.6)),
              plot.margin       = margin(0.04,0,0,0,unit="npc"))
      
      g.tok_b <- ggplot(gendat.vowmus_b,aes(x=mouse,y=vowel,fill=meancx)) + 
        geom_tile() + 
        ylab("Vowel") + 
        scale_y_discrete(labels=c("/u/","/e/","/ae/","/a/","/o/","/I/"),
                         position="right") +
        scale_fill_distiller(type="div", palette="RdGy", limits=c(0,1))+
        xlab("Mouse") +
        facet_grid(speaker ~ .,scales="free_y",space="free_y") +
        theme(panel.spacing.y  = unit(0,"npc"),
              panel.spacing.x  = unit(0,"npc"),
              panel.background = element_blank(), 
              panel.grid.major = element_blank(),
              panel.grid.minor = element_blank(),
              plot.background  = element_blank(),
              axis.line        = element_blank(),
              axis.ticks.x     = element_blank(),
              axis.text.x      = element_blank(),
              axis.title.y     = element_blank(),
              axis.text.y      = element_text(size=rel(0.5)),
              axis.title.x     = element_blank(),
              legend.position  = "none",
              strip.text       = element_text(size=rel(0.6)),
              plot.margin      = margin(0,0,0.04,0,unit="npc"))
      
      
      # Add panels on right for consonants
      g.tok_gtab <- ggplotGrob(g.tok_g)
      g.tok_gtab <- gtable_add_cols(g.tok_gtab, unit(g.tok_gtab$widths[[5]],'cm'), 7)
      g.tok_gtab <- gtable_add_grob(g.tok_gtab,
                                    rectGrob(gp=gpar(col=NA, fill=gray(0.5))),
                                    6,8,14, name="glab")
      g.tok_gtab <- gtable_add_grob(g.tok_gtab,
                                    textGrob("/g/", rot=-90, gp=gpar(col=gray(1))),
                                    6,8,14, name="gtext")
      
      g.tok_btab <- ggplotGrob(g.tok_b) 
      
      # Add dendrogram first
      g.dend_tab <- ggplotGrob(g.dendro)
      g.dend_tab$heights[8] <- unit(0,"cm")
      g.dend_tab$widths[3] <- unit(0,"cm")
      
      g.tok_btab <- gtable_add_rows(g.tok_btab, unit(0.3, "inches"), 17)
      g.tok_btab <- gtable_add_grob(g.tok_btab, g.dend_tab,
                                    17,4,19,4,name="dendro")
      
      g.tok_btab <- gtable_add_cols(g.tok_btab, unit(g.tok_btab$widths[[5]], 'cm'),7)
      g.tok_btab <- gtable_add_grob(g.tok_btab,
                                    rectGrob(gp=gpar(col=NA, fill=gray(0.5))),
                                    4,8,14, name="blab")
      g.tok_btab <- gtable_add_grob(g.tok_btab,
                                    textGrob("/b/",rot=-90,gp=gpar(col=gray(1))),
                                    4,8,14,name="btext")
      
      ###### PC plots
      # Calculate limits for bar
      pc_lim <- max(abs(pmc.melt$mean.val))
      pc_lim <- c(-pc_lim,pc_lim)
      
      g.pcw_g <- ggplot(data=pmc.melt_g,aes(x=ID,y=mean.val,fill=as.factor(speaker)))+
        scale_fill_manual(values=c("#000000","#757575","#000000","#757575","#000000"))+
        geom_bar(stat='identity')+
        coord_flip()+
        scale_x_discrete(limits=rev(levels(pmc.melt_g$ID)))+
        scale_y_continuous(limits=pc_lim)+
        facet_wrap(~X2)+
        theme(
          panel.background = element_blank(), 
          panel.grid.major = element_blank(),
          panel.grid.minor = element_blank(),
          plot.background  = element_blank(),
          axis.text        = element_blank(),
          axis.title       = element_blank(),
          axis.line        = element_blank(),
          axis.ticks       = element_blank(),
          strip.text       = element_blank(),
          strip.background = element_blank(),
          plot.margin      = margin(0,0,0,0,unit="npc"),
          panel.spacing.y  = unit(0,"npc"),
          legend.position  = "none"
        )
      
      g.pcw_b <- ggplot(data=pmc.melt_b,aes(x=ID,y=mean.val,fill=as.factor(speaker)))+
        scale_fill_manual(values=c("#000000","#757575","#000000","#757575","#000000"))+
        geom_bar(stat='identity')+
        coord_flip()+
        scale_x_discrete(limits=rev(levels(pmc.melt_b$ID)))+
        scale_y_continuous(limits=pc_lim)+
        facet_wrap(~X2)+
        theme(
          panel.background = element_blank(), 
          panel.grid.major = element_blank(),
          panel.grid.minor = element_blank(),
          plot.background  = element_blank(),
          axis.text        = element_blank(),
          legend.position  = "none",
          axis.title       = element_blank(),
          axis.line        = element_blank(),
          axis.ticks       = element_blank(),
          strip.text       = element_blank(),
          strip.background = element_blank(),
          plot.margin      = margin(0,0,0,0,unit="npc"),
          panel.spacing.y  = unit(0,"npc")
        )
      
      # Flip values so they align w/ heatmap orientation
      pmc.x$genbias <- pmc.x$genbias * -1
      
      ###### Now the scatter/line plots of PC loadings vs. bias/accuracy.
      g.pc1 <- ggplot(data=pmc.x,aes(x=PC1,y=genbias))+
        geom_point(aes(shape=cohort))+
        scale_shape_manual(values=c(16,1))+
        geom_smooth(method=lm,se=FALSE,size=0.75,color="black")+
        labs(x="PC1 Loading",y="Bias")+
        scale_y_continuous(breaks=c(-0.2,-0.1,0,0.1),labels=c("+20% /g/","+10% /g/","0%","+10% /b/"))+
        theme(
          panel.background = element_blank(), 
          panel.grid.major = element_blank(),
          panel.grid.minor = element_blank(),
          plot.background  = element_blank(),
          legend.position  = "none",
          axis.line        = element_blank(),
          plot.margin      = margin(0,0,1,3,unit="pt"),
          panel.spacing.y  = unit(0,"npc")
        )
      
      g.pc2 <- ggplot(data=pmc.x,aes(x=PC2,y=meancx))+
        geom_point(aes(shape=cohort))+
        scale_shape_manual(values=c(16,1))+
        geom_smooth(method=lm,se=FALSE,size=0.75,color="black")+
        labs(x="PC2 Loading",y="% Correct")+
        scale_y_continuous(limits=c(0.60,0.85),breaks=c(0.65,0.75,0.85),labels=c("65%","75%","85%"))+
        theme(
          panel.background = element_blank(), 
          panel.grid.major = element_blank(),
          panel.grid.minor = element_blank(),
          plot.background  = element_blank(),
          legend.position  = "none",
          axis.line        = element_blank(),
          plot.margin      = margin(1,0,0,3,unit="pt"),
          panel.spacing.y  = unit(0,"npc")
        )
      
      ## Combine heatmap and PCplots
      # Get pc grobs
      g.pcw_gtab <- ggplotGrob(g.pcw_g)
      g.pcw_btab <- ggplotGrob(g.pcw_b)
      g.pc1_tab  <- ggplotGrob(g.pc1)
      g.pc2_tab  <- ggplotGrob(g.pc2)
      
      # Add columns to heatmap grids
      g.tok_gtab <- gtable_add_cols(g.tok_gtab,rep(unit(0.2,"npc"),2),0)
      g.tok_btab <- gtable_add_cols(g.tok_btab,rep(unit(0.2,"npc"),2),0)
      
      # Add PC plots to heatmaps
      g.tok_gtab <- gtable_add_grob(g.tok_gtab,
                                    g.pcw_gtab,
                                    6,1,17,2,name="gtok")
      g.tok_btab <- gtable_add_grob(g.tok_btab,
                                    g.pcw_btab,
                                    4,1,15,2,name="gtok")
      
      # Add labels above PC plots
      g.tok_gtab <- gtable_add_rows(g.tok_gtab, unit(0.2,"inches"), 4)
      g.tok_gtab <- gtable_add_grob(g.tok_gtab,
                                    rectGrob(gp=gpar(col=NA,fill=gray(0.5))),
                                    5,1,5,name="pc1box")
      g.tok_gtab <- gtable_add_grob(g.tok_gtab,
                                    rectGrob(gp=gpar(col=NA,fill=gray(0.5))),
                                    5,2,5,name="pc2box")
      g.tok_gtab <- gtable_add_grob(g.tok_gtab,
                                    textGrob("PC1",gp=gpar(col=gray(1))),
                                    5,1,5,name="pc1")
      g.tok_gtab <- gtable_add_grob(g.tok_gtab,
                                    textGrob("PC2",gp=gpar(col=gray(1))),
                                    5,2,5,name="pc2")
      
      # Add scale bar
      colvals <- data.frame(xvals=seq(0,1,0.01),yvals=as.factor(rep(0,101)))
      g.scale <- ggplot(data=colvals,aes(x=xvals,y=yvals,fill=xvals))+
        scale_fill_distiller(type="div", palette="RdGy", limits=c(0,1))+
        geom_raster(interpolate=TRUE)+
        scale_x_continuous(expand=c(0,0))+
        annotate("text", x=0.1, y=1, label = "0%",colour = "white", size=3.5)+
        annotate("text", x=0.85, y=1, label = "100%", colour= "white", size = 3.5)+
        theme_void()+
        theme(legend.position="none",
              plot.margin = margin(0,0,0,0,unit="npc"))
      g.scalegrob <- ggplotGrob(g.scale)
      g.scalegrob$widths[3] <- unit(0,"npc")
      g.scalegrob$heights[7] <- unit(0,"npc")
      
      g.tok_gtab <- gtable_add_grob(g.tok_gtab,
                                    g.scalegrob,
                                    5,6,5,6,name="scale")
      
      # Combine/Align scatterplots
      g.pc2_tab$widths <- g.pc1_tab$widths
      g.pcs <- gtable(widths=unit(c(2),"null"),height=unit(c(1,1),"null"))
      g.pcs <- gtable_add_grob(g.pcs,g.pc1_tab,1,1)
      g.pcs <- gtable_add_grob(g.pcs,g.pc2_tab,2,1)
      
      g.tokpc <- arrangeGrob(g.tok_gtab,g.tok_btab,g.pcs,ncol=1,widths=unit(3.6,"inches"),heights=unit(c(2.45,2.55,2.5),"inches"))
      
      grid.newpage()
      grid.draw(g.tokpc)

![](ggplot_guide_files/figure-markdown_strict/unnamed-chunk-50-1.png)
