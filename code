Data source: https://data.ibb.gov.tr/dataset/ibb-istac-araclarinin-anlik-konum-ve-hiz-bilgileri

# The data contains the first month of the relevant year.

pip install geopy
pip install tqdm
pip install data-profiling

import numpy as np
import pandas as pd
from datetime import datetime, timedelta
import os
import re
from geopy.geocoders import Nominatim
from tqdm import tqdm

# =============================================================================
# 1) File directories
# =============================================================================

starting_time = datetime.now()
path = r'your_path/'
file_list = [i for i in os.listdir(path) if i.endswith('.csv')]

# =============================================================================
# 2) Import files
# =============================================================================

# Appending dataframe as a list is faster than appending itself. At least this is what I've experienced so far.

df = []
for file in file_list:
    if file == '01.2019_1-10.csv.csv':
        temporary_df = pd.read_csv(path + file, delimiter = ',', usecols = [1, 2, 3, 4, 5, 6, 7, 8, 9])
        df.append(temporary_df)
    else:
        temporary_df = pd.read_csv(path + file, delimiter = ',')
        df.append(temporary_df)
df = pd.concat(df)
df.drop(columns = ['day_day', 'day_month', 'day_year'], axis = 1, inplace = True)

# =============================================================================
# 3) Check
# =============================================================================

# No missing values. For the moment, I will not include vehicles have 0 speed. I will consider them after a while.

df.info()
df.isnull().sum() 
df['day_hour'] = pd.to_datetime(df['day_hour'], errors = 'coerce', format = '%Y-%m-%d %H:%M:%S')
df = df[df['speed'] != 0]

# We have 14 unique observations. The rest of them are duplicated.

len(df.drop_duplicates(subset = df.columns.to_list())) - len(df) / 2

df.drop_duplicates(subset = df.columns.to_list(), inplace = True)
df.reset_index(drop = True, inplace = True)

df.sort_values(by = ['day_hour', 'vehicle_type', 'speed'], 
               ascending = [True, True, True], 
               na_position = 'last', 
               inplace = True)
df.reset_index(drop = True, inplace = True)
print(df['day_hour'].min(), df['day_hour'].max())

# =============================================================================
# 4) Time Intervals
# =============================================================================

# Let's decide which time intervals we will take a look for car speed. There will be four parameters.

def participation_interval(file, kickoff, deadline, hourly_change):
    main_data = pd.DataFrame()
    survey_start = datetime.strptime(kickoff, '%Y-%m-%d %H:%M:%S')
    survey_finish = datetime.strptime(deadline, '%Y-%m-%d %H:%M:%S')
    groups = 1
    time_dictionary = {}
    while survey_finish > survey_start:
        final_df = np.where((file['day_hour'] >= survey_start) & (file['day_hour'] < survey_start + timedelta(hours = hourly_change)), 
                              groups, 0)
        final_df = pd.DataFrame(final_df).rename(columns = {0: 'time_interval'})
        final_df = final_df[final_df['time_interval'] != 0]
        final_df = pd.merge(file, final_df, how = 'inner', left_index = True, right_index = True)
        main_data = main_data.append(final_df)
        time_dictionary.update({groups: str(survey_start) + ' ile ' + str(survey_start + timedelta(hours = hourly_change)) + ' arası'})
        survey_start += timedelta(hours = hourly_change)
        groups += 1
    return main_data, time_dictionary
df = participation_interval(df, '2019-01-01 00:00:00', '2019-02-01 00:00:00', 1)

my_dictionary = dict(df[1]) # I will just keep it as my dictionary.
df = df[0]
for groups, description in my_dictionary.items():
    df['time_interval'] = df['time_interval'].replace(groups*1, description)

average_speed_time_interval = df.groupby(['time_interval', 'vehicle_type'])['speed'].mean().reset_index()
average_speed_time_interval.to_excel(os.path.join(os.path.dirname(path), 'car_speed_time_interval.xlsx'), 
                                     header = True, index = False, sheet_name = 'car_speed')

finish_time = datetime.now()
duration = finish_time - starting_time
duration.seconds/3600

# =============================================================================
# 5) Average Car Speed According to the Locations & Vehicle Type
# =============================================================================

# Let's take a sample from df.
test = df.sample(frac = 0.001)

test['vehicle_type'].value_counts()
test['latitude'] = test['latitude'].astype(str)
test['longitude'] = test['longitude'].astype(str)

test.drop(columns = test.columns.to_list()[-3:], axis = 1, inplace = True)

# In order to get the locations, I need comma between longitude and latitude.
test.insert(loc = 1, column = 'long_lat', value = test['longitude'] + ',' + test['latitude'])

location_finder = Nominatim(user_agent = 'http', timeout = 70000)

test['locations'] = test['long_lat'].apply(lambda x: location_finder.reverse(x))
test.drop(columns = ['geohash', 'longitude', 'latitude'], axis = 1, inplace = True)
test['locations'].head(10)

##### The addresses are not in order. Some addresses start with neighborhood, some starts with street, some starts with avenue.
# Therefore, I am still working on this part. I will upload the code once I finish.
