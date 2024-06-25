

```{r setup, include=FALSE}
library(tidyverse)
#library(dsbox)
library(lubridate)
library(ggplot2)
    
knitr::opts_chunk$set(out.width = "100%")
```




## Packages

This assignment will use the following packages:

-   `tidyverse`: a collection of packages for doing data analysis in a "tidy" way
-   `ggplot2`: a package that contains the function for the grammer of graphics
-   `lubridate`: a package for formatting dates in R (in case if you need)

We use the `library()` function to load packages.
In your R Markdown document you should see an R chunk labelled `load-packages` which has the necessary code for loading both packages.

```{r load-packages}
library(tidyverse)
library(ggplot2)
library(lubridate)
```




# Airbnb listings in Edinburgh

Airbnb is an online marketplace where people can book short-term stays in hotels or in other people's houses. 
Recent developments in Edinburgh regarding the growth of Airbnb and its impact on the housing market means a better understanding of the Airbnb listings is needed.
Using data provided by Airbnb, we can explore how Airbnb availability and prices vary by neighbourhood.

------

A copy of the Edinburgh Airbnb data set has been available (because of dsbox package version problem in Rstudio). To read this data, ensure that you have the following code near the beginning of your R Markdown file, from your directory having the data file (ie. `data/`).

```{r load-edibnb-data, message = FALSE}
edibnb <- read_csv("edibnb.csv")
```


------

You can view the dataset as a spreadsheet using the `View()` function.
Note that you should _not_ put this function in your R Markdown document, but instead type it directly in the Console, as it pops open a new window.
When you run this in the console, you will see the following **data viewer** window pop up.

```{r view-data, eval = FALSE}
View(edibnb)
```

- You can find out more about the dataset by inspecting its documentation, which you can access by running `?edibnb` in the Console or using the Help menu in RStudio to search for `edibnb` (If you can install and call `dsbox` package correctly). 

- Otherwise, you can also find this information [here](https://rstudio-education.github.io/dsbox/reference/edibnb.html).

The `edibnb` data set contain `r nrow(edibnb)` listings in Edinburgh, with values about `r ncol(edibnb)` different variables that includes the property neighborhood location, number of rooms and review ratings.


Edinburgh council collects the second data set from their assessments of a randomly selected sample of listings. The variables in this data set are:
  -   `id`, the ID number of the listing (according to `edibnb`)
  -   `rating`, the council's rating of the listing of the property either being good, adequate or poor.
  -   `assessment_date`, the date the council performed their assessment of the property.
  
The data is contained in the file `council_assessments.csv` within the `data/` directory. It can be loaded into your workflow using the following code (or any other directory name that you have instead of `/data`).

```{r message = FALSE}
council <- read_csv("council_assessments.csv")
```


## Exercise 1 

- In the `edibnb` data set, what is the ID of the listing that has the highest number of reviews with a perfect review score of 100%? 

- Which variables have the missing observations in the `edibnb` data set. 

- Visualize the missing observations using some functions from visdat or naniar packages (Notes from Week 4 might be useful)



```{r}

library(dplyr)
library(visdat)

# Read the data
edibnb <- read_csv("edibnb.csv")

# Find the listing with the highest number of perfect reviews
perfect_reviews <- edibnb %>%
  filter(review_scores_rating == 100) %>%
  group_by(id) %>%
  summarise(n = n()) %>%
  arrange(desc(n))

# Extract ID of the listing with the highest number of perfect reviews
top_listing_id <- perfect_reviews$id[1]

# Print the ID of the listing
print(paste("Listing ID with the highest number of perfect reviews:", top_listing_id))

# Variables with missing observations
vis_miss(edibnb)  # Visualizing missing observations




```



## Exercise 2 

Calculate the minimum, maximum and average price from the Airbnb properties in Southside for a single night stay for four people (Try to use the `summarise` function).



```{r}

library(tidyverse)

# Read the data
edibnb <- read_csv("edibnb.csv")

# Filter Airbnb properties in the Southside neighborhood that can accommodate four people
southside_df <- edibnb %>%
  filter(neighbourhood == "Southside" & accommodates == 4)

# Calculate minimum, maximum, and average price for a single night stay for four people
price_summary <- southside_df %>%
  summarise(min_price = min(price),
            max_price = max(price),
            avg_price = mean(price))

# Display the price summary
price_summary




```

## Exercise 3 

When looking at the data you will notice that some of the listings have a value for the number of bathrooms that is not a whole number, e.g. 1.5 or 2.5. 

- Mutate the `bathrooms` variable to round the number of bathrooms _up_ to the nearest whole number (hint, look at `ceiling()`). 

- Using this mutated variable, how many listings have more bathrooms than bedrooms?



```{r}
library(dplyr)

# Read the data
edibnb <- read.csv("edibnb.csv")

# Mutate the bathrooms variable to round up to the nearest whole number
edibnb <- edibnb %>%
  mutate(rounded_bathrooms = ceiling(bathrooms))

# Filter listings where the number of bathrooms is greater than the number of bedrooms
listings_with_more_bathrooms <- edibnb %>%
  filter(rounded_bathrooms > bedrooms) %>%
  # Summarize to count the number of listings meeting the criteria
  summarise(n = n())

# Display the count of listings with more bathrooms than bedrooms
listings_with_more_bathrooms


```

## Exercise 4 

Join the `edibnb` to the `council` data frames. Create a bar plot of the `neighbourhood` variable for the properties that have been assessed by the council (remember to iterate the data visualization to make it as informative as possible). 

- What does the data visualization suggest? 

- Is the council targeting all neighborhoods within Edinburgh equally?




```{r}

library(tidyverse)

# Read the data
council_data <- read.csv("council_assessments.csv")
edibnb_data <- read.csv("edibnb.csv")

# Rename 'id' column in council_data to match with edibnb_data
colnames(council_data)[1] <- "id"

# Join the data frames
joined_data <- left_join(edibnb_data, council_data, by = "id")

# Filter properties assessed by the council
assessed_properties <- joined_data %>%
  filter(!is.na(assessment_date))

# Create a bar plot
assessed_properties %>%
  ggplot(aes(x = neighbourhood, fill = neighbourhood)) +
  geom_bar() +
  labs(title = "Number of Properties Assessed by Council in Each Neighbourhood",
       x = "Neighbourhood",
       y = "Number of Properties") +
  theme_minimal()




```




