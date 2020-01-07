# Movie-Analysis
Analyzing movies from the IMDB database

## Data 

There are two CSV files for data `movies.csv` which is the main file I worked with and contained most of the data, and `small_movies.csv` which I used mostly for testing and implementing new functions to work with `movies.csv`.

The `movie` dataset contains 1,868 movies while the `small movies` dataset contains only 2 movies. However, both datasets contain the same information for each movie which is:

* Title
* Year
* Rating
* Directors
* Actors
* Genres

For the title, actors, and directors categories in the dataset, the main csv files list them as their IDs. As a result, it is necessary to also use the mappings for these datasets which has dictionaries with IDs mapped to the names of the movies, actors and directors in the dataset.

## Functions

Finding the median value of a list of numbers - used for median rating:

```python
def median(nums):
    nums = copy.copy(nums)
    nums.sort()
    if len(nums) % 2 == 1:
        return nums[len(nums) // 2]
    else:
        v1 = nums[len(nums) // 2]
        v2 = nums[len(nums) // 2 - 1]
        return (v1+v2) / 2
```

Load a file that can be used to lookup names from IDs:

```python
def get_mapping(path):
    file = open(path, encoding='utf-8')
    file_reader = csv.reader(file)
    file_data = dict(file_reader)
    file.close()
    return file_data
```

Open csv file, organize data into list of dicts:

```python
def get_raw_movies(path):
    file = open(path, encoding='utf-8')
    file_reader = csv.reader(file)
    file_data = list(file_reader)
    file.close()
    header = file_data[0]
    file_data = file_data[1:]
    movies = []
    for i in range(len(file_data)):
        movie_data = {}
        movie_data['title'] = file_data[i][header.index('title')]
        movie_data['year'] = int(file_data[i][header.index('year')])
        movie_data['rating'] = float(file_data[i][header.index('rating')])
        movie_data['directors'] = file_data[i][header.index('directors')].split(',')
        movie_data['actors'] = file_data[i][header.index('actors')].split(',')
        movie_data['genres'] = file_data[i][header.index('genres')].split(',')
        movies.append(movie_data)
    return movies
```

Connect movies in csv file to their mappings:

```python
def get_movies(movies_path, mapping_path):
    movie_data = get_raw_movies(movies_path)
    mapping_data = get_mapping(mapping_path)
    for movie in movie_data:
        for key in movie:
            for mapping in mapping_data:
                if type(movie[key]) == list:
                    for name in movie[key]:
                        idx = movie[key].index(name)
                        if name == mapping:
                            movie[key][idx] = mapping_data[mapping]
                if type(movie[key]) == str:
                    if movie[key] == mapping:
                        movie[key] = mapping_data[mapping]
    return movie_data
```

Return a list of movies made in a certain year:

```python
def filter_movies_by_year(movies, year):
    movies = copy.deepcopy(movies)
    i = 0
    while i < len(movies):
        if movies[i]["year"] != year:
            movies.pop(i)
        else:
            i += 1
    return movies
```

Return the number of unique datapoints:

```python
def unique(dataset, datapoint):
    u = []
    for movie in dataset:
        for key in movie:
            if key == datapoint:
                for i in movie[key]:
                    if i not in u:
                        u.append(i)
    return len(u)
```

Bucketize movies based on a given key:

```python
def bucketize(movie_list, movie_key):
    movie_values = {}
    for movie in movie_list:
        for key in movie:
            if key == movie_key and type(movie[movie_key]) != list:
                if movie[key] not in movie_values:
                    movie_values[movie[key]] = []
                movie_values[movie[key]].append(movie)
            elif key == movie_key and type(movie[movie_key]) == list:
                for genre in movie[movie_key]:
                    if genre not in movie_values:
                        movie_values[genre] = []
                    movie_values[genre].append(movie)
                
    return movie_values
```

Plot a bar graph of a dict:

```python
def plot_dict(d, label="Please Label Me!!!"):
    ax = pandas.Series(d).sort_index().plot.bar(color="black", fontsize=16)
    ax.set_ylabel(label, fontsize=16)
```

Filter movies to only use movies made within the given timeframe:

```python
def filter_year(movie_list, beginning_year, end_year):
    if beginning_year == None:
        begin = 0
    else:
        begin = beginning_year
    if end_year == None:
        end = 2018
    else:
        end = end_year
    years = []
    bucket_year = bucketize(movie_list, 'year')
    for year in bucket_year:
        if begin <= year <= end:
            years.append(year)
    return years
```

Count the number of movies in a genre in a given timeframe:

```python
def bucket_counts(year_range, movie_key):
    bucket_genre = bucketize(movies, movie_key)
    buckets = {}
    for genre in bucket_genre:
        for movie in bucket_genre[genre]:
            for key in movie:
                if key == 'year' and movie['year'] in year_range:
                    if genre not in buckets:
                        buckets[genre] = []
                    buckets[genre].append(movie)
    for genre in buckets:
        buckets[genre] = len(buckets[genre])
    return buckets
```

Function used to sort by career span:

```python
def row_ranking(row):
    return row["span"]
```

Return a list of dicts mapping people to career span:

```python
def top_n_span(buckets, n):
    spans = buckets
    rows = []
    for name in spans:
        span = buckets[name]
        rows.append({"name": name, "span": span})

    rows.sort(key=row_ranking, reverse=True)
    
    if len(rows) > n:
        diff = n - len(rows)
        rows = rows[:diff]
    return rows
```
Function used to sort by rating:

```python
def row_rank(row):
    return row['rating']
```

Calculating the highest median rating using a given bucket (such as year or genre) as bucket, number of entries as n, and minimum needed by each category to be considered as minimum:

```python
def highest_med_rating(bucket, n, minimum):
    
    val_median = {}
    for val in bucket:
        for movie in bucket[val]:
            for key in movie:
                if key == 'rating':
                    if val not in val_median:
                        val_median[val] = []
                    val_median[val].append(movie[key])
    vals = []
    for val in val_median:
        med = median(val_median[val])
        count = len(val_median[val])
        if minimum == None:
            vals.append({'category': val, 'rating': med, 'count': count})
        elif count >= minimum:
            vals.append({'category': val, 'rating': med, 'count': count})


    vals.sort(key=row_rank, reverse=True)

    if len(vals) > n:
            diff = n - len(vals)
            vals = vals[:diff]
    return vals
```

## Data Analysis

#### The first three movies in `movies.csv` are:

Title | Year | Rating | Directors | Actors | Genres
----- | ---- | ------ | --------- | ------ | ------
The Big Wedding | 2013 | 5.6 | Justin Zackham | Robert de Niro | Comedy, Drama, Romance
The Affair of the Necklace | 2001 | 6.1 | Charles Shyer | Simon Baker, Jonathan Pryce, Adrien Brody | Drama, History, Romance
The Barefoot Executive | 1971 | 6.0 | Robert Butler | Kurt Russell, Joe Flynn, Harry Morgan, Wally Cox | Comedy, Family

#### And the last three are:

Title | Year | Rating | Directors | Actors | Genres
----- | ---- | ------ | --------- | ------ | ------
Fortitude and Glory: Angelo Dundee and His Fighters | 2012 | 7.2 | Chris Tasara | Angelo Dundee, George Foreman, Freddie Roach | Sport
Ivanhoe | 1952 | 6.8 | Richard Thorpe | Robert Taylor, George Sanders | Adventure, Drama, History
The Great Gatsby | 1949 | 6.6 | Elliott Nugent | Alan Ladd, Macdonald Carey | Drama

#### Movies produced in 1931:

Title | Year | Rating | Directors | Actors | Genres
----- | ---- | ------ | --------- | ------ | ------
Arizona | 1931 | 6.0 | George B. Seitz | John Wayne, Forrest Stanley | Drama, Romance
City Lights | 1931 | 8.5 | Charles Chaplin | Charles Chaplin, Harry Myers | Comedy, Drama, Romance
The Range Feud | 1931 | 5.8 | D. Ross Lederman | Buck Jones, John Wayne, Edward LeSaint | Mystery, Western

#### Movies produced in 1932:

Title | Year | Rating | Directors | Actors | Genres
----- | ---- | ------ | --------- | ------ | ------
Texas Cyclone | 1932 | 6.2 | D. Ross Lederman | Wallace MacDonald, Tim McCoy, Wheeler Oakman, John Wayne | Action, Western
Haunted Gold | 1932 | 5.5 | Mack V. Wright | Otto Hoffman, John Wayne, Duke, Harry Woods, Erville Alderson | Horror, Mystery, Western
Girl Crazy | 1932 | 6.3 | William A. Seiter | Bert Wheeler, Robert Woolsey, Eddie Quillan | Comedy
Hot Saturday | 1932 | 6.6 | William A. Seiter | Cary Grant, Randolph Scott, Edward Woods | Drama, Romance
Lady and Gent | 1932 | 5.7 | Stephen Roberts | Morgan Wallace, George Bancroft, Charles Starrett, James Gleason, John Wayne | Drama, Sport
The Big Stampede | 1932 | 5.8 | Tenny Wright | John Wayne, Noah Beery, Paul Hurst | Western
The Shadow of the Eagle | 1932 | 5.8 | B. Reeves Eason, Ford Beebe | John Wayne, Walter Miller, Kenneth Harlan | Crime, Drama, Mystery
Ride Him, Cowboy | 1932 | 5.4 | Fred Allen | Otis Harlan, John Wayne, Duke, Henry B. Walthall | Romance, Western
Smilin' Through | 1932 | 7.0 | Sidney Franklin | Fredric March, Leslie Howard, O.P. Heggie | Drama, Romance
The Hurricane Express | 1932 | 5.6 | J.P. McGowan, Armand Schaefer | Tully Marshall, Conway Tearle, John Wayne | Action, Adventure, Crime

### Unique function

Using the `unique()` function from above, I found that:

* There are 18 unique genres in the movies dataset
* There are 1,247 uniqure directors in the movies dataset

### Most Actors

The movie Shoulder Arms was the movie that had the most actors in the movies dataset

### Average Rating

In the dataset, all of the movies had an average rating of `6.401659528907912`. I found this using the code: 

```python
total = 0
for movie in movies:
    for key in movie:
        if key == 'rating':
            total += movie[key]
denominator = len(movies)
avg = total/denominator
avg
```
### Unique Actors in Dataset

Using the `bucketize()` function from above, I found that there are 2,605 unique actors in the Movies dataset.

### Movies per Genre

I used the code: 

```python
num_per_genre = {}
bucket = bucketize(movies, "genres")
for genre in bucket:
    num_per_genre[genre] = len(bucket[genre])
num_per_genre
```

In order to create a dictionary containing each genre and the number of movies that fell under that genre. Giving me the results:

Genre | # of Movies
----- | -----------
Comedy | 485
Drama | 1094
Romance | 352
History | 73
Family | 85
Mystery | 121
Thriller | 250
Action | 299
Crime | 357
Adventure | 283
Western | 226
Music | 38
Animation | 45
Sport | 48
Fantasy | 59
War | 99
Sci-Fi | 69
Horror | 85

By far the most popular genre in the dataset is Drama which appears in 1,094 movies

This data can be shown as a bar plot as shown:

bar plot 1

Now if looking at this data only for the years prior to 2000, the data remains proportionally similar to the full data. Slight changes can be seen as the number of Adventure, History, and Western films is higher, while the number of Action, Horror, and Thriller movies are lower. This can mainly be attributed to western adventure films being highly popular in the early years in the dataset, looking at the 1931 and 1932 tables above we see that nearly every film has the western genre. However, Horror and Thriller films became much more popular with audiences in more recent years. 

bar plot 2

These changes are much more apparent when we look at the bar graph for 2000 to the present as Action and Thriller are much higher.

bar plot 3

### Number of Movies by Year

Breaking down the number of movies made each year from 2000 to the present, there were a higher amount of movies produced in 2001, 2012, 2013, and 2015 while 2011 had much less movies than other years. I'm not considering 2018 in this case as the data does not have a complete list for movies in 2018. 

bar plot 4

### Number of Years Directed by Directors with at least 30 years of experience

There are 14 directors in the dataset with at least 30 years of experience directing. Stanley Kubrick has the most experience directing with 46 years. The full list is as follows:

Director | Years Directing
-------- | ---------------
Howard Hawks | 42
Charles Chaplin | 34
Henry Hathaway | 36
Stanley Kubrick | 46
Taylor Hackford | 32
Cecil B. DeMille | 30
Lee H. Katzin | 30
Richard Fleischer | 32
Sidney Lumet | 33
George Sherman | 33
John Huston | 30
Robert Siodmak | 30
Eldar Ryazanov | 31
Martin Ritt | 32

### Actors with most experience

Among the actors with at least 40 years of experience acting, Mickey Rooney had the longest span of years acting with 75 years. The ten actors with the longest span of acting are:

Actor | Acting Span
----- | -----------
Mickey Rooney | 75
Anthony Quinn | 61
George Burns | 60
Dean Stockwell | 53
Glenn Ford | 52
James Caan | 52
Robert Mitchum | 51
Kurt Russell | 50
Robert De Niro | 49
Marlon Brando | 49

### Highest Ratings

The three genres with the highest median rating were Animation with at rating of 7.3, History with 6.7, and War with 6.7

The best ten years for movies were:

Year | Rating
---- | ------
1921 | 8.3
1925 | 8.2
1919 | 7.5
1923 | 7.3
1962 | 7.2
1964 | 7.1
1957 | 7.0
1985 | 7.0
1976 | 7.0
1963 | 6.95

However, the five best years for movies where there were at least 10 movies in a year were:

Year | Rating
---- | ------
1962 | 7.2
1964 | 7.1
1957 | 7.0
1985 | 7.0
1976 | 7.0

The four best directors with at least 3 movies were:

Director | Rating
-------- | ------
Christopher Nolan | 8.5
Leonid Gayday | 8.4
Stanley Kubrick | 8.3
Sergio Leone | 8.3

Finally, the three best actors with at least 5 movies were:

Actor | Rating
-------- | ------
Henry Bergman | 8.2
Ioan Gruffudd | 8.2
Robert Lindsay | 8.2
