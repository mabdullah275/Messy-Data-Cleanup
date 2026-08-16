# Messy Data Cleanup
Performed data cleaning on the 2023 Ask A Manager Salary Survey responses.<br/><br/>
**Skills:** Data Cleaning <br/>
**Tools:** Python and its libraries - Pandas, NumPy, RegEx.

## Scenario
The [responses](/Ask%20A%20Manager%20Salary%20Survey%202023%20(Responses).csv) to the [Ask A Manager Salary Survey 2023](https://docs.google.com/forms/d/10sn2XFmvjtRxrw7vedkpSp3cAd2kvKrOjnqHpeUXl4U/viewform?edit_requested=true) are informative but inconsistent, suffering from missing data, formatting inconsistencies, and unstandardised categories. My goal is to clean and standardise the dataset so it becomes complete, consistent, and ready for meaningful analysis.

## Code File
[`data_cleaning.ipynb`](/data_cleaning.ipynb)

## Data Cleaning Methodology

The raw salary survey data contained user-submitted responses, which meant that several fields contained missing values, inconsistent formatting, free-text responses and different currencies. My aim during the cleaning process was to create a dataset that was more consistent, comparable and suitable for analysis while retaining the variables most relevant to the project.

### 1. Initial inspection

I first loaded the dataset using Pandas and inspected its dimensions, sample records, column names, data types and missing values.

The original dataset contained **17,161 rows and 20 columns**.

This initial inspection helped identify the main cleaning requirements before making any changes to the data.

### 2. Simplifying column names

The original survey questions were used as column names and were relatively long. I renamed these to shorter, consistent `snake_case` names such as:

* `Annual salary (gross)` → `annual_salary_gross`
* `Additional monetary compensation` → `additional_compensation`
* `Years of experience in field` → `experience_field`
* `Highest level of education completed` → `education_level`
* `Remote or on-site?` → `remote_or_onsite`

The purpose was to make the dataset easier to read and the columns easier to reference throughout the analysis.

### 3. Cleaning the submission date

The survey timestamp was converted into a date value and the time component was removed.

Since the project focuses on the **2023 salary survey**, I then filtered the dataset so that only responses submitted during 2023 were retained.

### 4. Consolidating currency information

The survey contained both a `currency` field and a separate `currency_other` field for respondents who selected "Other".

I combined these into a single `currency` column. Where the respondent selected "Other", their free-text currency response was used instead. The now-redundant `currency_other` column was then removed.

This produced a single field representing the currency associated with each salary.

### 5. Standardising industry categories

Because `industry` was important to the planned analysis, records without an industry value were removed.

The survey also allowed respondents to provide values outside the predefined industry options. To maintain consistent groups for comparison, I retained only responses that matched the original checkbox categories supplied by the survey.

The reasoning here was to favour a smaller set of clearly defined categories rather than attempting to manually interpret and recategorise every free-text industry response.

### 6. Handling missing compensation

Missing values in `additional_compensation` were replaced with `0`.

For this analysis, a missing additional-compensation value was therefore treated as no additional monetary compensation. This ensured that the field remained numeric and could later be converted and analysed consistently alongside salary.

### 7. Cleaning demographic fields

For `gender`, I retained responses belonging to the three defined categories:

* Man
* Woman
* Non-binary

Responses outside these categories or without a valid value were removed to maintain consistent groups for analysis.

Rows without an `education_level` were also removed because education was one of the categorical variables required for later analysis.

For the `race` column, respondents could select multiple options. Where a response also contained `"Another option not listed here or prefer not to answer"`, I removed only that part of the response while preserving any other race categories they had selected.

If removing that value left the race field empty, or if race was missing entirely, the row was removed.

### 8. Converting salaries to a common currency

Salary comparisons are not meaningful when the values are expressed in different currencies, so I standardised both:

* `annual_salary_gross`
* `additional_compensation`

to **GBP**.

I used average 2023 exchange rates and applied the appropriate conversion rate according to each respondent's currency. Once converted, the `currency` value was updated to `GBP`.

For the combined `AUD/NZD` survey option, I used the average of the AUD and NZD conversion rates.

Responses using currencies for which a conversion rate had not been defined were removed. This ensured that every salary remaining in the analytical dataset was expressed on the same monetary scale.

### 9. Normalising country names

The `country` field contained free-text responses, which introduced differences in spelling, abbreviations and formatting.

I first removed periods and then used Python's `SequenceMatcher` to compare each response against a predefined set of country names. Each value was mapped to the country name with the highest similarity score when it exceeded the chosen similarity threshold.

I chose fuzzy matching because manually correcting every country response would not scale well for a dataset of this size.

This approach is not guaranteed to classify every response perfectly, so I inspected the resulting unique country values afterwards rather than relying entirely on the automated matching.

Following this inspection, I standardised:

* `United States` → `USA`
* `United Kingdom` → `UK`

I also found cases where respondents had entered sentences or unrelated information rather than simply entering a country. As a simple additional filter, country values longer than 30 characters were removed.

### 10. Resetting the index

Several filtering operations removed rows from the dataset. I therefore reset the DataFrame index so that the cleaned dataset had a continuous index from zero onwards.

### 11. Removing unnecessary features

Finally, I removed columns that were not required for the objectives of the analysis:

* `age`
* `job_title_context`
* `income_context`
* `us_state`
* `experience_overall`

For experience, I retained `experience_field` instead of `experience_overall` because experience within a respondent's current field was more relevant to the planned salary analysis.

This step reduced unnecessary information and kept the final dataset focused on the variables required for the project.

### 12. Exporting the cleaned dataset

After completing the cleaning and standardisation process, I exported the resulting DataFrame as:

[`clean_dataset.csv`](/clean_dataset.csv)

This cleaned dataset can now be used for subsequent analysis.

## Overall Approach

My approach was based on three main principles:

1. **Consistency** — standardise categories, country names, column names and currencies so that values could be compared reliably.
2. **Analytical relevance** — retain variables and responses that were useful for the intended analysis and remove fields that did not contribute to it.
3. **Scalability** — use programmatic methods, such as predefined category filtering and fuzzy country matching, instead of manually correcting thousands of survey responses.

Some cleaning decisions involve assumptions—for example, treating missing additional compensation as zero, restricting industries and gender values to predefined categories, and using fuzzy matching for country names. These decisions were made to produce a structured and comparable dataset, but they should also be considered when interpreting the final results.
