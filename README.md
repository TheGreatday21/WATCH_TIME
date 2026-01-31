# WATCH_TIME

Watch time is juptyer script that enables me to see how many hours i spent watching a particular show on Netflix and the trend of how often plus which days i watch this show the most.


## Steps
### Download Your Netflix Data
To grab your own, make sure you're logged in to Netflix . From the main Netflix screen click your account icon in the top right, click "Account", and then click "Download your personal information" on the page that loads.On the next page you should see "Request a copy of your information "
Netflix will then say preparing your data and this process may take up to 30 days for you to receive your information.

### Understand your data
For this particular small project our main focus will be on the Content_Interaction folder containing the ViewingActivity.csv file which is a log of everything you have watched on netflix.

From the Coversheet pdf we notice that the StartTime column uses the UTC timezone which we will have to change to match our local timezone .

### Exploring the dataset
First run the command in terminal to install dependencies 
   ```bash
   pip install -r requirements.txt
   ```
<br>
Then open the notebook , and  in a new code cell with python kernel , import the pandas library, the matplot lib library ,and your viewing activity dataset

    ```
        import pandas as pd
        import matplotlib.pyplot as plt
        df = pd.read_csv('ViewingActivity-sample.csv')
    ```
<br>
We can then carry out alittle exploration to understand the variables in our dataset from how many they are to their datatypes 

    ```
        df.head(5) #this shows you the first columns of your dataset
        df.shape() #show us how many rows and columns are in the dataset
    ```

### Preparing for analysis
For this particular project i only need the Start Time, Duration, and Title columns since iam just analysing how much and when ive watched a particular show . We use the drop function to drop the rest

    ```
        df = df.drop(['Profile Name', 'Attributes', 'Supplemental Video Type', 'Device Type', 'Bookmark', 'Latest Bookmark', 'Country', axis=1])
        df.head() #to ensure see our remaining columns 
    ```
<br>
To see which particular datatypes each column has we use the dtypes function. We notice the duration and start time are in obj format which wont favor our calculation so we convert them using pandas to suitable formats . Datetime for Starttime and timedelta for Duration

    ```
        df['Start Time'] = pd.to_datetime(df['Start Time'], utc=True)
        df['Duration'] = pd.to_timedelta(df['Duration'])
        print(df.dtypes)

    ```

<br>
To convert the Starttime to my local time i have to first turn the start time into an index using the set index function, change the time zone with the tz convert function  then  revert it back to a regular column using the reset index fucntion 

    ```
        # change the Start Time column into the dataframe's index
        df = df.set_index('Start Time')
        # convert from UTC timezone to east african time
        df.index = df.index.tz_convert('Africa/Nairobi')
        # reset the index so that Start Time becomes a column again
        df = df.reset_index()
        #double-check that it worked
        df.head()
    ```
<br>    
Now that our data types are fixed we can proceed to analysis.

### Chosing a show to analyse
We proceed by creating a new dataframe for a specific show to  see how much time is spent watching this show and when i watch this show 
Used the str.contains function that takes 2 arguments. The first is the string and the second is regex , which tells the function that the previous argument is a string and not a regular expression.
    ```
        show_i_like = df[df['Title'].str.contains('show_i_like', regex=False)]
        show_i_like.head() #to see if you have succeeded 
    ```

### The Analysis
From the data we notice that even < 1 minute previews are counted as  episodes watch so we are filtering further to show those with a duration greater than 3 minutes atleast.
    ```
        show_i_like = show_i_like[(show_i_like['Duration'] > '0 days 00:01:00')]
        show_i_like.shape
    ```
<br>
We can now simply run the sum function on the dataframe and see how much time weve spent watching this particular show 
    ```
        show_i_like['Duration'].sum() #this will show you the amount of time you spent on that particular show 
    ```
<br>
To answer the question of when you watched this show the most, we first need to engineer 2 additional features from the start time column i.e weekday and hour . We will be using the .dt.weekday and .dt.hour methods on the Start Time column and assigning  the results to new columns named weekday and hour.

    ```
        show_i_like['weekday'] = show_i_like['Start Time'].dt.weekday
        show_i_like['hour'] = show_i_like['Start Time'].dt.hour

        # check to make sure the columns were added correctly
        show_i_like.head()
    ```
<br>
We will now use visuals from matplotlib to see 
-  On which days of the week have I watched the most Office episodes?
-  During which hours of the day do I most often start Office episodes?

### Visualisations
#### **By day** 
For a more understandeable visual, convert the order of ploting to ascending order
    ```
        show_i_like['weekday'] = pd.Categorical(show_i_like['weekday'],         categories= [0,1,2,3,4,5,6],ordered=True)
    ```
Creating a new feature to store the count of episodes watched per day 
    ```
        show_by_day = show_i_like['weekday'].value_counts()
    ```
Sort the index using our categorical, so that Monday (0) is first, Tuesday (1) is second, etc.
    ```
        show_by_day = show_by_day.sort_index()
    ```
A plot of the watch frequency by day 
    ```
        show_by_day.plot(kind='bar', figsize=(20,10), title='Episodes Watched by Day',xlabel='Day of the Week (Monday - Sunday)', ylabel='Number of Episodes Watched')
    ```
#### **By time of day** 
We define the order of plotting for hours. Using the 24 hour system 
    ```
        show_i_like['hour'] = pd.Categorical(show_i_like['hour'], categories=
        [0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23],ordered=True)
    ```
Creating a new feature to store the count of  rows for each hour
    ```
        show_by_hour = show_i_like['hour'].value_counts()
    ```

Sort the index in the desired order of our categorical  so that midnight (0) is first, 1 a.m. (1) is second, etc.
    ```
        show_by_hour = show_by_hour.sort_index()
    ```
Plot show_by_hour as a bar chart with the listed size and title
    ```
        show_by_hour.plot(kind='bar', figsize=(20,10), title='Episodes Watched by Hour',xlabel='Hour of the Day (0-23)', ylabel='Number of Episodes Watched')
    ```







