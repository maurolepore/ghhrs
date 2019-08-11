---
output:
  html_document:
    keep_md: yes
resource_files:
- README.html
---

<!-- README.md is generated from README.Rmd. Please edit that file -->



# My work on r2dii packages

Adapted from https://rpubs.com/hadley/gh.

On July 1, 2019 I started working on an ecosystem of R packages named "r2dii". Here I explore my work so far.


```r
library(gh)
library(purrr)
library(tibble)
library(dplyr)
#> 
#> Attaching package: 'dplyr'
#> The following objects are masked from 'package:stats':
#> 
#>     filter, lag
#> The following objects are masked from 'package:base':
#> 
#>     intersect, setdiff, setequal, union
library(readr)
library(lubridate)
#> 
#> Attaching package: 'lubridate'
#> The following object is masked from 'package:base':
#> 
#>     date
library(ggplot2)
library(forcats)
```

I start by getting a list of the 50 repos that I’ve touched most recently.


```r
my_repos <- function(type = c("all", "owner", "public", "private", "member"), 
                     limit = 50) {
  type <- match.arg(type)
  
  gh(
    "GET /user/repos",
    type = type, 
    sort = "updated",
    .limit = limit
  )
}
repos <- my_repos("owner", limit = 50)
length(repos)
#> [1] 50

full_name <- repos %>% map_chr("full_name")
head(full_name, 20)
#>  [1] "maurolepore/ghhrs"                
#>  [2] "maurolepore/r2dii.data"           
#>  [3] "maurolepore/r2dii.dataprep"       
#>  [4] "maurolepore/r2dii.dataraw"        
#>  [5] "maurolepore/junr"                 
#>  [6] "maurolepore/DCC"                  
#>  [7] "maurolepore/drake"                
#>  [8] "maurolepore/drake-manual"         
#>  [9] "maurolepore/covr"                 
#> [10] "maurolepore/maurolepore.github.io"
#> [11] "maurolepore/todo"                 
#> [12] "maurolepore/r2dii.usethis"        
#> [13] "maurolepore/meetings"             
#> [14] "maurolepore/cv"                   
#> [15] "maurolepore/flagr"                
#> [16] "maurolepore/GitHistoryTracker"    
#> [17] "maurolepore/ghactions"            
#> [18] "maurolepore/github-demo"          
#> [19] "maurolepore/compareWith"          
#> [20] "maurolepore/praise"
```

(If you’re doing this yourself, you’ll need to make sure you’ve set up an environment variable GITHUB_PAT with a GitHub personal access token.)

And then, for each repo, I get all the commits since 2019-07-01, around the time I became a full time software developer. I collaborate with other people, so I make sure to extract the author of the commit.


```r
repo_commits <- function(full_name, since = "2019-07-01") {
  message("Requesting commits for ", full_name)
  
  commits <- gh("GET /repos/:full_name/commits", 
    full_name = full_name, 
    since = since,
    .limit = Inf
  )
  
  if (length(commits) == 0) {
    return(NULL)
  }
  
  tibble(
    full_name = full_name,
    author = commits %>% map_chr(c("author", "login"), .null = NA_character_),
    datetime = commits %>% map_chr(c("commit", "author", "date"), .null = NA_character_)
  )
}

commits <- full_name %>% map(repo_commits) %>% compact() %>% bind_rows()
#> Requesting commits for maurolepore/ghhrs
#> Requesting commits for maurolepore/r2dii.data
#> Requesting commits for maurolepore/r2dii.dataprep
#> Requesting commits for maurolepore/r2dii.dataraw
#> Requesting commits for maurolepore/junr
#> Requesting commits for maurolepore/DCC
#> Requesting commits for maurolepore/drake
#> Requesting commits for maurolepore/drake-manual
#> Requesting commits for maurolepore/covr
#> Requesting commits for maurolepore/maurolepore.github.io
#> Requesting commits for maurolepore/todo
#> Requesting commits for maurolepore/r2dii.usethis
#> Requesting commits for maurolepore/meetings
#> Requesting commits for maurolepore/cv
#> Requesting commits for maurolepore/flagr
#> Requesting commits for maurolepore/GitHistoryTracker
#> Requesting commits for maurolepore/ghactions
#> Requesting commits for maurolepore/github-demo
#> Requesting commits for maurolepore/compareWith
#> Requesting commits for maurolepore/praise
#> Requesting commits for maurolepore/a-repo
#> Requesting commits for maurolepore/rocker
#> Requesting commits for maurolepore/meetups
#> Requesting commits for maurolepore/gh4projects
#> Requesting commits for maurolepore/what-they-forgot
#> Requesting commits for maurolepore/confs
#> Requesting commits for maurolepore/pkgdoc
#> Requesting commits for maurolepore/fs
#> Requesting commits for maurolepore/gitignore
#> Requesting commits for maurolepore/fgeo.install
#> Requesting commits for maurolepore/drat
#> Requesting commits for maurolepore/tor
#> Requesting commits for maurolepore/rodev
#> Requesting commits for maurolepore/project
#> Requesting commits for maurolepore/fgeo.krig
#> Requesting commits for maurolepore/fgeo.misc
#> Requesting commits for maurolepore/fgeo
#> Requesting commits for maurolepore/dotfiles
#> Requesting commits for maurolepore/temp.gh
#> Requesting commits for maurolepore/fgeo.plot
#> Requesting commits for maurolepore/git-comun
#> Requesting commits for maurolepore/slack
#> Requesting commits for maurolepore/testthat
#> Requesting commits for maurolepore/commit
#> Requesting commits for maurolepore/quienes-somos
#> Requesting commits for maurolepore/un-repositorio
#> Requesting commits for maurolepore/revdepcheck
#> Requesting commits for maurolepore/gmailr
#> Requesting commits for maurolepore/dev_guide
#> Requesting commits for maurolepore/ixplorer
commits
#> # A tibble: 1,229 x 3
#>    full_name              author      datetime            
#>    <chr>                  <chr>       <chr>               
#>  1 maurolepore/ghhrs      maurolepore 2019-08-11T00:27:36Z
#>  2 maurolepore/ghhrs      maurolepore 2019-08-10T23:32:58Z
#>  3 maurolepore/ghhrs      maurolepore 2019-08-10T23:01:10Z
#>  4 maurolepore/ghhrs      maurolepore 2019-08-10T21:59:43Z
#>  5 maurolepore/ghhrs      maurolepore 2019-08-10T21:30:37Z
#>  6 maurolepore/r2dii.data maurolepore 2019-07-09T12:26:56Z
#>  7 maurolepore/r2dii.data maurolepore 2019-07-09T09:43:11Z
#>  8 maurolepore/r2dii.data maurolepore 2019-07-09T09:23:20Z
#>  9 maurolepore/r2dii.data maurolepore 2019-07-09T09:18:51Z
#> 10 maurolepore/r2dii.data maurolepore 2019-07-09T08:27:11Z
#> # ... with 1,219 more rows
```

Next, I parse the commit date, and set my timezone. I break the datetime into separate date and time pieces as that will make plotting easier later on.


```r
commits <- commits %>% mutate(
  datetime = lubridate::with_tz(readr::parse_datetime(datetime), "America/Chicago"),
  date = floor_date(datetime, "day"),
  time = update(datetime, yday = lubridate::yday("2019-08-01"))
)
#> Warning in (function (object, years = integer(), months = integer(), days =
#> integer(), : partial argument match of 'yday' to 'ydays'
commits
#> # A tibble: 1,229 x 5
#>    full_name author datetime            date               
#>    <chr>     <chr>  <dttm>              <dttm>             
#>  1 maurolep~ mauro~ 2019-08-10 19:27:36 2019-08-10 00:00:00
#>  2 maurolep~ mauro~ 2019-08-10 18:32:58 2019-08-10 00:00:00
#>  3 maurolep~ mauro~ 2019-08-10 18:01:10 2019-08-10 00:00:00
#>  4 maurolep~ mauro~ 2019-08-10 16:59:43 2019-08-10 00:00:00
#>  5 maurolep~ mauro~ 2019-08-10 16:30:37 2019-08-10 00:00:00
#>  6 maurolep~ mauro~ 2019-07-09 07:26:56 2019-07-09 00:00:00
#>  7 maurolep~ mauro~ 2019-07-09 04:43:11 2019-07-09 00:00:00
#>  8 maurolep~ mauro~ 2019-07-09 04:23:20 2019-07-09 00:00:00
#>  9 maurolep~ mauro~ 2019-07-09 04:18:51 2019-07-09 00:00:00
#> 10 maurolep~ mauro~ 2019-07-09 03:27:11 2019-07-09 00:00:00
#> # ... with 1,219 more rows, and 1 more variable: time <dttm>
```

Next, I do a couple of quick checks to make sure the data looks reasonable.


```r
commits %>% count(full_name, sort = TRUE) %>% print(n = 20)
#> # A tibble: 50 x 2
#>    full_name                             n
#>    <chr>                             <int>
#>  1 maurolepore/r2dii.dataraw           661
#>  2 maurolepore/drake                   263
#>  3 maurolepore/ghactions                38
#>  4 maurolepore/DCC                      33
#>  5 maurolepore/drake-manual             31
#>  6 maurolepore/github-demo              28
#>  7 maurolepore/r2dii.usethis            25
#>  8 maurolepore/covr                     23
#>  9 maurolepore/junr                     20
#> 10 maurolepore/r2dii.data               20
#> 11 maurolepore/meetings                 17
#> 12 maurolepore/r2dii.dataprep           14
#> 13 maurolepore/ghhrs                     5
#> 14 maurolepore/maurolepore.github.io     5
#> 15 maurolepore/todo                      5
#> 16 maurolepore/compareWith               4
#> 17 maurolepore/flagr                     3
#> 18 maurolepore/cv                        2
#> 19 maurolepore/a-repo                    1
#> 20 maurolepore/commit                    1
#> # ... with 30 more rows

commits %>% count(author, sort = TRUE)
#> # A tibble: 22 x 2
#>    author            n
#>    <chr>         <int>
#>  1 maurolepore     771
#>  2 wlandau-lilly   157
#>  3 wlandau         130
#>  4 maxheld83        38
#>  5 ronnyhdez        27
#>  6 <NA>             26
#>  7 jimhester        21
#>  8 2diiKlaus        17
#>  9 Clare2D          16
#> 10 ecamo19           5
#> # ... with 12 more rows
```

Now I pull out just the commits that I made, and only for repositories matching "r2dii".


```r
mauro <- commits %>% 
  filter(author == "maurolepore", grepl("r2dii", full_name))

mauro
#> # A tibble: 685 x 5
#>    full_name author datetime            date               
#>    <chr>     <chr>  <dttm>              <dttm>             
#>  1 maurolep~ mauro~ 2019-07-09 07:26:56 2019-07-09 00:00:00
#>  2 maurolep~ mauro~ 2019-07-09 04:43:11 2019-07-09 00:00:00
#>  3 maurolep~ mauro~ 2019-07-09 04:23:20 2019-07-09 00:00:00
#>  4 maurolep~ mauro~ 2019-07-09 04:18:51 2019-07-09 00:00:00
#>  5 maurolep~ mauro~ 2019-07-09 03:27:11 2019-07-09 00:00:00
#>  6 maurolep~ mauro~ 2019-07-09 03:27:23 2019-07-09 00:00:00
#>  7 maurolep~ mauro~ 2019-07-09 03:10:32 2019-07-09 00:00:00
#>  8 maurolep~ mauro~ 2019-07-09 03:26:37 2019-07-09 00:00:00
#>  9 maurolep~ mauro~ 2019-07-09 03:50:16 2019-07-09 00:00:00
#> 10 maurolep~ mauro~ 2019-07-09 03:23:20 2019-07-09 00:00:00
#> # ... with 675 more rows, and 1 more variable: time <dttm>
```

To start, lets figure out what I’ve been working on. I’ll just look at the top 25 repos.


```r
mauro %>% 
  mutate(repo = full_name %>% fct_reorder(date) %>% fct_rev() %>% fct_lump(25)) %>% 
  ggplot(aes(date, repo)) + 
  geom_jitter()
#> Warning in rank(-count, ties = ties.method): partial argument match of
#> 'ties' to 'ties.method'
```

![](README_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

What times of day do I usually work on things?


```r
mauro %>% 
  ggplot(aes(date, time)) + 
  geom_jitter()
```

![](README_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

We can look at my average work week by breaking down by day of week.


```r
mauro %>% 
  mutate(wday = wday(date, label = TRUE) %>% fct_shift(1) %>% fct_rev()) %>% 
  ggplot(aes(time, wday)) + 
  geom_jitter()
```

![](README_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

How much time have I spent working on each project?


```r
hours_spent <- mauro %>% 
  group_by(date, full_name) %>% 
  arrange(datetime, full_name) %>% 
  mutate(time_spent = last(time) - first(time)) %>% 
  select(full_name, date, time_spent) %>% 
  unique() %>% 
  ungroup() %>% 
  group_by(full_name) %>% 
  summarize(hours_spent = sum(time_spent, na.rm = TRUE)) %>% 
  arrange(desc(hours_spent)) %>% 
  mutate(
    hours_spent = as.double(hours_spent) / 3600,
    # Rest lunch (~1 hour every 8 hours)
    hours_spent = hours_spent - (hours_spent %/% 8)
  )
hours_spent
#> # A tibble: 4 x 2
#>   full_name                  hours_spent
#>   <chr>                            <dbl>
#> 1 maurolepore/r2dii.dataraw       175.  
#> 2 maurolepore/r2dii.usethis        12.0 
#> 3 maurolepore/r2dii.dataprep        8.09
#> 4 maurolepore/r2dii.data            5.51
```

How many expected work hours have I accumulated so far?


```r
days <- as.double(today() - date("2019-07-01"))
weekend_days <- (days %/% 7) * 2
work_days <- days - weekend_days
work_hours <- work_days * 8
work_hours
#> [1] 240
```

What percent of my time goes to r2dii packages?


```r
work_on_r2dii <- sum(hours_spent$hours_spent)
percen_work_on_r2dii <- work_on_r2dii / work_hours * 100
percen_work_on_r2dii
#> [1] 83.74965
```


