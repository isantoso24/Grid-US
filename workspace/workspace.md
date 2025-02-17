Project Workspace
================
Energyyy

``` r
library(knitr)
knitr::opts_chunk$set(message = FALSE, warning = FALSE)
```

# 1. Load packages

``` r
library(tidyverse)
library(broom)
library(tidyr)
library(readxl)
#install.packages("leaflet")
library(leaflet)
#install.packages("sf")
library(sf)
#install.packages("gganimate")
library(gganimate)
library(ggthemes)
library(gapminder)
#install.packages("visdat")
library(visdat)
#install.packages("naniar")
library(naniar)
```

# 2. Introduction

The United States of America has one of the most diverse electric grids
in the world. With that many stakeholders it is interesting to know
which factors affect electricity distribution and pricing to which
extent in the US. We are looking into data from the U.S. Energy
Information Administration (EIA). EIA collects, analyzes, and
disseminates independent and impartial information on the US energy
sector. The utility companies are required to self-report this data due
to national regulations. The variables that we will be looking at are
ownership type, customers (Count), sales (Megawatt hours), revenues
(Thousands Dollars), and average price (cents/kWh), US state, energy
sectors, disturbances, and CAIDI - a reliability index.

## Research Question

Which factors affect electricity distribution and pricing to what extent
in the US?

- sectors

- regional differences in reliability

- utility company’s ownership

# 3. Data

``` r
table_Residential <- read_excel("/cloud/project/data/table_Residential.xlsx",
                                na = c("."))
table_Commercial <- read_excel("/cloud/project/data/table_Commercial.xlsx",
                               na = c("."))
table_Industrial <- read_excel("/cloud/project/data/table_Industrial.xlsx", 
                               na = c("."))
table_Transportation <- read_excel("/cloud/project/data/table_Transportation.xlsx", 
                                   na = c("."))
table_Disturbance <- read_excel("/cloud/project/data/table_Disturbance.xlsx",
                                na= c(".", ". Hours,  . Minutes", "Unknow", ".        ."))
table_CAIDI <- read_excel("/cloud/project/data/table_CAIDI.xlsx")

USA_df <- sf::st_read("/cloud/project/data/USA_States_Generalized.shp")
```

    ## Reading layer `USA_States_Generalized' from data source 
    ##   `/cloud/project/data/USA_States_Generalized.shp' using driver `ESRI Shapefile'
    ## Simple feature collection with 51 features and 56 fields
    ## Geometry type: MULTIPOLYGON
    ## Dimension:     XY
    ## Bounding box:  xmin: -178.2176 ymin: 18.92179 xmax: -66.96927 ymax: 71.40624
    ## Geodetic CRS:  WGS 84

# 4. Ethics review

## Limitations in data sources:

- table_CAIDI: Most utility companies use IEE standards to report CAIDI,
  but some do not. We have decided to leave out companies of our
  analysis that do not use IEE standards, so we will not have a complete
  account of ALL utility companies, but of most of them. We decided not
  to use the CAIDI index that includes Major Event Days because those
  represent hurricanes, storms etc. which utility companies do not have
  control over. Thus, they do not attest to the utility’s ability to
  provide reliable services.

- The data collection is standardized and done by governmental
  institutions, so the data should be reliable, of high quality and
  equal across utility companies and sectors. However, the EIA relies on
  the honesty of the utility company to share accurate data with the
  EIA.

- Ilham and Anna do not have an extensive background in the energy
  industry, so we rely on Rudy for most of the expertise in our project.

## Positive effects on people:

- Education of lay people: might help citizens to be better informed of
  the US energy industry which is highly complex.
- Decision-makers: gain a better understanding to improve policy-making
  and can therefore evaluate the efficiency and effectiveness of utility
  companies. This enables them to encourage the companies to become more
  efficient and effective.
- Education of Rudy: improves his personal understanding of the US
  energy sector
- We could enhance these positive effects by dispersing our research
  more and communicating it to citizens and decision-makers.

## Negative effects on people:

- The reputation of specific utility companies might suffer from our
  research if they are connected to higher prices for lower-quality
  offering of energy services.

## Minimising negative impact:

- We will talk about the limitations/disclaimers of our research, so
  that people are aware of what we cannot show with our data, so that
  people do not draw wrong conclusions from our research.
  - This is a beneficial action to take because people will be more
    aware of the limitations and can understand our data and our
    visualizations better.
- We are presenting to our class at COA only and we do not intend to
  share our outcomes with the wider community where it would lead to the
  implementation of policies etc.

# 5. Tidying data

This mutate function adds a column which contains the sector of the
data.

``` r
USA_df <- USA_df %>%
 rename(`Census Division\r\nand State` = STATE_NAME)
residential <- table_Residential %>%
  mutate(Sector = c("Residential"))
commercial <- table_Commercial %>%
  mutate(Sector = c("Commercial"))
transportation <- table_Transportation %>%
  mutate(Sector = c("Transportation"))
industrial <- table_Industrial %>%
  mutate(Sector = c("Industrial"))
```

This code binds all the tables with sectors together.

``` r
energy_sector <- bind_rows(residential, commercial, industrial, transportation)
```

``` r
table_Disturbance <- table_Disturbance |>
  mutate(Entity = `Utility/Power Pool`)

energy_sector_rev_sum <- energy_sector |> 
  select(Entity, Ownership, `Revenues (Thousands Dollars)`) |>
  group_by(Entity, Ownership) |>
  summarise(sum_revenue = sum(`Revenues (Thousands Dollars)`)) |>
  as.data.frame()

energy_sector_customer_sum <- energy_sector |> 
  select(Entity, Ownership, `Customers (Count)`) |>
  group_by(Entity, Ownership) |>
  summarise(sum_customer = sum(`Customers (Count)`)) |>
  as.data.frame()
```

``` r
table_CAIDI <- table_CAIDI |>
 pivot_longer(cols = `2013`:`2022`,
              names_to = "Year",
              values_to = "CAIDI")
```

``` r
disturbance_count <- table_Disturbance |>
  group_by(Entity) |>
  count()

disturbance_complete <- disturbance_count |> 
  inner_join(energy_sector_rev_sum)
```

``` r
disturbance_ownership <- disturbance_count |> 
  inner_join(energy_sector_customer_sum)
```

## Missing data

``` r
vis_miss(energy_sector)
```

<img src="workspace_files/figure-gfm/vis-missing-data-1.png" alt="Graph showing percentage of missing data points for each variable in three dataframes (Energy sector, Disturbance, CAIDI). Most datasets are almost complete with the Disturbance dataset missing the most data points at 3.9 %"  />

``` r
vis_miss(table_Disturbance)
```

<img src="workspace_files/figure-gfm/vis-missing-data-2.png" alt="Graph showing percentage of missing data points for each variable in three dataframes (Energy sector, Disturbance, CAIDI). Most datasets are almost complete with the Disturbance dataset missing the most data points at 3.9 %"  />

``` r
vis_miss(table_CAIDI)
```

<img src="workspace_files/figure-gfm/vis-missing-data-3.png" alt="Graph showing percentage of missing data points for each variable in three dataframes (Energy sector, Disturbance, CAIDI). Most datasets are almost complete with the Disturbance dataset missing the most data points at 3.9 %"  />

## Theme

``` r
our_theme <- theme_grey() + 
  theme(text = element_text(color = "#1d91c0"), 
        legend.text = element_text(color = "black"))
```

# 6. Data Analysis

## 6.1. Ownership vs price of electricity

**Q:** Does ownership affect the price of electricity? We wonder whether
cooperatives offer on average a lower price of electricity because that
is what the proposers of the Pine Tree Power bill in Maine have recently
advertised to convince people to vote in favor of buying out
investor-owned company to turn them into cooperatives. And, looking at
differences between sectors, is there a general trend for all sectors or
do different ownership types offer different price levels to different
sectors? E.g. Are cooperatives more beneficial for the residential
sector?

``` r
energy_sector |>
  filter(!is.na(Ownership)) |>
  mutate(
    Ownership = fct_relevel(
      Ownership,
      "Cooperative", "Municipal", "Political Subdivision", "State", "Federal", "Behind the Meter", "Retail Power Marketer", "Investor Owned"
    )) |>

 ggplot(aes(x = as.numeric(Ownership), y = `Average Price (cents/kWh)`)) +
  geom_boxplot(outlier.shape = NA, aes(color = Ownership)) +
  geom_smooth(method = "loess", color = "black") +
  ylim(0,25)+
  facet_wrap(~Sector) +
  scale_color_manual(values = c("Cooperative" = "#ffffd9", "Municipal" = "#edf8b1", "Political Subdivision" = "#c7e9b4", "State" = "#7fcdbb", "Federal" = "#41b6c4", "Behind the Meter" = "#1d91c0", "Retail Power Marketer" = "#225ea8", "Investor Owned" = "#0c2c84")) +
  labs(
    title = "Average price (cents/kWh) according to type of ownership",
    subtitle = "Per sector based on 2022 data",
    caption = "Source : https://www.eia.gov/electricity/data.php#sales",
    x = "") +
  our_theme +
  theme(axis.text.x = element_blank(), 
        axis.ticks.x = element_blank(),
        panel.grid = element_blank())
```

<img src="workspace_files/figure-gfm/ownership-average-price-state-1.png" alt="Boxplot with trendline of average price in cents per kilowatthour in 2022 in each sector according to type of ownership. Types of ownership are ordered on a spectrum from mor cooperative to more investor-owned. Federally-owned companies and companies owned by political subdivisions seem to offer lower than average prices in all sectors. The industrial sector generally has lower average prices than the other sectors."  />

``` r
#ggsave(filename = "Ownership vs price.png", device = "png")
```

**A:** Federally-owned utility companies offer below average prices in
all sectors. All other ownerships do not deviate significantly from the
average price. We do not pay a lot of attention to the transportation
sector, even though its trendline looks different and interesting,
because there are only 43 data points in comparison to more than 1500
data entries for the commercial and residential sector.

``` r
ggplot(energy_sector, aes(x = forcats::fct_infreq(Sector), fill = Sector)) +
  geom_bar() +
  labs(title = "Number of data points per sector", x = "Sector", y = "Count") +
  scale_fill_viridis_d() +
  our_theme +
  theme(legend.position = "none") 
```

<img src="workspace_files/figure-gfm/vis-number-of-data-points-sector-1.png" alt="Barplot showing number of data points for each sector. Number ranges from 1500 entries for the commercial and residential sectors to 43 entries for the transportation sector."  />

## 6.2. Disturbance and Reliability

### 6.2.1. Blackouts and ownership

**Q:** How do blackouts, revenue and ownership relate to each other? Do
cooperatives that are less profit-oriented, but customer-oriented
correlate with less blackouts? Or do bigger companies have more means to
prevent blackouts than small cooperatives, even if they are for profit
and investor-owned?

``` r
ggplot(disturbance_complete, aes(
  y = sum_revenue/1000000, 
  x = `Ownership`,
  size = n)) +
  geom_point(color = "#1d91c0", alpha = 0.8) +
  labs(title = "Number of disturbances per utility company", 
       subtitle = "in relation to ownership type and revenue", 
       x = "Type of ownership", y = "Revenue in Million Dollars ($)", 
       size = "Number of\ndisturbances",
       caption = "Source : https://www.eia.gov/electricity/data.php#sales") +
  our_theme
```

<img src="workspace_files/figure-gfm/disturbances-vs-revenue-and-ownership-1.png" alt="Scatterplot of number of Disturbances per utility company in relation to the type of ownership of the utility company and the revenue made by the company in 2022 in Millions of dollars. Size of the points reflects number of disturbances experienced in 2022. Investor-owned companies experience by far the most disturbances."  />

**A:** Most disturbances occur in investor-owned companies. Revenue does
not seem to play a major role in how many disturbances occur as the
disturbances in investor-owned companies happen in all companies with
low and high revenues. To check whether the number of disturbances is so
high for investor-owned companies because there are simply more
investor-owned companies than e.g. cooperatives or municipally-owned
utilities, we plotted the number of data entries for each type of
ownership.

``` r
energy_sector_customer_sum |>
  filter(!is.na(Ownership)) |>
ggplot(aes(x = forcats::fct_infreq(Ownership), fill = Ownership)) +
  geom_bar() +
  labs(title = "Number of data points per type of ownership", x = "Type of ownership", y = "Count", caption = "Source : https://www.eia.gov/electricity/data.php#sales") +
  scale_fill_viridis_d() +
  our_theme +
  theme(legend.position = "none") +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))
```

<img src="workspace_files/figure-gfm/vis-number-of-data-points-ownership-1.png" alt="Barplot showing number of data points for each Type of Ownership. Number ranges from 1800 entries for cooperatives and 600 for investor-owned companies to around 50 entries for state-owned companies."  />
This shows us that the number of disturbances in investor-owned
companies is not high simply because there are many companies with this
type of ownership because there are three times as many cooperatives and
they still do not have a lot of disturbances.

**Q:** Maybe, most disturbances are related to investor-owned companies
because they simply serve more customers?

``` r
energy_sector_customer_sum |>
  filter(!is.na(Ownership)) |>
ggplot(aes(x = Ownership, y = sum_customer/1000000)) +
  geom_boxplot() +
  ylim(0, 4) +
  labs(title = "Number of customers versus ownership", subtitle = "in each utility company", y = "Number of customers (in Millions)") +
  our_theme +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))
```

<img src="workspace_files/figure-gfm/number-of-customers-per-ownership-1.png" alt="Boxplot showing number of customers of each utility company versus ownership. Investor-owned companies have by far the most customers with an average of 500000 customers per utility company, some even serving more than 4 million households. All other ownership types serve less than 100000 customers on average."  />

**A:** Investor-owned companies have by far the most customers. All
other ownership types serve a lot less customers on average. Thus, the
amount of disturbances in investor-owned companies might correlate
positively with the number of customers they serve.

``` r
disturbance_ownership_average <- disturbance_ownership |>
  group_by(Ownership) |>
  summarise(mean(sum_customer), mean(n))
  
ggplot(disturbance_ownership_average, aes(
         x = `mean(sum_customer)`/1000, 
         y = `mean(n)`)) +
   geom_smooth(method = "glm", alpha = 0.5) +
  geom_point(aes(color = Ownership), size = 3) +
  scale_color_viridis_d() +
  labs(title = "Average number of customers vs number of disturbances",
       subtitle = "per type of ownership", 
       x = "Average number of customers (in Thousands)", 
       y = "Average number of disturbances") +
  our_theme
```

<img src="workspace_files/figure-gfm/disturbances-customers-average-per-ownership-1.png" alt="Scatterplot with trendline showing average number of customers versus average number of disturbances per type of ownership. Investor-owned companies have a below-average number of disturbances, while companies owned by political subdivisions have an above average number of disturbances."  />

``` r
#ggsave(filename = "Average_customer_disturbances.png", device = "png")
```

**A:** There seems to be a positive correlation between number of
customers per type of ownership and number of disturbances.
Investor-owned companies have a below-average number of disturbances,
while companies owned by political subdivisions have an above average
number of disturbances. Federally-owned and cooperative-owned utilities
have low average numbers of customers and low numbers of disturbances.

### 6.2.2. Electricity network reliability across the US

**Q:** Not taking into account major event days, such as hurricanes,
storms, etc. that cause blackouts, how does the Customer Average
Interruption Duration Index (CAIDI) vary throughout the US states over
time from 2013-2022?

``` r
# Create a base plot
CAIDI_leaflet_df$Year <-as.numeric(CAIDI_leaflet_df$Year)

p <- CAIDI_leaflet_df |>
  ggplot(aes(fill = CAIDI,
             group = Year)) +
  geom_sf() +
  xlim(125, 68) +
  ylim(24.5, 50) +
  scale_fill_viridis_c() +
  our_theme +
  theme(axis.text.x = element_blank(), 
        axis.ticks.x = element_blank(),
        axis.text.y = element_blank(), 
        axis.ticks.y = element_blank(),
        panel.grid = element_blank(),
        panel.background = element_rect(fill='white')) +
  labs(title = "Customer Average Interruption Duration Index (CAIDI) map of lower 48 US states", 
       subtitle = "Year: {closest_state}",
       caption = "Source: https://www.eia.gov/electricity/data/annual/") 

# Create an animated plot
p <- p + transition_states(
  states = Year,
  transition_length = 3,
  state_length = 3)
#animate(p, duration = 30, renderer = gifski_renderer("map.gif"))
```

**A:** West Virginia and Michigan are consistently suffering from low
reliability in electricity supply while Florida and North Dakota have a
low number of interruptions. There are no dramatic changes for any of
the states over time.

## 6.3. Amount of electricity consumed vs average price

**Q:** How does the amount of electricity consumed per customer affect
price? Is it cheaper to buy electricity depending on the sector?

Historically, the industrial sector benefited from lower electricity
prices because they constituted a reliable demand and were the preferred
customers for utility companies because it was cheaper for the utility
company to provide electricity to customers with a constant, reliable
demand than it was for them to provide to the residential sector where
demand always varies (Bakke, 2016).

Does this tendency toward the industrial sector still exist today?

Bakke, G., 2016. “The Grid: The Fraying Wires Between Americans and Our
Energy Future”. Bloomsbury, USA.

``` r
#creating the mean of average price in each sector
line_test <- energy_sector |>
  summarise(mean(`Average Price (cents/kWh)`, na.rm = TRUE),.by = Sector)

line_test$mean <- line_test$`mean(\`Average Price (cents/kWh)\`, na.rm = TRUE)` 

line_test$label <-
  round(line_test$mean, 2)

#create graph using mean of average price in each sector
energy_sector |>
  ggplot(aes(x = (
    `Sales (Megawatthours)`*1000/`Customers (Count)`), y = `Average Price (cents/kWh)`, color = Sector)) +
  geom_point() +
  geom_smooth() +
  geom_hline(data = line_test, aes(yintercept = mean)) +
  geom_text(data = line_test, aes(50000000, y = label, label = label, vjust = -1), color = "black") +
  facet_wrap(~Sector) +
  scale_color_viridis_d() +
  xlim(0, 400000000) +
  labs(
    title = "Average kWh consumption per customer vs average price",
    subtitle = "by sector in 2022",
    x = "Average kWh consumption per customer",
    y = "Average price (cents/kWh)",
    caption = "Source : https://www.eia.gov/electricity/data.php#sales"
  ) +
  our_theme +
  theme(panel.grid = element_blank())
```

<img src="workspace_files/figure-gfm/average-price-in-kWH-consumption-1.png" alt="Scatterplot showing average kilowatthour consumption per customer versus average price in cents per kilowatthour, faceted by sector. Each facet includes the mean average price offered by the companies to each sector. The industrial and transportation sector have the widest spread of electricity use while the Residential and Commercial sector use less electricity per customer. The industrial sector pays on average the least for their electricity with 9.25 cents per kilowatthour in comparison to 13.55 cents per kilowatthour in the residential sector where there are many customers which consume smaller amounts of electricity each."  />

``` r
ggsave(filename = "price_per_cust_consumption.png", device = "png")
```

**Q:** Yes, this tendency still exists today as the industrial sector
with the highest electricity consumption per consumer still gets the
lowest average prices. There does not seem to be a general trend of
lower prices for higher electricity consumption per customer as none of
the facets show a negative trendline.

## 6.4. Number of customers vs price in the residential sector

**Q:** Considering only the residential sector, how does the number of
customers that a utility company serves affect the average price they
offer? Are bigger utility companies with more customers able to offer
lower average prices due to scale?

``` r
energy_sector |>
  filter(Sector == "Residential") |>
  ggplot(aes(x = `Customers (Count)`, y = `Average Price (cents/kWh)`)) +
  geom_point() +
  geom_smooth() +
  xlim(0, 250000) +
  labs(title = "Average price vs. number of customers", 
       subtitle = "per utility company for the residential sector in 2022", 
       x = "Number of customers", 
       y = "Average price (cents/kWh)",
       caption = "Source : https://www.eia.gov/electricity/data.php#sales") +
  our_theme
```

<img src="workspace_files/figure-gfm/price-by-customers-1.png" alt="Scatterplot and trendline of Number of Customers in the Residential Sector per Utility Company versus average price in cents per kilowatthour. There is no correlation between the number of customers and the average price. The price that the utility companies offer depends on other factors. Most utility companies serve electricity to less than 50000 households, while some serve 250000 and more households."  />

**A:** Scale does not seem to play an important role in determining
prices as average price does not lower with an increase in the number of
customers. In fact, there is a slight opposite trend.

## 6.5. Per household consumption of electricity vs price

**Q:** Considering only the residential sector, how does electricity
consumption vary in relation to the average price? Do consumers use less
electricity if it is more expensive?

This analysis does not consider general trends in electricity prices
over time, but only the difference between different utility companies.

``` r
ggplot(table_Residential, aes(x = (`Sales (Megawatthours)`*1000)/`Customers (Count)`, y = `Average Price (cents/kWh)`)) +
  geom_hex() +
  geom_smooth() +
  labs(title = "Electricity Consumption per Customer vs. Average Price", 
       subtitle = "in the Residential Sector in 2022", 
       x = "Average Electricity Consumption per Customer (in kWh)", 
       y = "Average Price (cents/kWh)", 
       fill = "Number of\nUtility\nCompanies",
       caption = "Source : https://www.eia.gov/electricity/data.php#sales") +
  our_theme
```

<img src="workspace_files/figure-gfm/household-consumption-electricity-1.png" alt="Heatmap and trendline of average electricity consumption per customer in the Residential Sector in kilowatthours of each utility company versus average price in cents per kilowatthours offered by the respective utility company. Heatmap shows that most customers use around 14000 kilowatthours of electrcity annually paying between 10 to 14 cents per kilowatthour. There is a slight negative correlation. Household consumption is lower when the price is high."  />

``` r
ggsave(filename = "heatmap_consumption.png", device = "png")
```

**A:** There is a slight negative correlation between average
electricity consumption per customer and average price. Most customers
use around 14,000 kilowatthours of electricity annually paying between
10 to 14 cents per kilowatthour. The per household consumption of
electricity is slightly lower when the price is high.
