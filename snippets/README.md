# Tidyverse snippets

Some 'recipes' and patterns used in wrangling and exploring data in R.

### Selecting or filtering by matching a list
`list` is a data frame containing a column of values you'd like to match.
```
data %>%
  filter(col %in% pull(list, matching_col))
```

### Renaming column headers
```
# rename when selecting
data %>%
  select(geography_code, pop = obs_value)

data %>%
  rename(new_header_name = old_header_name)
```

### Renaming labels for a column
```
# method 1
data %>%
  mutate(variable = case_when(
    variable == "old_label_1" ~ "new_label_1",
    TRUE ~ variable
  ))

# method 2
lookup <- c(
  "old_label_1" = "new_label_1",
  "old_label_2" = "new_label_2",
  "old_label_3" = "new_label_3"
)
data %>%
  mutate(variable = lookup[variable])
```

### Filling in missing data
Useful for when there are missing dates in the dataset. `complete` fills in gaps in the date, `full_seq` specifies what to fill it with.
```
complete(date = full_seq(date, period = 1))
```

### Calculating year-on-year percent change
```
data %>%
  group_by(category, month) %>%
  arrange(category, date) %>%
  mutate(yoy_change = (category_value - lag(category_value)) / lag(category_value)) %>%
  ungroup()
```

### Calculating rolling averages
```
library(zoo)

data %>%
  group_by(category) %>%
  arrange(date) %>%
  mutate(rollmean_12month = rollmean(total, k = 12, fill = NA, align = "right)) %>%
  ungroup()
```

### Caluculating variables based on conditions
Think of square brackets as "where".
```
data %>%
  group_by(category) %>%
  mutate(value = min(variable_A[variable_B > 1]))
```

### Read and write data to google sheets
```
library(googlesheets4)

read_sheet("url", sheet = "name")
write_sheet(data, "url", sheet = "name")
```

## ggplot

### Mapping vs setting colours
Inside aesthetics, colour takes an expression that evaluates values to true/false and set colours accordingly.
```
# setting colour
ggplot(data) +
  geom_point(aes(x = x_variable, y = y_variable), colour = "steelblue")

# mapping colours
ggplot(data) +
  geom_point(aes(x = x_variable, y = y_variable), colour = x_variable < 3)
```

### Inverting scales
```
ggplot(data) +
  geom_point(aes(x = x_variable, y = y_variable)) +
  scale_y_reverse()
```

### Reordering categories based on their values
```
# change order of factor levels based on sorted data
data %>%
  arrange(value) %>%
  mutate(category = factor(category, levels = category))

# if we want a specific order, set levels manually
data %>%
  mutate(category = factor(category, levels = c("category_A", "category_B", "category_C", "category_D")))
```

### stat_summary
On the fly calculations on the plot, e.g. adding an average to a dot plot
```
ggplot(data) +
  geom_jitter(aes(x = x_variable, y = y_variable)) +
  stat_summary(aes(x = x_variable, y = y_variable), fun = mean, geom = "point", colour = "red")
```

### Changing scales in facets
```
# flexible scales for each facet
ggplot(data) +
  geom_point(aes(x = x_variable, y = y_variable)) +
  facet_wrap(~ y_variable, scales = "free") # also: "free_x", "free_y"

# removing empty categories
ggplot(data) +
  geom_bar(aes(y = category)) +
  facet_grid(group ~ ., space = "free_y", scales = "free_y")
```

### Coordinates
Limits on scales are applied at the beginning and can remove data points from the plot. Limits on coordinates are applied after the geometry and works more like a window in framing the data (think of it as stretching the fabric of data to fit the frame). Coordinates only affects how the data is drawn.
```
ggplot(data) +
  geom_bar(aes(x = class)) +
  # instead of changing the scale
  # scale_y_continuous(limits = c(0, 40))
  coord_cartesian(ylim = c(0, 40))
```

### Changing legend type
Mapping both colour and size to a single variable for the legend
```
ggplot(data) +
  geom_point(aes(x = x_variable, y = y_variable, colour = z_variable, size = z_variable)) +
  guides(colour = "legend")
```

### Rotating x-axis tick labels

### geom_histogram
When evaluating colour according to an expression, the histogram will stack the distribution by default. You can change this to show the distribution is overlapping (with alpha), or display them side by side, by changing the position setting.
```
# overlapping
ggplot(data) +
  geom_histogram(aes(x = x_variable, y = y_variable, fill = z_variable < 60), position = "identity", alpha = 0.5)

# side by side
ggplot(data) +
  geom_histogram(aes(x = x_variable, y = y_variable, fill = z_variable < 60), position = "dodge")
```

### Saving ggplot
```
mychart <- data %>% ggplot(aes(x, y)) + geom_line()
ggsave("charts/mychart.svg", plot = mychart, device = "svg)
```