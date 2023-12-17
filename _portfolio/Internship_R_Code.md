---
title: "Academic Internship Assignment: R Code"
excerpt: "R code from my academic assignment during my internship at the Royal Danish Embassy in Tokyo. This project focuses on Japan's aging population, utilizing data analysis to reveal trends and implications in a critical demographic issue."
collection: portfolio
---

# Introduction

This R code represents the core of my assignment, which I've developed during my academic internship at the Royal Danish Embassy in Tokyo, Japan. The project focuses on the aging population in Japan, a critical demographic issue with profound social and economic implications. My assignment aims to delve into the complexities of this phenomenon by analyzing various datasets to understand the underlying dynamics better.

# Code
```r
###################### preamble ######################
library("pacman")     
p_load(
         dplyr
       , tidyr
       , stringr
       , ggplot2
       , rio,lubridate
       , plotly
       , patchwork
       , cowplot
       , mapdata
       , mapproj
       , rnaturalearth
       , rnaturalearthdata
       , viridis
       )

# when installing rnaturalearth for the first time uncomment the two lines below
#install.packages("devtools")
#devtools::install_github("ropensci/rnaturalearthhires")

# workspace path
path_to_workspace <- "/Users/olivernyropweeks/Library/CloudStorage/OneDrive-UniversityofCopenhagen/KUonedriveOW/8. Semester - Praktik/Internship Assignment/Workspace/"

###################### 65 % above graph ###################### 

# data import
file_path <- paste0(path_to_workspace, "WorldBankJapanAge.xls")
df <- import(file_path, skip = 2)

# filter for top 10 biggest economies
countries_of_interest <- c(
                            "Japan"
                            ,"Denmark"
                            ,"United States"
                            ,"China"
                            ,"Germany"
                            ,"India"
                            ,"United Kingdom"
                            ,"France"
                            ,"Italy"
                            ,"Brazil"
                            ,"Canada"
                            , "World"
                           
                           )  

# filter for top 10 GDP per capita
countries_of_interest2 <- c(
                            "Japan"
                            ,"Denmark"
                            ,"Luxembourg"
                            ,"Ireland"
                            ,"Switzerland"
                            ,"Norway"
                            ,"Singapore"
                            ,"Qatar"
                            ,"United States"
                            ,"Iceland"
                            ,"Australia"
                            , "World"

  
) 

# filter the data for selected countries
selected_data <- df %>%
  filter(`Country Name` %in% countries_of_interest)

selected_data2 <- df %>%
  filter(`Country Name` %in% countries_of_interest2)

# convert the data from wide to long format
selected_long <- selected_data %>% 
  pivot_longer(
    cols = matches("^\\d{4}$"),
    names_to = "Year",
    values_to = "Percentage65Plus"
  ) %>% 
  mutate(Year = as.numeric(Year)) # convert year to numeric if it's not already

selected_long2 <- selected_data2 %>% 
  pivot_longer(
    cols = matches("^\\d{4}$"),
    names_to = "Year",
    values_to = "Percentage65Plus"
  ) %>% 
  mutate(Year = as.numeric(Year)) # convert year to numeric if it's not already

# reorder the factor levels so that it is in specified order
selected_long$`Country Name` <- factor(selected_long$`Country Name`, levels = countries_of_interest)
selected_long2$`Country Name` <- factor(selected_long2$`Country Name`, levels = countries_of_interest2)

# create the plot for top 10 largest economies
plot1 <- ggplot(data = selected_long, aes(x = Year, y = Percentage65Plus, color = `Country Name`)) +
  geom_line(size = 1.2) +
  labs(
    x = "Year",
    y = "Population age 65 & above (%)",
    title = "A) Population age 65 & above over years for top 10 countries by GDP (current prices, $)"
  ) +
  xlim(1959, 2023) +
  scale_x_continuous(
    breaks = seq(from = min(selected_long$Year), to = max(selected_long$Year), by = 5)
  ) +
  ylim(0, 32) +
  theme_minimal() +
  theme(
    text = element_text(size = 12),
    axis.title.x = element_blank(),
    axis.text.x = element_blank(),
    axis.title.y = element_text(size = 14, margin = margin(t = 0, r = 10, b = 0, l = 0)),
    axis.text = element_text(size = 12),
    plot.title = element_text(size = 16, hjust = 0.5)
  )

# create the plot for top 10 GDP per capita
plot2 <- ggplot(data = selected_long2, aes(x = Year, y = Percentage65Plus, color = `Country Name`)) +
  geom_line(size = 1.2) +
  labs(
    x = "Year",
    y = "Population age 65 & above (%)",
    title = "B) Population age 65 & above over years for top 10 countries by GDP (current prices, $) per capita"
  ) +
  xlim(1959, 2023) +
  scale_x_continuous(
    breaks = seq(from = min(selected_long$Year), to = max(selected_long$Year), by = 5)
  ) +
  ylim(0, 32) +
  theme_minimal() +
  theme(
    text = element_text(size = 12),
    axis.title.x = element_text(size = 14, margin = margin(t = 10, r = 0, b = 0, l = 0)), 
    axis.title.y = element_text(size = 14, margin = margin(t = 0, r = 10, b = 0, l = 0)), 
    axis.text = element_text(size = 12),
    plot.title = element_text(size = 16, hjust = 0.5)
  )

# print plots and combine
print(plot1)
print(plot2)
combined_plot <- plot1 / plot2
print(combined_plot)

# save the plot
Pic_path_combined <- paste0(path_to_workspace, "CombinedPlots.pdf")
ggsave(Pic_path_combined, plot = combined_plot, width = 11, height = 8, dpi = 300)

###################### Japan Statistics 4-way graph ###################### 

# load datasets
population_path <- paste0(path_to_workspace, "Population.xls")
population <- import(population_path, skip = 3)

death_path <- paste0(path_to_workspace, "DeathRate.xls")
death <- import(death_path, skip = 3)

birth_path <- paste0(path_to_workspace, "BirthRate.xls")
birth <- import(birth_path, skip = 3)

fertility_path <- paste0(path_to_workspace, "FertilityRate.xls")
fertility <- import(fertility_path, skip = 3)

# filter for Japan and convert from wide to long format
population_long <- population %>%
  filter(`Country Name` == "Japan") %>%
  pivot_longer(
    cols = matches("^\\d{4}$"),
    names_to = "Year",
    values_to = "Population"
  ) %>%
  mutate(
    Year = as.numeric(Year),
    Population = Population / 1e9  # convert population to billions
  )

death_long <- death %>%
  filter(`Country Name` == "Japan") %>%
  pivot_longer(
    cols = matches("^\\d{4}$"),
    names_to = "Year",
    values_to = "DeathRate"
  ) %>%
  mutate(Year = as.numeric(Year))

birth_long <- birth %>%
  filter(`Country Name` == "Japan") %>%
  pivot_longer(
    cols = matches("^\\d{4}$"),
    names_to = "Year",
    values_to = "BirthRate"
  ) %>%
  mutate(Year = as.numeric(Year))

fertility_long <- fertility %>%
  filter(`Country Name` == "Japan") %>%
  pivot_longer(
    cols = matches("^\\d{4}$"),
    names_to = "Year",
    values_to = "FertilityRate"
  ) %>%
  mutate(Year = as.numeric(Year))

# pop plot
plot_population <- ggplot(data = population_long, aes(x = Year, y = Population)) +
  geom_line(size = 1.2) +
  labs(
    y = "Total Population (Billions)",
    title = "A) Population"
  ) +
  scale_x_continuous(breaks = seq(from = min(population_long$Year), to = max(population_long$Year), by = 10), limits = c(1959, 2023)) +
  theme_minimal() +
  theme(
    text = element_text(size = 12),
    axis.title.x = element_text(size = 14, margin = margin(t = 10, r = 0, b = 0, l = 0)),
    axis.title.y = element_text(size = 14, margin = margin(t = 0, r = 10, b = 0, l = 0)),
    axis.text.y = element_text(size = 12),
    plot.title = element_text(size = 16, hjust = 0.5)
  )

# death plot
plot_death <- ggplot(data = death_long, aes(x = Year, y = DeathRate)) +
  geom_line(size = 1.2) +
  labs(
    y = "Death Rate",
    title = "B) Death rate, crude (per 1,000 people)"
  ) +
  scale_x_continuous(breaks = seq(from = min(death_long$Year), to = max(death_long$Year), by = 10), limits = c(1959, 2023)) +
  theme_minimal() +
  theme(
    text = element_text(size = 12),
    axis.title.x = element_text(size = 14, margin = margin(t = 10, r = 0, b = 0, l = 0)),
    axis.title.y = element_text(size = 14, margin = margin(t = 0, r = 10, b = 0, l = 0)),
    axis.text.y = element_text(size = 12),
    plot.title = element_text(size = 16, hjust = 0.5)
  )

# birth plot
plot_birth <- ggplot(data = birth_long, aes(x = Year, y = BirthRate)) +
  geom_line(size = 1.2) +
  labs(
    x = "Year",
    y = "Birth Rate",
    title = "C) Birth rate, crude (per 1,000 people)"
  ) +
  scale_x_continuous(breaks = seq(from = min(birth_long$Year), to = max(birth_long$Year), by = 10), limits = c(1959, 2023)) +
  theme_minimal() +
  theme(
    text = element_text(size = 12),
    axis.title.x = element_text(size = 14, margin = margin(t = 10, r = 0, b = 0, l = 0)),
    axis.title.y = element_text(size = 14, margin = margin(t = 0, r = 10, b = 0, l = 0)),
    axis.text.y = element_text(size = 12),
    plot.title = element_text(size = 16, hjust = 0.5)
  )

# fertility plot
plot_fertility <- ggplot(data = fertility_long, aes(x = Year, y = FertilityRate)) +
  geom_line(size = 1.2) +
  labs(
    x = "Year",
    y = "Fertility Rate, total (births per woman)e",
    title = "D) Fertility rate"
  ) +
  scale_x_continuous(breaks = seq(from = min(fertility_long$Year), to = max(fertility_long$Year), by = 10), limits = c(1959, 2023)) +
  theme_minimal() +
  theme(
    text = element_text(size = 12),
    axis.title.x = element_text(size = 14, margin = margin(t = 10, r = 0, b = 0, l = 0)),
    axis.title.y = element_text(size = 14, margin = margin(t = 0, r = 10, b = 0, l = 0)),
    axis.text.y = element_text(size = 12),
    plot.title = element_text(size = 16, hjust = 0.5)
  )

# combine the plots in a 4-way split using patchwork
combined_plots <- (plot_population | plot_death) /
  (plot_birth | plot_fertility)

plot(combined_plots)

# save the combined plot
combined_overview_path <- paste0(path_to_workspace, "JapanStatistics.pdf")
ggsave(combined_overview_path, plot = combined_plots, width = 11, height = 8, dpi = 300)

# make the 3-way split plot

# nerge birth and death rate data
birth_death_long <- full_join(birth_long, death_long, by = "Year")

# create a combined birth and death rate plot
plot_birth_death <- ggplot(birth_death_long) +
  geom_line(size = 1.2,aes(x = Year, y = BirthRate, color = "Birth Rate")) +
  geom_line(size = 1.2,aes(x = Year, y = DeathRate, color = "Death Rate")) +
  labs(
    x = "Year",
    y = "Rate (per 1,000 people)",
    color = "Rate Type",
    title = "C) Birth and Death Rates"
  ) +
  scale_x_continuous(breaks = seq(from = min(death_long$Year), to = max(death_long$Year), by = 5), limits = c(1959, 2023)) +
  theme_minimal() +
  theme(
    legend.position = "bottom",
    text = element_text(size = 12),
    axis.title.x = element_text(size = 14),
    axis.title.y = element_text(size = 14),
    plot.title = element_text(size = 16, hjust = 0.5)
  )

# combine all plots with the 3-way split using patchwork
combined_plots3way <- (plot_population | plot_fertility ) /
  plot_birth_death +
  plot_layout(heights = c(1, 0.5)) # adjust the relative heights as needed

# print the combined plot
print(combined_plots3way)

# save the combined plot
combined_overview3way_path <- paste0(path_to_workspace, "JapanStatistics3way.pdf")
ggsave(combined_overview3way_path, plot = combined_plots3way, width = 11, height = 8, dpi = 300)

###################### Violin graphs ###################### 

# graph for 1996

# load data
JPAgePop1996_path <- paste0(path_to_workspace, "JapanAgePop1996.xlsx")
JPAgePop1996<- import(JPAgePop1996_path)

# pivot the data to a long format
JPAgePop1996_long <- JPAgePop1996 %>%
  pivot_longer(
    cols = c('Male', 'Female'),
    names_to = 'Gender',
    values_to = 'Count'
  ) %>%
  mutate(
    Count = ifelse(Gender == 'Male', -Count, Count),
    Gender = factor(Gender, levels = c('Male', 'Female')) 
  )

# create the population pyramid plot
pyramid_plot1996 <- ggplot(JPAgePop1996_long, aes(x = Age, y = Count, fill = Gender)) +
  geom_bar(stat = "identity") +
  scale_y_continuous(labels = abs, limits = c(-1300, 1300)) +
  scale_x_continuous(breaks = seq(from = min(JPAgePop1996_long$Age), 
                                  to = max(JPAgePop1996_long$Age), by = 5)
                     ,labels = function(x) ifelse(x == 90, " 90 +", ifelse(x > 90, " ", x))
  ) +
  coord_flip() +
  labs(x = "Age", y = "Population Count (Thousands)", fill = "Gender", title = "A) 1996") +
  theme_minimal() +
  theme(
    text = element_text(size = 16),
    legend.position = "none",
    axis.title.x = element_blank(), 
    plot.title = element_text(hjust = 0.5)
  )

print(pyramid_plot1996)

# graph for 2010

# load data
JPAgePop2010_path <- paste0(path_to_workspace, "JapanAgePop2010.xlsx")
JPAgePop2010<- import(JPAgePop2010_path)

# pivot the data to a long format
JPAgePop2010_long <- JPAgePop2010 %>%
  pivot_longer(
    cols = c('Male', 'Female'),
    names_to = 'Gender',
    values_to = 'Count'
  ) %>%
  mutate(
    Count = ifelse(Gender == 'Male', -Count, Count),
    Gender = factor(Gender, levels = c('Male', 'Female')) 
  )

# create the population pyramid plot
pyramid_plot2010 <- ggplot(JPAgePop2010_long, aes(x = Age, y = Count, fill = Gender)) +
  geom_bar(stat = "identity") +
  scale_y_continuous(labels = abs, limits = c(-1300, 1300)) +
  scale_x_continuous(breaks = seq(from = min(JPAgePop2010_long$Age), 
                                  to = max(JPAgePop2010_long$Age), by = 5)
                     ,labels = function(x) ifelse(x == 100, "100+", x)
  ) +
  coord_flip() +
  labs(x = "Age", y = "Population Count (Thousands)", fill = "Gender", title = "B) 2010") +
  theme_minimal() +
  theme(
    text = element_text(size = 16),
    legend.position = "top",
    axis.text.x = element_text(angle = 90, hjust = 1),
    axis.title.x = element_text(size = 14, margin = margin(t = 10, r = 0, b = 0, l = 0)),
    plot.title = element_text(hjust = 0.5),
    axis.title.y = element_blank(), 
  )

print(pyramid_plot2010)

# graph for 2022

# load data
JPAgePop2022_path <- paste0(path_to_workspace, "JapanAgePop2022.xlsx")
JPAgePop2022<- import(JPAgePop2022_path)

# pivot the data to a long format
JPAgePop2022_long <- JPAgePop2022 %>%
  pivot_longer(
    cols = c('Male', 'Female'),
    names_to = 'Gender',
    values_to = 'Count'
  ) %>%
  mutate(
    Count = ifelse(Gender == 'Male', -Count, Count),
    Gender = factor(Gender, levels = c('Male', 'Female'))
  )

# create the population pyramid plot
pyramid_plot2022 <- ggplot(JPAgePop2022_long, aes(x = Age, y = Count, fill = Gender)) +
  geom_bar(stat = "identity") +
  scale_y_continuous(labels = abs, limits = c(-1300, 1300)) +
  scale_x_continuous(breaks = seq(from = min(JPAgePop2022_long$Age), 
                                  to = max(JPAgePop2022_long$Age), by = 5)
                     ,labels = function(x) ifelse(x == 100, "100+", x)
  ) +
  coord_flip() +
  labs(x = "Age", y = "Population Count (Thousands)", fill = "Gender", title = "C) 2022") +
  theme_minimal() +
  theme(
    text = element_text(size = 16),
    legend.position = "none",
    axis.text.x = element_text(angle = 90, hjust = 1),
    axis.title.x = element_blank(), 
    plot.title = element_text(hjust = 0.5),
    axis.title.y = element_blank() 
  )


# print the plot
print(pyramid_plot2022)

violins <- pyramid_plot1996 + pyramid_plot2010 + pyramid_plot2022

print(violins)

violins_save_path <- paste0(path_to_workspace, "ViolinPlots.pdf")
ggsave(violins_save_path, plot = violins, width = 11, height = 8, dpi = 300)


###################### Geo Plot ###################### 

# load map data
map_japan <- rnaturalearth::ne_states("Japan", returnclass = "sf")

# load age / pop data
file_names <- c("PrefectureAge.xlsx", "PrefecturePop.xlsx")
clean_and_match <- function(file_name, map_data) {
  # load data
  data_path <- paste0(path_to_workspace, file_name)
  data <- import(data_path)
  
  # clean prefecture column
  data$Prefecture <- sub("-ken$", "", data$Prefecture)
  data$Prefecture <- gsub("Gumma", "Gunma", data$Prefecture)
  data$Prefecture <- gsub("Tokyo-to", "Tokyo", data$Prefecture)
  data$Prefecture <- gsub("Kyoto-fu", "Kyoto", data$Prefecture)
  data$Prefecture <- gsub("Osaka-fu", "Osaka", data$Prefecture)
  data$Prefecture <- gsub("Hyogo", "Hyōgo", data$Prefecture)
  
  # remove the entry for "Japan"
  data <- data %>% 
    filter(Prefecture != "Japan")
  
  # nerge with map data
  merged_data <- merge(map_data, data, by.x = "woe_name", by.y = "Prefecture")
  
  return(merged_data)
}

# loop over the file names and process them
list_of_merged_data <- lapply(file_names, clean_and_match, map_data = map_japan)

# retrieve dataset from list
prefecture_age_merge = list_of_merged_data[[1]]
prefecture_pop_merge = list_of_merged_data[[2]]

# make plot
choro_plot_age <- ggplot(prefecture_age_merge) +
  geom_sf(aes(fill = `65 and over`)) +
  labs(
    fill = "% Aged 65+",
    title = "A) % of population aged 65 and above, 2022"
  ) +
  scale_fill_viridis_c(
    name = "% of population aged 65+", 
    breaks = round(seq(min(prefecture_age_merge$`65 and over`), max(prefecture_age_merge$`65 and over`), length.out = 5), digits = 1)  # Ensure breaks are whole numbers
  ) +
  guides(
    fill = guide_colourbar(
      barwidth = 10,
      title.hjust = 0.1, 
      title.vjust = 0.75, 
    )
  ) +
  theme(
    ) +
  theme_minimal() +
  theme(
    legend.position = "bottom",
    plot.title = element_text(hjust = 0.5),
    axis.text.x = element_blank(),
    axis.text.y = element_blank(),
    axis.ticks = element_blank(), 
    panel.grid.major = element_blank(),
    panel.grid.minor = element_blank()
  ) +
  coord_sf(ylim = c(30.5, 45), xlim = c(125,145))

plot(choro_plot_age)

# save the plot
Pic_path_choro_age <- paste0(path_to_workspace, "JapanChoroplethAge.pdf")
ggsave(Pic_path_choro_age, plot = choro_plot_age, width =4.5, height = 5, dpi = 300)

# nake plot

# calculate the median value of PctPopChange21_22
median_value <- median(prefecture_pop_merge$PctPopChange21_22, na.rm = TRUE)

choro_plot_pop <- ggplot(prefecture_pop_merge) +
  geom_sf(aes(fill = PctPopChange21_22)) +
  labs(
    fill = "Pop Change",
    title = "B) % change in population, 2021-2022"
  ) +
  scale_fill_viridis_c(
    name = "% change in population",
    breaks = round(seq(min(prefecture_pop_merge$PctPopChange21_22), max(prefecture_pop_merge$PctPopChange21_22), length.out = 5), digits = 1)  # Ensure breaks are whole numbers
  ) +
  guides(
    fill = guide_colourbar(
      barwidth = 10,
      title.hjust = 0.1,
      title.vjust = 0.75,
    )
  ) +
  theme(
  ) +
  theme_minimal() +
  theme(
    legend.position = "bottom", 
    plot.title = element_text(hjust = 0.5),
    axis.text.x = element_blank(),
    axis.text.y = element_blank(), 
    axis.ticks = element_blank(),  
    panel.grid.major = element_blank(), 
    panel.grid.minor = element_blank()  
  ) +
  coord_sf(ylim = c(30.5, 45), xlim = c(125,145))

plot(choro_plot_pop)

# save the plot
Pic_path_choro_pop <- paste0(path_to_workspace, "JapanChoroplethPop.pdf")
ggsave(Pic_path_choro_pop, plot = choro_plot_pop, width =4.5, height = 5, dpi = 300)

# combine plots

combined_choro <- choro_plot_age + choro_plot_pop
print(combined_choro)

Pic_path_choro_combined <- paste0(path_to_workspace, "JapanChoroplethCombined.pdf")
ggsave(Pic_path_choro_combined , plot = combined_choro, width =9, height = 5, dpi = 300)

###################### labor graph ###################### 

LaborData_path <- paste0(path_to_workspace, "LaborData.xlsx")
LaborData<- import(LaborData_path)

scaling_factor <- 0.0005

# plot
laborplot <-ggplot(LaborData, aes(x = Year)) +
  geom_bar(aes(y = `Rate` / scaling_factor, fill = "Application/Job Opening Rate"), stat = "identity", alpha = 0.7) +
  geom_line(aes(y = `Monthly active applications(person(s))` / 1000, color = "Applications"), linewidth = 2) +
  geom_line(aes(y = `Monthly active job openings (person (s))` / 1000, color = "Job Openings"), linewidth = 2) +
  labs(color = "", fill = "") +
  scale_fill_manual(values = c("Application/Job Opening Rate" = "darkgrey")) +
  scale_y_continuous(
    name = "Monthly Active Job Applicants & Job Openings \n (Thousands of People)",
    labels = scales::comma, 
    sec.axis = sec_axis(~ . * scaling_factor, name = "Rate", labels = scales::comma, breaks = seq(from = 0, to = 1.75, by = 0.25))
    ,limits = c(0, 3500)
    ,breaks = seq(from = 0, to = max(LaborData$`Monthly active job openings (person (s))`), by = 500)) +
  scale_x_continuous(
    breaks = seq(from = min(LaborData$Year), to = max(LaborData$Year), by = 1)
  ) +
  theme_minimal() +
  theme(
    text = element_text(size = 14),
    axis.title.x = element_text(size = 12, margin = margin(t = 10, r = 0, b = 0, l = 0)),
    axis.title.y = element_text(size = 12, margin = margin(t = 0, r = 10, b = 0, l = 0)),
    axis.title.y.right = element_text(size = 12, margin = margin(t = 0, r = 0, b = 0, l = 10)),
    legend.position = "bottom"
  )

laborplot

Pic_path_laborplot <- paste0(path_to_workspace, "LaborPlot.pdf")
ggsave(Pic_path_laborplot , plot = laborplot, width =9, height = 5.5, dpi = 300)

  ylab("Employment-to-Population Ratio (%)")

# CLEAR ####
#rm(list = ls())         # Clear environment
#p_unload(all)           # Clear packages
#cat("\014")             # Clear console
```
