---
title: "Academic Internship Assignment: R Code"
excerpt: ""
collection: portfolio
---
# Introduction

This R code represents the core of my assignment, which I've developed during my academic internship at the Royal Danish Embassy in Tokyo, Japan. The project focuses on the aging population in Japan, a critical demographic issue with profound social and economic implications. My assignment aims to delve into the complexities of this phenomenon by analyzing various datasets to understand the underlying dynamics better.

```r
# Preamble ##### 
library("pacman")     
p_load(dplyr,tidyr,stringr,ggplot2,rio,lubridate, plotly)

# Define the path variable
path_to_workspace <- "/Users/olivernyropweeks/Library/CloudStorage/OneDrive-UniversityofCopenhagen/KUonedriveOW/8. Semester - Praktik/Internship Assignment/Workspace/"

# Import first dataset
file_path <- paste0(path_to_workspace, "WorldBankJapanAge.xls")
df <- import(file_path, skip = 2)

# Filter the data for Japan
countries_of_interest <- c("Japan"
                           , "Denmark"
                           , "China"
                           , "Germany"
                           , "India"
                           , "United Kingdom"
                           , "France"
                           , "Italy"
                           , "Brazil"
                           , "Canada"
                           
                           )  # Add more country names as needed

selected_data <- df %>%
  filter(`Country Name` %in% countries_of_interest)


# Convert the data from wide to long format
selected_long <- selected_data %>% 
  pivot_longer(
    cols = matches("^\\d{4}$"),
    names_to = "Year",
    values_to = "Percentage65Plus"
  ) %>% 
  mutate(Year = as.numeric(Year)) # Convert Year to numeric if it's not already

plot <- ggplot(data = selected_long, aes(x = Year, y = Percentage65Plus, color = `Country Name`)) +
  geom_line(size = 1.5) +
  labs(
    x = "Year",
    y = "Population Age 65 & Above (%)",
    title = "Population Age 65 & Above in Selected Countries Over Time"
  ) +
  scale_x_continuous(
    breaks = seq(from = min(selected_long$Year), to = max(selected_long$Year), by = 5) # Ticks every 5 years
  ) +
  theme_minimal() +
  theme(
    text = element_text(size = 12),
    axis.title = element_text(size = 14),
    axis.text = element_text(size = 12),
    plot.title = element_text(size = 16, hjust = 0.5)
  )

print(plot)
Pic_path1 <- paste0(path_to_workspace, "JapanOver65.pdf")
ggsave(Pic_path1, plot = plot, width = 10, height = 6, dpi = 300)

# CLEAR ####
#rm(list = ls())         # Clear environment
#p_unload(all)           # Clear packages
#cat("\014")             # Clear console

```
