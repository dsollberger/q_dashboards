# Interactive Dashboards

In this blog post, I want to outline my upcoming Princeton Wintersession workshop *Interactive Dashboards for Collaborative Communication*.  In this workshop, we will explore the new technology called [Quarto dashboards](https://quarto.org/docs/dashboards/). In this exploration, I have chosen a different data set per my interest, but much of the credit for the design of this workshop goes to the folks at Posit and other data scientists who have already created wonderful tutorials (see Additional Resources).

## Dashboards

* [Examples](https://quarto.org/docs/dashboards/#examples) from Posit

## Why R?

```{r}
#| message: false
#| warning: false

library("bslib")
library("bsicons")
library("sf")
library("tidyverse")

bee_data <- readr::read_csv("bee_data.csv")
states_shp <- readr::read_rds("us_states_shp.rds")

color_1 <- "#E77500"
color_2 <- "#121212"
```


## Why Quarto?

## Posit Cloud

# Case Study: Bees!

## Concern for Bee Populations

:::: {.columns}

::: {.column width="45%"}
![Time](bee_news_1.png)
:::

::: {.column width="10%"}
	
:::

::: {.column width="45%"}
![ABC](bee_news_2.png)
:::

::::

## Data Source

Our data come from the Quick Stats at [USDA NASS](https://www.nass.usda.gov/) (United States Department of Agriculture National Agricultural Statistics Service).  Users can access a subset of the data through dropdown menus.  For replicability, I list my selections below:

![USDA NASS Quick Stats](bee_search_criteria.png)

* **Commodity:**

    * **Program:** SURVEY
    * **Sector:** ANIMALS & PRODUCTS
    * **Group:** SPECIALTY
    * **Commodity:** HONEY
    * **Category:** INVENTORY
    * **Data item:** HONEY, BEE COLONIES - INVENTORY, MEASURED IN COLONIES
    
* **Location:**

    * **Geographic Level:** STATE
    * **State:** [leave unselected for all states]
    
* **Time:**

    * **Year:** [selected 2014 to 2024 to obtain about a decade of data]
    * **Period Type:** ANNUAL (note: the "annual" measurements were done by each state, while other granularity might not be available for all states)
    * **Period:** MARKETING YEAR

## Bar Charts

:::: {.panel-tabset}

## Plot

```{r}
#| echo: false
#| eval: true
#| message: false
#| warning: false

bee_data |> 
  filter(Year >= 2015 & Year <= 2019) |>
  group_by(Year) |>
  mutate(total_colonies = sum(Value)) |>
  ungroup() |>
  select(Year, total_colonies) |>
  distinct() |>
  mutate(num_colonies = total_colonies / 1e6) |>
  ggplot(aes(x = factor(Year), y = num_colonies)) +
  geom_bar(color = "#121212", fill = "#E77500",
           stat = "identity") +
  coord_cartesian(ylim = c(2.5,3.0)) +
  labs(title = "US Bee Population",
       subtitle = "Survey of bee colonies",
       caption = "source: USDA NASS",
       x = "year",
       y = "bee colonies (in millions)") +
  theme_minimal()
```

## Code

```{r}
#| echo: true
#| eval: false
#| message: false
#| warning: false

bee_data |> 
  group_by(Year) |>
  mutate(total_colonies = sum(Value)) |>
  ungroup() |>
  select(Year, total_colonies) |>
  distinct() |>
  mutate(num_colonies = total_colonies / 1e6) |>
  ggplot(aes(x = factor(Year), y = num_colonies)) +
  geom_bar(color = "#121212", fill = "#E77500",
           stat = "identity") +
  coord_cartesian(ylim = c(2.5,3.0)) +
  labs(title = "US Bee Population",
       subtitle = "Survey of bee colonies",
       # caption = "source: USDA NASS",
       x = "year",
       y = "bee colonies (in millions)") +
  theme_minimal()
```

## Each State

```{r}
#| echo: false
#| eval: true
#| message: false
#| warning: false

nj_plot <- bee_data |> 
  filter(State == "NEW JERSEY") |>
  mutate(num_colonies = Value / 1e3) |>
  ggplot(aes(x = factor(Year), y = num_colonies)) +
  geom_bar(color = "#121212", fill = "#E77500",
           stat = "identity") +
  labs(title = "NEW JERSEY",
       subtitle = "Survey of bee colonies",
       caption = "source: USDA NASS",
       x = "year",
       y = "bee colonies (in thousands)") +
  theme_minimal()

nj_plot
```

## Code

```{r}
#| echo: true
#| eval: false
#| message: false
#| warning: false

nj_plot <- bee_data |> 
  filter(State == "NEW JERSEY") |>
  mutate(num_colonies = Value / 1e3) |>
  ggplot(aes(x = factor(Year), y = num_colonies)) +
  geom_bar(color = "#121212", fill = "#E77500",
           stat = "identity") +
  labs(title = "NEW JERSEY",
       subtitle = "Survey of bee colonies",
       caption = "source: USDA NASS",
       x = "year",
       y = "bee colonies (in thousands)") +
  theme_minimal()

nj_plot
```

## Facets

```{r}
#| echo: false
#| eval: true
#| message: false
#| warning: false

bee_data |>
  filter(Year >= 2015 & Year <= 2019) |>
  filter(State %in% c("CALIFORNIA", "NEW JERSEY")) |>
  ggplot(aes(x = Year, y = Value)) +
  facet_wrap(vars(State), scales = "free_y") +
  geom_bar(stat = "identity")
```

## Code

```{r}
#| echo: true
#| eval: false
#| message: false
#| warning: false

bee_data |>
  filter(Year >= 2015 & Year <= 2019) |>
  filter(State %in% c("CALIFORNIA", "NEW JERSEY")) |>
  ggplot(aes(x = Year, y = Value)) +
  facet_wrap(vars(State), scales = "free_y") +
  geom_bar(stat = "identity")
```

::::

## Maps

:::: {.panel-tabset}

## Spatial Viz

```{r}
#| echo: false
#| eval: true
#| message: false
#| warning: false

states_shp |>
  mutate(this_selection = ifelse(NAME == "New Jersey", 
                               "focus", "other states")) |>
  ggplot() +
  geom_sf(aes(fill = this_selection)) +
  labs(title = "State Selection", 
         subtitle = "toward a Dashboard card",
         caption = "Princeton Wintersession") +
  scale_fill_manual(values = c(color_1, "gray80")) +
  theme_minimal()
```

## Code

```{r}
#| echo: true
#| eval: false
#| message: false
#| warning: false

states_shp |>
  mutate(this_selection = ifelse(NAME == "New Jersey", 
                               "focus", "other states")) |>
  ggplot() +
  geom_sf(aes(fill = this_selection)) +
  labs(title = "State Selection", 
         subtitle = "toward a Dashboard card",
         caption = "Princeton Wintersession") +
  scale_fill_manual(values = c(color_1, "gray80")) +
  theme_minimal()
```

## Each State

```{r}
#| echo: false
#| eval: true
#| message: false
#| warning: false

nj_map <- states_shp |>
  mutate(this_selection = ifelse(NAME == "New Jersey", 
                               "focus", "other states")) |>
  ggplot() +
  geom_sf(aes(fill = this_selection)) +
  labs(title = "New Jersey", 
         # subtitle = "toward a Dashboard card",
         caption = "Princeton Wintersession") +
  scale_fill_manual(values = c(color_1, "gray80")) +
  theme_minimal() +
  theme(axis.title.x = element_blank(),
        axis.text.x  = element_blank(),
        axis.ticks.x = element_blank(),
        axis.title.y = element_blank(),
        axis.text.y  = element_blank(),
        axis.ticks.y = element_blank(),
        legend.position = "none")

nj_map
```

## Code

```{r}
#| echo: true
#| eval: false
#| message: false
#| warning: false

nj_map <- states_shp |>
  mutate(this_selection = ifelse(NAME == "New Jersey", 
                               "focus", "other states")) |>
  ggplot() +
  geom_sf(aes(fill = this_selection)) +
  labs(title = "New Jersey", 
         # subtitle = "toward a Dashboard card",
         caption = "Princeton Wintersession") +
  scale_fill_manual(values = c(color_1, "gray80")) +
  theme_minimal() +
  theme(axis.title.x = element_blank(),
        axis.text.x  = element_blank(),
        axis.ticks.x = element_blank(),
        axis.title.y = element_blank(),
        axis.text.y  = element_blank(),
        axis.ticks.y = element_blank(),
        legend.position = "none")

nj_map
```

::::



# Setup (YAML)

In a Posit Cloud session, create a new Quarto document, and then change the `format` to `dashboard`.

```
---
title: "My Dashboard"
format: dashboard
---

# notebook content goes here...
```


# Layout

## Cards

Each additional visible object is arranged into the dashboard as a *card*.

## Rows

By default, the arrangement of the cards is in rows.

```
nj_plot
```

```
nj_map
```

## Columns

To change the arrangements into columns, change the YAML header

```
format: 
  dashboard:
    orientation: columns
```

More specifically, our file will use 

* first-level headers (`#`) for tabs
* second-level headers (`##`) for columns
* third-level headers (`###`) for rows

## Value Boxes

```
::: {.valuebox icon="award-fill" color="#E77500"}
NEW JERSEY
:::
```

Of importance, for a valuebox to appear and function as intended, the `R` code needs to output a value.


## Sidebar

```
# {.sidebar}

Instructions to user goes here
```


## Heights and Widths

At the desired header, use commands like

* `{height = "25%"}`
* `{width = "30%"}`




# Interactivity

## Inputs

Let the user pick the state. Place the following code into the sidebar.

```
selectInput(
  "state_choice",
  "Choose a state:",
  choices = unique(bee_data$State)
)
```

Let the user pick the year.  Place the following code into the sidebar.

```
sliderInput(
  "year_choice", "Choose a year:",
  min = 2015, max = 2023, step = 1, 
  value = 2023,
)
```

## Server

Now, we need use the `shiny` framework to perform these computations in a *reactive* way.  Create the following code chunk, and be sure to set `#| context: server`.

```
#| context: server
output$state_plot <- renderPlot({
  bee_data |> 
  filter(State == input$state_choice) |>
  mutate(num_colonies = Value / 1e3) |>
  mutate(year_color = ifelse(Year == input$year_choice,
         TRUE, FALSE)) |>
  ggplot(aes(x = factor(Year), y = num_colonies)) +
  geom_bar(aes(fill = year_color),
           color = "#121212", ,
           stat = "identity") +
  labs(title = paste0(str_to_title(input$state_choice)),
       subtitle = "Survey of bee colonies",
       caption = "source: USDA NASS",
       x = "year",
       y = "bee colonies (in thousands)") +
  scale_fill_manual(values = c("#121212", "#E77500")) +
  theme_minimal() +
  theme(legend.position = "none")
})

output$state_map <- renderPlot({
  states_shp |>
    mutate(this_selection = ifelse(
      NAME == str_to_title(input$state_choice),
      "focus", "other states")) |>
    ggplot() +
    geom_sf(aes(fill = this_selection)) +
    labs(title = paste0(str_to_title(input$state_choice)),
         caption = "Princeton Wintersession") +
    scale_fill_manual(values = c(color_1, "gray80")) +
    theme_minimal() +
    theme(axis.title.x = element_blank(),
          axis.text.x  = element_blank(),
          axis.ticks.x = element_blank(),
          axis.title.y = element_blank(),
          axis.text.y  = element_blank(),
          axis.ticks.y = element_blank(),
          legend.position = "none")
})

output$num_colonies <- renderText({
  bee_data |>
    filter(State == input$state_choice) |>
    filter(Year == input$year_choice) |>
    pull(Value)
})

output$state_choice <- renderText({input$state_choice})
output$year_choice <- renderText({input$year_choice})
```

## Plots

Now we have have the `output` in place, we can use more `shiny` code to plot that output. Where the barchart code was intended to be, place the following

```
plotOutput("state_plot")
```

Do something similar for the geographic map.

## Valueboxes

Now we have have the `output` in place, we can use more `shiny` code to display  certain numbers. Inside the valueboxes' divs, place the following

```
r textOutput("state_choice")
```

```
r textOutput("year_choice")
```

Do something similar for the number of bee colonies.










    
# Additional Resources

For this workshop, I followed the materials presented by Mine Cetinkaya-Rundel and Posit

* [Part 1](https://www.youtube.com/watch?v=HW7QbqI4fH0)
* [Part 2](https://www.youtube.com/watch?v=KdsQgwaY950)
* [Part 3](https://www.youtube.com/watch?v=NigWSB-jG4Y)

Folks that are interested in learning more about Quarto Dashboards can refer to resources such as the following:

* How to [Automate](https://www.youtube.com/watch?v=xnJuXOw7iu8) your Reporting by Isabella Velasquez
* [Easy Dashboards](https://www.youtube.com/watch?v=R3sNVE8TYeo) by The GRAPH Courses
* Interactive Infographic using [DisplayR](https://www.youtube.com/watch?v=RXdcHCPJRg8) by Melisaa Van Bussel
* [Quarto Dashboards](https://www.youtube.com/watch?v=jX4_Dnzhl0M) by Charles Teague
* [Quarto Dashboards](https://www.youtube.com/watch?v=Kq1hgg2NtE4) by Isabella Velasquez
* How to [Style](https://www.youtube.com/watch?v=jX4_Dnzhl0M) your Quarto docs without HTML & CSS by Albert Rapp

* [Beyond Dashboards](https://www.youtube.com/watch?v=fzykFa6c3LI) by Sean Nguyen
