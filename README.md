# Analytics Divvy using R
## Purpose
Answer the question: How do annual members and casual riders user Cyclistic bikes differently?


## See HTML file Data Analytic
[Click here](https://46785dd9168f41f5b282d07a81a4a5ed.app.posit.cloud/file_show?path=%2Fcloud%2Fproject%2FDivvy_trips.html)


## Practice

1. Install package "tidyverse"

```bash
  install.packages("tidyverse")
```
2. Import data [Divvy_Trips_2019_Q1.csv](https://docs.google.com/spreadsheets/d/1uCTsHlZLm4L7-ueaSLwDg0ut3BP_V4mKDo2IMpaXrk4/template/preview?pli=1&resourcekey=0-dQAUjAu2UUCsLEQQt20PDA#gid=1797029090)

```bash
library(readr)
all_trip_2019 <- read_csv("Divvy_Trips_2019_Q1.csv")
```
3. Rename columns

```bash
library(dplyr)
all_trip_2019 <- rename(all_trip_2019
                   ,ride_id = trip_id
                   ,rideable_type = bikeid
                   ,started_at = start_time
                   ,ended_at = end_time
                   ,start_station_name = from_station_name
                   ,start_station_id = from_station_id
                   ,end_station_name = to_station_name
                   ,end_station_id = to_station_id
                   ,member_casual = usertype)

```
4. Change type of 2 columns ride_id, rideable_type to character

```bash
all_trip_2019 <- mutate(all_trip_2019, ride_id = as.character(ride_id)
                        ,rideable_type = as.character(rideable_type))
```

5. Create a new table name Data_trips_2019  just includes information is necessary

```bash
Data_trips_2019 <- all_trip_2019%>%
  select(ride_id, rideable_type, member_casual, 
         start_station_id, start_station_name,
         end_station_id, end_station_name)
```

6. Check Data_trips_2019

```bash
colnames(Data_trips_2019) # Name of columns
nrow(Data_trips_2019) # Number of rows
dim(Data_trips_2019) # Number of columms
summary(Data_trips_2019) # Information for each column (min, max, mean,...)
```

7. Change data in member_casual column

```bash
Data_trips_2019 <- mutate(Data_trips_2019,member_casual = recode(member_casual
                                                 ,"Subcriber" = "member"
                                                 ,"Customer" = "casual"))
```

8. Check data of member_casual column

```bash
unique(Data_trips_2019$member_casual)
```

9. Add date, month, day, year, day_of_week into Data_trips_2019 table

```bash
Data_trips_2019$date <- as.Date(all_trip_2019$started_at) #The default format is yyyy-mm-dd
Data_trips_2019$month <- format(as.Date(Data_trips_2019$date), "%m")
Data_trips_2019$day <- format(as.Date(Data_trips_2019$date), "%d")
Data_trips_2019$year <- format(as.Date(Data_trips_2019$date), "%Y")
Data_trips_2019$day_of_week <- format(as.Date(Data_trips_2019$date), "%A")
```

10. Add a column to calcuate ride_length and change type 

```bash
Data_trips_2019$ride_length <- difftime(all_trip_2019$ended_at,all_trip_2019$started_at)

Data_trips_2019 <- mutate(Data_trips_2019,
                          ride_length = as.numeric(ride_length))
```

11. Create new table name Data_trips_2019_v2 to remove bad data

```bash
Data_trips_2019_v2 <- Data_trips_2019 %>%
  filter(ride_length > 0 | start_station_name == "HQ QR")
```

12. Sort day_of_week column

```bash
Data_trips_2019_v2$day_of_week <- ordered(Data_trips_2019_v2$day_of_week, levels=c("Sunday", "Monday"
        ,"Tuesday", "Wednesday"
        ,"Thursday", "Friday", "Saturday"))
```
13. Calculate mean ride_length group by member_casual and day_of_week

```bash
aggregate(Data_trips_2019_v2$ride_length ~ Data_trips_2019_v2$member_casual + Data_trips_2019_v2$day_of_week,
          FUN = mean)
```
14. Calculate number of rides by type ride and visualization

```bash
library(lubridate) # to use function wday()
number_of_rides_by_type <- Data_trips_2019_v2 %>%
  mutate(weekday = wday(date, label = TRUE)) %>% #creates weekday field using wday()
group_by(member_casual, weekday) %>% #groups by usertype and weekday
  summarise(number_of_rides = n() #calculates the number of rides and average duration
            ,average_duration = mean(ride_length)) %>% # calculates the average duration
arrange(member_casual, weekday) #sort


library(ggplot2)
ggplot(data = number_of_rides_by_type)+
 geom_col(mapping = aes(x = weekday, y = number_of_rides, fill = member_casual, position = "dodge"))
```
17. Create a visualization for average duration

```bash
library(lubridate) # to use function wday()
number_of_rides_by_type <- Data_trips_2019_v2 %>%
  mutate(weekday = wday(date, label = TRUE)) %>% #creates weekday field using wday()
group_by(member_casual, weekday) %>% #groups by usertype and weekday
  summarise(number_of_rides = n() #calculates the number of rides and average duration
            ,average_duration = mean(ride_length)) %>% # calculates the average duration
arrange(member_casual, weekday) #sort
```
Create visual
```{r}
library(ggplot2)
ggplot(data = number_of_rides_by_type)+
 geom_col(mapping = aes(x = weekday, y = number_of_rides, fill = member_casual, position = "dodge"))
```
17. Create a visualization for average duration
```{r}
average_duration <- Data_trips_2019_v2 %>%
  mutate(weekday = wday(date, label = TRUE)) %>%
  group_by(member_casual, weekday) %>%
  summarise(number_of_rides = n()
            ,average_duration = mean(ride_length)) %>%
  arrange(member_casual, weekday)

  ggplot(data = average_duration)+
geom_col(mapping = aes(x = weekday, y = average_duration, fill = member_casual, position = "dodge"))
```
