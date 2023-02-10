---
layout: single
title:  Webscraping Workouts Behind a Login screen
date:   2023-02-09 22:58:15 +0000
author: Noah

categories: 
  - Data Science
  - Web Scraping
tags: 
 - Web scraping

categories_label: "Categories: "

toc: true
---

Ever wanted a collection of workouts from a WOD subscription service contained into one txt file. Well I have cracked the key to scraping wodify. Building webscrapers are genuinely quite simple especially with the tools and resources available online. The Goal for this project is start a collection of workouts to be used in the future. If you want a look at the code it can be found on my [github](https://github.com/BranchManager/Chalk_scraper) page. I'll be going over in detail how I constructed the main parts of this. I won't be going over the functions. Again this code is in my github repositories. The Goal for this psot is to share how I started scraping a website that required a login. The website I scraped....well just take a look on github. The link is in the code. However you do need a paid subscription for this.

Here are the libraries in use:
- BeautifulSoup
- selenium
- datetime
- time

Most of the libraries are to handle webscraping but I do take advantage of the datetime library. You'll see why later.

### First things first.

When I write code I generally take things step by step. Firstly I like to see If I can load a browser and get to the webpage. If I can use the below code and get a browser to come up then we are good.
Notice the commented out section. If you are running this remotely then setting those options is a must. The driver will not be able to load a display. However since I usually was working on the local machine this wasn't needed. I took a while to research this when I was remoting into my machine. 
~~~ python
#options.headless   
#driver = webdriver.Firefox(options=options)
driver.get(url_link)
~~~
### How to log in
Next I had to sign in to verify I was going in the write path. How I do this is by using the id tags in the website as shown below.
![page](/assets/images/2023-02-09-wodify_scraper_Passwrdid.png)

Using that information I can find the xpath or finds by id. I chose the find id option for this. Once I have found the elementid I can send the string I need as my username and/or password using the send_keys function. Next I use the XPATH method to grab the signin button and click it to continue to the next page. Also notice how I get the window. Since I am building a collection of workouts we will be cycling through each page with a workout. That variable will give you an array though so you will have to specify the the first index. I use this window_handles variable to grab each window as the site goes to a new page. Also it is wise to call a sleep method after you navigate to each page. That way you give the page time to load up before loading the next page. 
~~~ python
oldwin=driver.window_handles
html = driver.page_source
element_usr = driver.find_element("id","Input_UserName").send_keys(email)
element_pass = driver.find_element("id","Input_Password").send_keys(psswrd)
sin_in = driver.find_element(By.XPATH,"//button[text()='SIGN IN']")
clicked = sin_in.click()

newwindow = driver.window_handles[0]
driver.switch_to.window(newwindow)
~~~

### ---

### Things that don't matter to log in but how I went about the rest of the project. 
Now, there are different types of workout for each day. So I loop though each day starting at a specific day, that being today, and then loop through each type of workout.

First I set the date and the number of days to loop through.
~~~ python
today = date.today()
duration = timedelta(days=1095)
~~~

Then we start looping through each date. This for loop is pretty much how you loop through each day after setting a starting date. The next_date function basically goes to the webpage and enters the next date to grab information from. Then goes to that page and grabs that window. That function is written in my repository. It's only a few lines long. 
~~~ python
for d in range(duration.days+1):
  day = today - timedelta(days=d)
  date_as_str = day.strftime("%m/%d/%Y")
  print("TODAYS DATE: "+date_as_str+" *****************************************************************************************************:   \n \n ")
  
  next_date(driver,date_as_str)
~~~

Within that loop I create another loop which goes through each type of workout for that day. Workout_type is just a list that contains the value for the specific workout type. Otherwise this loop works similarly like the loop before it.  The get_next_type_of_workout function functions similarly to the next_date workout. It will just enter the selector info for the new workout type, go the that page and get the new window information. The only function of note is the get_content method. That may be worth looking at in the repo. This is what scrapes the specific content from each page.
~~~ python
    for the_type in workout_type:
        print("\n")
        print("WORKOUT TYPE: "+workout_type[the_type]+" *****************************************************************************************************:   \n  ")
        get_next_type_of_workout(the_type,driver)

        content,title_date = get_page(driver)
        print("WORKOUT TITLE: "+title_date+" *****************************************************************************************************:   \n  ")

        #this will print the actual workout type
        print(content)
~~~