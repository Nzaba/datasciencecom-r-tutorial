# From web scraping to modelling: Analyzing online food blogs with R
### Author: Burak Himmetoglu
---------

The web is an ocean where data scientists can gather lots of useful and interesting data. Despite its vastness, this data usually comes in a rather messy format, and requires significant cleaning and wrangling before it can be used in an inferential study. Therefore, the data scientist's **hacking skills** are needed for this usually quite cumbersome task.  

In this tutorial, I will walk the reader through the steps of obtaining, cleaning and visualizing data scraped from the web using R. As an example, I will consider online food blogs and illustrate how one can get insights on recipes, ingredients and visitors' food preferences. The tutorial will also illustrate the use of the [tidyverse](https://www.tidyverse.org/) packages in R, which offer an excellent set of data wrangling and visualization tools. 

## Web Scraping
First, we need to obtain the data from the blog posts. For this tutorial, I have chosen to scrape data from two sites:

1. [Pich of Yum](http://pinchofyum.com)
2. [The Full Helping](https://www.thefullhelping.com/)

These are excellent food blogs with lots of great recipes and nice photographs. Let us consider the first blog (Pinch of Yum) as an example, since it has more recipe entries. There are 51 pages (at the time where this tutorial was written) of recipes, each containing 15 recipe links. The first task is to collect all the links to these recipes, which we can do with the following code snippet

```r
get_recipe_links <- function(page_number){
  page <- read_html(paste0("http://pinchofyum.com/recipes?fwp_paged=", 
  			as.character(page_number)))
  links <- html_nodes(page, "a")
  
  # Get locations of recipe links
  loc <- which(str_detect(links, "<a class"))
  links <- links[loc]

  # Trim the text to get proper links
  all_recipe_links <- map_chr(links, trim_)
  
  # Return
  all_recipe_links
 }
 ```

 Given a page number (1 to 51), the function `get_recipe_links` first reads the given page, and then stores all the links to each recipe. In every page, the links to recipes are found within `<a class="block-link" href=" ... ">`, so we first get the nodes associated with *a* by the `html_nodes` function of the `rvest` package. Then, using `str_detect`, we obtain the locations of each link as a list. The `trim_` function is applied on all the links in the list by `map_chr` function, which returns a clean link without some unwanted characters like `\` and `<`.  `trim_` is a user function which looks like

 ```r
trim_ <- function(link){
	temp1 <- str_split(link, " ")[[1]][3] %>%
      str_replace_all("\"", "") %>% # Remove \'s
      str_replace("href=", "") %>%
      str_replace(">", " ")

    # Return 
    str_split(temp1, " ")[[1]][1]
 }
```
To be able to see how I came uo with these, the reader should open one of the pages and look at the html source (which can be done by any browser's source view tool). The source code can be messy, but locating relevant pieces of information by repeating patterns become straighforward after looking through a few of the pages that are being scraped. The code for scraping all the recipe links from [The Full Helping](https://www.thefullhelping.com/) is almost identical, with a few small twists. 

Now we have all the links, the next step is to one by one connect to each link and gather the data from each recipe. This step is more tedious than the previous one, since every site stores its recipe data in a different format. For example, [Pinch of Yum](http://pinchofyum.com) uses a `json` format for each recipe, which is excellent since the data is pretty much standard across all the pages. Instead, [The Full Helping](https://www.thefullhelping.com/) has the recipe information in html, so it requires a bit more work to collect. 

Let's look at how we collect data from each recipe in Pinch of Yum. The below code snippet illustrates the main pieces of this process

```r
# Get the recipe page
page <- read_html(link_to_recipe)
  
# Get recipe publish date/time
meta <- html_nodes(page, "meta")
dt <- meta[map_lgl(meta, str_detect, "article:published_time")] %>%
    str_replace_all("\"|/|<|>", " ") %>%
    str_replace_all("meta|property=|article:published_time|content=", "") %>%
    str_trim() %>%
    str_split("T")
  
date <- dt[[1]][1]
time <- dt[[1]][2]

# JSON data
script_ <- html_nodes(page, "script")
loc_json <- which(str_detect(script_, "application/ld"))
if (length(loc_json) == 0){
  return(NULL) # If the link does not contain a recipe, just return null
}
  
# Load the data to JSON
recipe_data <- fromJSON(html_text(script_[loc_json][[1]]))
```

The above code snippet first reads the page from a give `link_to_recipe`, then collects the date and time when the recipe is published and finally reads the recipe data which is in JSON format. The date/time information is stored in the node "meta" and after we get it, we simply clean it with `stringr` operations like `str_replace_all`, `str_trim` and `str_split`. I recommend the reader to open the source of one of the recipes in her/his browser and locate the *meta* node with date/time information and compare with the code snippet to see how it all works. 

Obtaining the JSON data is rather straightforward with `fromJSON` function of the `jsonlite` package. While the JSON data comes rather clean after this step, there are a few nooks and crannies one has to deal with to put the data in a useful format. I will not discuss these steps in this post, they are located in the repository and I recommend the reader to check them out. At the end, the user function `get_recipe_data` which contains the above snippet, return a data frame containing the recipe information from a given `link_to_recipe`. 

Now that we have the two functions `get_recipe_links` and `get_recipe_data`, we can scrape the whole site by using the following lines of code:

```r 
# Get all links
all_links <- 1:51 %>% map(get_recipe_links) %>% unlist()

# Get all recipes and combine in a single data frame
all_recipes_df <- rbind(all_recipes_df, 
                        get_recipe_data(link_to_recipe))
```
The data frame `all_recipes_df` contains all the recipes on the blog and has the following fields:

```
 [1] "name"                "pub_time"            "pub_date"            "description"        
 [5] "ingredients"         "prepTime"            "cookTime"            "nReviews"           
 [9] "rating"              "servingSize"         "calories"            "sugarContent"       
[13] "sodiumContent"       "fatContent"          "saturatedFatContent" "transFatContent"    
[17] "carbohydrateContent" "fiberContent"        "proteinContent"      "cholesterolContent" 
```

The code for The Full Helping is significantly different, and the returned data frame does not have the same fields. I will not dicsuss it here on the post, but the code is included in the repository for interested readers. 

## Exploratory Analysis
Now that we have the collected all the recipes, we can do some exploration. First, let's see which words are most common in the recipes. We will make use of the `tidytext` and `tokenizers` packages for this task. The `ingredients` field contains a flat text file with each line ending with "\n". So our task is to split the text, remove any leftover html tags, and then convert the data frame in long format. What is meant with the long format is that each column will contain one word, so a ringle recipe is spread into multiple columns, whose size is determined by the number of words in the ingredients. This is achieved by the following code snippet

```r
# Assing and ID number to each recipe
all_recipes_df <- all_recipes_df %>% mutate(ID = 1:nrow(all_recipes_df))

# Construct a data frame using words appearing in ingredients
df_ingrdt <- all_recipes_df %>% 
  select(ID, ingredients) %>%
  mutate(ingredients = str_replace(ingredients, "\n", " ") %>% 
  	str_replace("<.*?>", " ")) %>%
  unnest_tokens(word, ingredients)
```

For example, the first 10 entries would look like
```
##     ID   word
## 1    1      1
## 1.1  1      4
## 1.2  1    cup
## 1.3  1  olive
## 1.4  1    oil
## 1.5  1      2
## 1.6  1 cloves
## 1.7  1 garlic
## 1.8  1 minced
## 1.9  1      1
```
The `unnest_tokens` function achieves the transformation to the long format. Notice that this is not great, since we have numbers and other non-informative words that is so common in all the ingredients, that we would like to get rid of them. While there are better ways of removing very common but not informative words (e.g. [tf-idf](https://en.wikipedia.org/wiki/Tf-idf)), let's simply remove `stop_words` and some common mesurement related words manually:

```r
# Stop words from tokenizers package
data("stop_words")

# Also remove the following (which is not included in stopwords)
word_remove = c("cup", "cups", "teaspoon", "teaspoons", "tablespoon", "tablespoons", 
                "ounce", "ounces", "lb", "lbs", "tbs", "tsp", "oz", "handful", "handfull",
                "inch", "i", "can")

df_ingrdt <- df_ingrdt %>% 
  filter(!(word %in% stopwords())) %>%
  filter(!(word %in% word_remove)) %>%
  filter(!(str_detect(word, "[0-9]")))  # Remove numbers as well
```

After this step, we have a much better looking data frame. The wordcloud from all the ingredients look lik

![title](img/wordcloud.png)

Now ...

