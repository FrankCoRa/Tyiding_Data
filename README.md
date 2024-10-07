# TidyingData_Python
Cleaning and organizing Tiktok data using Python
## Understand the situation
Prepare by reading in the data, viewing the data dictionary, and exploring the dataset to identify key variables for the stakeholder.
The purpose of this project is to investigate and understand the data provided. This activity will cover:
- Compile summary information about the data
- Begin the process of EDA and reveal insights contained in the data
- In-depth Exploratory Data Analysis
## Imports and data loading
We start by importing the packages that you will need to load and explore the dataset.

Then, load the dataset into a dataframe. Creating a dataframe will help you conduct data manipulation, exploratory data analysis (EDA), and statistical activities.
```r
# Import packages
import pandas as pd
import numpy as np
data = pd.read_csv("tiktok_dataset.csv")
```
## Understand the data
- Use `data.head()` to preview the first few rows of the dataset, allowing for an initial inspection of data structure.
- Execute `data.info()` to assess variable types, identify null values, and understand the overall composition of numeric and categorical data.
- Apply `data.describe()` to generate summary statistics, revealing key insights such as distributions, central tendency, and potential outliers in numeric variables.

A good first step towards understanding the data might therefore be examining the `claim_status` variable. Begin by determining how many videos there are for each different claim status.
```r
# What are the different values for claim status and how many of each are in the data?
data['claim_status'].value_counts()
```
The counts of each claim status are quite balanced:
- claim:      9608
- opinion:    9476

We used Boolean masking to filter the data according to claim status, then calculate the mean and median view counts for each claim status.
```r
# What is the average view count of videos with "claim" status?
claims = data[data['claim_status'] == 'claim']
print('Mean view count claims:', claims['video_view_count'].mean())
print('Median view count claims:', claims['video_view_count'].median())

# What is the average view count of videos with "opinion" status?
opinions = data[data['claim_status'] == 'opinion']
print('Mean view count opinions:', opinions['video_view_count'].mean())
print('Median view count opinions:', opinions['video_view_count'].median())
```
Having as result:
- Mean view count claims: 501029.4527477102
- Median view count claims: 501555.0
- Mean view count opinions: 4956.43224989447
- Median view count opinions: 4953.0
The mean and the median within each claim category are close to one another, but there is a vast discrepancy between view counts for videos labeled as claims and videos labeled as opinions.

Now , we calculate how many videos there are for each combination of categories of claim status and author ban status.
```r
# Get counts for each group combination of claim status and author ban status
data.groupby(['claim_status', 'author_ban_status']).count()[['#']]
```
| Claim Status | Author Ban Status | #     |
|--------------|-------------------|-------|
| Claim        | Active             | 6,566 |
|              | Banned             | 1,439 |
|              | Under Review       | 1,603 |
| Opinion      | Active             | 8,817 |
|              | Banned             | 196   |
|              | Under              | 463   |  

There are many more claim videos with banned authors than there are opinion videos with banned authors. This could mean a number of things, including the possibilities that:
- Claim videos are more strictly policed than opinion videos
- Authors must comply with a stricter set of rules if they post a claim than if they post an opinion.
  
Now we continue investigating engagement levels, now focusing on `author_ban_status`.

Calculating the count, mean, and median of each author ban status.
```r
data.groupby(['author_ban_status']).agg(
    {'video_view_count': ['mean', 'median'],
     'video_like_count': ['mean', 'median'],
     'video_share_count': ['mean', 'median']})
```
| author_ban_status | video_view_count (count) | video_view_count (mean) | video_view_count (median) | video_like_count (count) | video_like_count (mean) | video_like_count (median) | video_share_count (count) | video_share_count (mean) | video_share_count (median) |
|-------------------|--------------------------|-------------------------|---------------------------|--------------------------|-------------------------|---------------------------|---------------------------|--------------------------|----------------------------|
| active            | 15383                    | 215,927.04               | 8,616.0                   | 15383                    | 71,036.53               | 2,222.0                   | 15383                     | 14,111.47                | 437.0                      |
| banned            | 1635                     | 445,845.44               | 448,201.0                 | 1635                     | 153,017.24              | 105,573.0                 | 1635                      | 29,998.94                | 14,468.0                   |
| under review      | 2066                     | 392,204.84               | 365,245.5                 | 2066                     | 128,718.05              | 71,204.5                  | 2066                      | 25,774.70                | 9,444.0                    |

A few observations stand out:
- Banned authors and those under review get far more views, likes, and shares than active authors.
- In most groups, the mean is much greater than the median, which indicates that there are some videos with very high engagement counts.

Now, We created three new columns to help better understand engagement rates:
- 'likes_per_view': represents the number of likes divided by the number of views for each video
- 'comments_per_view': represents the number of comments divided by the number of views for each video
- 'shares_per_view': represents the number of shares divided by the number of views for each video
```r
# Create a likes_per_view column
data['likes_per_view'] = data['video_like_count'] / data['video_view_count']
# Create a comments_per_view column
data['comments_per_view'] = data['video_comment_count'] / data['video_view_count']
# Create a shares_per_view column
data['shares_per_view'] = data['video_share_count'] / data['video_view_count']
```
We used `groupby()` to compile the information in each of the three newly created columns for each combination of categories of claim status and author ban status, then use `agg()` to calculate the count, the mean, and the median of each group.
```r
data.groupby(['claim_status', 'author_ban_status']).agg(
    {'likes_per_view': ['count', 'mean', 'median'],
     'comments_per_view': ['count', 'mean', 'median'],
     'shares_per_view': ['count', 'mean', 'median']})
```
Final Results:
![Alt text](https://github.com/FrankCoRa/TidyingData_Python/blob/main/tidying_results.png)
- Video performance and engagement: Videos from banned or under-review authors tend to attract significantly more views, likes, and shares compared to those by active authors. However, the engagement rate (likes, shares, and comments per view) is influenced more by the video’s claim status rather than the author’s ban status.
- Claim vs. opinion videos: Claim videos consistently outperform opinion videos in terms of views, likes, and engagement metrics. This indicates that claim videos not only garner higher viewership but are also more positively received, generating more likes, comments, and shares compared to opinion-based content.
- Engagement trends by author ban status: For claim videos, banned authors exhibit slightly higher likes/view and shares/view ratios than active or under-review authors. In contrast, for opinion videos, active and under-review authors achieve higher engagement rates across all metrics when compared to banned authors.
