Cyclist Data Analysis
This project involves analyzing cyclist data to understand usage patterns and trends. The analysis covers the duration of trips, types of bikes used, and user membership status, among other metrics. Various data visualization techniques are employed to present the findings.

Installation
To run this project, you need to have R installed on your system. Additionally, you need to install the following R packages:

r
Copy code
install.packages('tidyverse')
install.packages('janitor')
install.packages('lubridate')
install.packages('data.table')
install.packages('dplyr')
Loading the Packages
After installing the required packages, load them using the following commands:

r
Copy code
library(tidyverse)
library(janitor)
library(lubridate)
library(data.table)
library(dplyr)
Importing the Data
Import the cyclist data CSV files for each month of 2021:

r
Copy code
jan_2021 <- read_csv("Cyclist Data/202101-divvy-tripdata.csv")
feb_2021 <- read_csv("Cyclist Data/202102-divvy-tripdata.csv")
mar_2021 <- read_csv("Cyclist Data/202103-divvy-tripdata.csv")
apr_2021 <- read_csv("Cyclist Data/202104-divvy-tripdata.csv")
may_2021 <- read_csv("Cyclist Data/202105-divvy-tripdata.csv")
jun_2021 <- read_csv("Cyclist Data/202106-divvy-tripdata.csv")
jul_2021 <- read_csv("Cyclist Data/202107-divvy-tripdata.csv")
aug_2021 <- read_csv("Cyclist Data/202108-divvy-tripdata.csv")
sep_2021 <- read_csv("Cyclist Data/202109-divvy-tripdata.csv")
oct_2021 <- read_csv("Cyclist Data/202110-divvy-tripdata.csv")
nov_2021 <- read_csv("Cyclist Data/202111-divvy-tripdata.csv")
dec_2021 <- read_csv("Cyclist Data/202112-divvy-tripdata.csv")
Combining and Cleaning the Data
Combine all the data into a single dataset and clean it by removing empty rows and columns:

r
Copy code
# List all CSV files
temp <- list.files("G:/Database/R Codes/Cyclistic/Cyclist Data", full.names = TRUE, pattern = "\\.csv$")

# Read all files and combine them using bind_rows
combined_trip_data <- bind_rows(lapply(temp, read.csv))

# Clean data by removing empty rows and columns
combined_trip_data <- remove_empty(combined_trip_data, which = c("rows", "cols"))
Adding Trip Duration
Add columns for trip duration in minutes and hours:

r
Copy code
combined_trip_data <- combined_trip_data %>%
  mutate(
    start_time = as.POSIXct(started_at, format = "%Y-%m-%d %H:%M:%S"),
    end_time = as.POSIXct(ended_at, format = "%Y-%m-%d %H:%M:%S"),
    trip_duration_minutes = as.numeric(difftime(end_time, start_time, units = "mins")),
    trip_duration_hours = trip_duration_minutes / 60
  )
Calculating Metrics
Calculate average trip duration, count types of bikes used, and count user membership status:

r
Copy code
# Calculate average trip duration
average_trip_duration <- combined_trip_data %>%
  summarise(
    avg_duration_minutes = mean(trip_duration_minutes, na.rm = TRUE),
    avg_duration_hours = mean(trip_duration_hours, na.rm = TRUE)
  )
print(average_trip_duration)

# Count types of bikes used
bike_type_counts <- combined_trip_data %>%
  count(rideable_type)
print(bike_type_counts)

# Count user membership status
user_type_counts <- combined_trip_data %>%
  count(member_casual)
print(user_type_counts)
Saving the Data
Save the updated main dataframe with the calculated metrics to a CSV file:

r
Copy code
combined_trip_data <- merge(combined_trip_data, average_trip_duration, by = NULL)
combined_trip_data <- merge(combined_trip_data, bike_type_counts, by = NULL)
combined_trip_data <- merge(combined_trip_data, user_type_counts, by = NULL)
write.csv(combined_trip_data, "combined_trip_data_with_metrics.csv", row.names = FALSE)
Data Visualization
Use ggplot2 to visualize the data with various charts:

r
Copy code
library(ggplot2)

# Bar Chart - Number of Trips by User Type
ggplot(combined_trip_data, aes(x = member_casual)) +
  geom_bar() +
  ggtitle("Number of Trips by User Type") +
  xlab("User Type") +
  ylab("Number of Trips")

# Pie Chart - Proportion of Bike Types Used by User Type
bike_usage_by_user <- combined_trip_data %>%
  group_by(member_casual, rideable_type) %>%
  summarise(count = n())

ggplot(bike_usage_by_user, aes(x = "User Type", y = count, fill = member_casual)) +
  geom_bar(stat = "identity", width = 1) +
  coord_polar("y") +
  facet_wrap(~member_casual) +
  ggtitle("Proportion of Bike Types Used by User Type") +
  scale_fill_discrete(name = "Bike Type")

# Histogram - Distribution of Trip Duration by User Type
cleaned_trip_data <- combined_trip_data %>%
  filter(is.finite(trip_duration_minutes))

ggplot(cleaned_trip_data, aes(x = trip_duration_minutes, fill = member_casual)) +
  geom_histogram(binwidth = 5, position = "dodge") +
  ggtitle("Distribution of Trip Duration by User Type") +
  xlab("Trip Duration (minutes)") +
  ylab("Frequency") +
  scale_fill_discrete(name = "User Type")

# Line Graph - Average Trip Duration Over Time by User Type
combined_trip_data$date <- as.Date(combined_trip_data$start_time)

average_trip_duration_over_time <- combined_trip_data %>%
  group_by(date, member_casual) %>%
  summarise(avg_duration = mean(trip_duration_minutes, na.rm = TRUE))

ggplot(average_trip_duration_over_time, aes(x = date, y = avg_duration, color = member_casual)) +
  geom_line() +
  ggtitle("Average Trip Duration Over Time by User Type") +
  xlab("Date") +
  ylab("Average Trip Duration (minutes)") +
  scale_color_discrete(name = "User Type")

# Heatmap - Trip Count by Hour and User Type
combined_trip_data$hour <- format(combined_trip_data$start_time, "%H")

trip_counts_by_hour <- combined_trip_data %>%
  group_by(hour, member_casual) %>%
  summarise(count = n())

ggplot(trip_counts_by_hour, aes(x = hour, y = member_casual, fill = count)) +
  geom_tile() +
  ggtitle("Trip Count by Hour and User Type") +
  xlab("Hour of Day") +
  ylab("User Type") +
  scale_fill_gradient(low = "white", high = "blue", name = "Trip Count")
Conclusion
This analysis provides insights into cyclist behaviors and trends throughout 2021. By cleaning, transforming, and visualizing the data, we can better understand the factors influencing trip durations and bike usage patterns. The results are saved in a CSV file for further analysis or reporting.
