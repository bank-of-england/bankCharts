<!-- README.md is generated from README.Rmd. Please edit that file -->

boeCharts
=========

Overview
--------

`boeCharts` is an R package to help you create charts that are in-line
with [the Bank’s visual identity
standards](https://bankofengland.frontify.com/d/RPk6pMZziBFw/bank-standards),
recommended (and designed) for use in conjunction with
[ggplot2](https://ggplot2.tidyverse.org/). It contains design patterns
for approximating charts found within flagship publications, including:

-   [Bank Overground](https://www.bankofengland.co.uk/bank-overground)
-   [Monetary Policy
    Report](https://www.bankofengland.co.uk/monetary-policy-report/2019/november-2019)
    (FKA Inflation Report)
-   [Financial Stability
    Report](https://www.bankofengland.co.uk/financial-stability-report/2019/july-2019)
-   [Statistical
    Releases](https://www.bankofengland.co.uk/statistics/money-and-credit/2019/april-2019)

Installation
------------

Ensure you are installing R packages via
[Artifactory](https://binarycentral/artifactory/webapp/#/home) by
**running this once** (within an R session):

``` r
local({
  r <- list("boe-cran-remote-repo" = "https://binarycentral/artifactory/boe-cran-remote-repo/",
            "boe-cran-local-repo" = "https://binarycentral/artifactory/boe-cran-local-repo/",
            "rcran" = "http://cran.rstudio.com/"
  )
  options(repos = r)
})
```

You can then install the latest stable version of `boeCharts` from
Artifactory with:

``` r
install.packages("boeCharts")
```

You can also install the development version of `boeCharts` like so:

``` r
remotes::install_git(
  "https://almplatform/tfs/UnmanagedCollection/Shared%20Analytical%20Code/_git/boeCharts", 
  git = "external"
)
```

N.B. the development version should only be used for experimentation.
Use the latest stable version, released via Artifactory, in any
production code.

Usage
-----

### Complementing a ggplot2 worflow

`boeCharts` provides native support for use with
[ggplot2](https://ggplot2.tidyverse.org/) (a popular R graphics
library). You can use `scale_[colour|fill]_[continuous|discrete]_boe`
functions for adopting Bank colour palettes within charts generated by
`ggplot2`, and `theme_xxx` functions for applying full chart styles
supported by `boeCharts`.

#### A minimal example

First, load packages:

``` r
library(boeCharts)
#> boeCharts: Bank Chart Themes and Styles for 'ggplot2'. 
#>      For more information visit:
#>      https://almplatform/tfs/UnmanagedCollection/Shared%20Analytical%20Code/_git/boeCharts?path=%2FREADME.md
#> 
library(ggplot2)
library(palmerpenguins) # for example purposes only
```

Here is an out-of-the-box `ggplot2` chart.

``` r
# build chart
(chart <- ggplot(penguins, aes(flipper_length_mm, body_mass_g, color = species)) +
  # jittery scatter
  geom_jitter(size = 3)
)
```

![](man/figures/README-unnamed-chunk-6-1.png)

Custom themes can be applied to ggplot objects as additional “layers”.
As mentioned above, all `boeCharts` theme functions are prefixed by
`theme_` e.g. `theme_overground()` applies the [Bank
Overground](https://www.bankofengland.co.uk/bank-overground) chart
theme:

``` r
(chart <- chart +
  # add Bank Overground theme
  theme_overground()
)
```

![](man/figures/README-unnamed-chunk-7-1.png)

Custom ggplot scales help with applying Bank colour palettes. In this
case, the points can be coloured with a “vibrant” Bank colour
combination variant, via `scale_colour_discrete_boe()`, again as an
additional ggplot layer:

``` r
chart <- chart +
  # add a "vibrant" Bank colour combination
  scale_colour_discrete_boe(palette = "vibrant_c")

chart
```

![](man/figures/README-unnamed-chunk-8-1.png)

Now, switching in the [Monetary Policy
Report](https://www.bankofengland.co.uk/monetary-policy-report/2019/november-2019)
theme and pre-MPC colours. Additionally, `scale_y_continuous()` is used
to move the y-axis across to the right.

``` r
chart +
  # add MPR theme
  theme_mpr() +
  # add a "vibrant" Bank colour combination
  scale_colour_discrete_boe(palette = "pre_mpc") +
  # move the y-axis to the right
  scale_y_continuous(
    position = "right", sec.axis = dup_axis(labels = NULL)
    )
```

![](man/figures/README-unnamed-chunk-9-1.png)

#### An in-depth example

Another `ggplot2` + `boeCharts` creation, this time investigating some
more customization options, including automatic axis breaks/limits
(using `boe_breaks|limits_date|numeric()`), direct line labels (using
`position_voronoi()`) and above-plot y-axis title positioning (using
`move_ylab()`).

``` r
# create chart
chart_fang <- ggplot(data = FANG, aes(x = date, y = close, colour = symbol)) +
  # add lines + hide legend
  geom_line(show.legend = FALSE) +
  # add series labels + hide legend
  geom_text(aes(label = symbol), position = position_voronoi(), 
            family = "Calibri", show.legend = FALSE) +
  # add some chart labels
  labs(
    title = "BoE Palette Test", subtitle = "A plot for demonstration purposes",
    caption = caption_boe(source = "Investopedia"),
    y = "Closing price", x = NULL
    ) +
  # use 'highlights' palette
  scale_colour_discrete_boe(palette = "boe_highlights") +
  # add Bank Overground theme
  theme_overground() +
  # apply custom axis settings
  scale_y_continuous(
    expand = c(0, 0), breaks = boe_breaks_numeric(), 
    limits = boe_limits_numeric(), position = "right"
    ) +
  scale_x_date(
    expand = c(0, 0), labels = boe_date_labels(),
    breaks = boe_breaks_date(), limits = boe_limits_date()
    )

# re-position y axis title above plot
move_ylab(chart_fang)
```

![](man/figures/README-example-1.png)

#### Markdown theme variants

A Markdown variant of each {boeCharts} theme has been made with a `md_`
suffix, like `theme_fsr_md()`. These themes add support for rendering
text as markdown, thanks to the
[ggtext](https://wilkelab.org/ggtext/index.html) package. Here’s an
example:

``` r
fsr_chart_fang <- chart_fang +
  theme_fsr_md() +
  labs(title = "**Chart A.2** BoE Palette Test")

move_ylab(fsr_chart_fang)
```

![](man/figures/README-unnamed-chunk-10-1.png)

### Using custom fonts

`boeCharts` helps you safely import and use fonts other than the basic
fonts R already uses (with help from the `extrafont` package). These
fonts can be imported using an associated `import_` utility function.
For example, to import Calibri (used by default in some `boeCharts`
themes), you should run the `import_calibri()` function. If you want to
use fonts in PDF (or Postscript) output files, set the
`boeCharts.loadfonts` option to `TRUE`.

N.B. the font import step only needs to be run once on your machine.
Fonts will be loaded automatically when you load `boeCharts` in future
sessions.

More guides/examples
--------------------

Talks:

-   [Bank-style data viz made
    simple](http://intranet/Banknav/IML.asp?svr=BOE-DMS&db=Analytical&id=8093829&v=0)

Vignettes:

-   [Using
    boeCharts](http://collaborate/workspaces/RHelpCentre/R%20Markdown/using-boeCharts.html)
-   [Bank-standard colours with
    boeCharts](http://collaborate/workspaces/RHelpCentre/R%20Markdown/Bank-standard-colours-with-boeCharts.html)
-   [‘Last-mile’
    formatting](http://collaborate/workspaces/RHelpCentre/R%20Markdown/last-mile-formatting.html)

Cookbooks:

-   [Introduction to boeCharts - produce “Bank-ready” data visualisation
    with
    ggplot2](http://collaborate/workspaces/RHelpCentre/R%20Markdown/boeCharts_intro.html)
-   [Visualising
    Distributions](http://collaborate/workspaces/RHelpCentre/R%20Markdown/ChartsVisualisingDistributions.html)
-   [Other charts from SPERI
    speech](http://collaborate/workspaces/RHelpCentre/R%20Markdown/ChartsOthers.html)

How to contribute
-----------------

This is an ongoing project to abstract and consolidate Bank charting
guidelines, and any/all contributions are encouraged. Inspecting the
source code in this repository will help with understanding how to make
a custom palette, or building a chart theme for your business area. The
following resources are also helpful (and inspired much of this effort):

-   [Modify components of a
    theme](https://ggplot2.tidyverse.org/reference/theme.html)
-   [govstyle](https://github.com/ukgovdatascience/govstyle)
-   [hrbrthemes](https://github.com/hrbrmstr/hrbrthemes)

Also, for general consideration when contributing to the package, try
to:

-   Observe principles in Hadley Wickham’s [R Packages
    book](https://r-pkgs.org/)
-   Follow the Git branch (see
    [here](https://blog.rstudio.com/2017/09/13/rstudio-v1.1-the-little-things/)
    for how to create a branch from within R Studio) & [pull
    request](https://docs.microsoft.com/en-us/azure/devops/repos/git/pullrequest?view=azure-devops)
    model, or [contact Ewen](mailto:ewen.henderson@bankofengland.co.uk)

### To do

-   Create a `theme_fsr()`
-   Chart export destination helpers (e.g. 16x9 ppt half size)
-   More extensive “using grey” examples, poss a vignette/cookbook
    demonstrating `gghighlight`
