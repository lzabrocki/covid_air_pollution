# Individual and Environmental Risk Factors for COVID-19 Mortality in Elderly

![plot](./3.outputs/1.figures/0.eda/graph_mortality_mean_exposure.pdf)

This repo provides all the necessary R codes to reproduce the analysis of the paper entitled "Individual and Environmental Risk Factors for COVID-19 Mortality in Elderly". Due to statistical confidentiality, the data used in the project could not be shared. Should you have any questions or find coding errors, please do not hesistate to reach me at leo.zabrocki[at]psemail.eu.

**The repo is organized as follows:**

* `1.data`: the folder that would contain the dataset.
* `2.scripts`: contains all Rmarkdown and .html files to reproduce the analysis.
  * `1.data_wrangling` would contain the script to clean and merge the raw datasets on patients' health status and environmental exposure.
  * `2.eda` contains the script to carry out an exploratory data analysis.
  * `3.analysis` contains the scripts to reproduce the **statististical analyses**.
* `3.outputs`: contains the tables and graphs displayed in the paper.

**In `2.scripts/3.analysis`, each folder contains a specific analysis:**

* `1.study_mortality_comorbidities`: contains the analysis for the association between co-morbidities and COVID-19 mortality.
* `2.study_comorbidites_long_term_air_pollution`: contains the analysis for the association between long-term exposure to air pollutants and co-morbidities.
* `3.study_mortality_long_term_air_pollution`: contains the analysis for the association between long-term exposure to air pollutants and COVID-19 mortality.
* `4.study_mortality_short_term_environmental_exposure`: contains the analysis for the assocation between short-term exposure to environmental indicators (air pollution and weather parameters) and COVID-19 mortality.

**Important remark:** all codes are rendered as .html files using the [Distill](https://rstudio.github.io/distill/) package. If you want to see the rendered output, you should first download the raw file and then display it with your web browser.






