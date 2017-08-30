## Usage

For each website, you just need to run `scrape.R`. It will look for existing data, and if there are none, it will go ahead and connect to the websites to gather data. 

Then you can load the data frame `all_recipes_df` in each case by

```r
load(file="all_recipes.RData")
```


