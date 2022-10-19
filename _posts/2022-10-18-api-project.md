# API's: Right As Rain 

## Introduction 

![drought](/assets/images/drought.png)

North America is in a crisis. A crisis that trumps all other crises we are experiencing. The west is enduring one of the longest droughts ever recorded, which is bringing many dire consequences. While this problem looms over our heads, many are stepping up to the plate to solve this issue of water usage. 

Currently, I am working with Dr. Heaton in the BYU Stat department to see how we can help farmers use water more wisely during this massive drought using machine learning. This topic of water scarcity in the west has made me curious about weather patterns and how much precipitation our area is getting. I scraped my dataset on weather measurements in Provo, Utah using an API key. So, let’s begin!

## Basics of APIs 

![Courier](/assets/images/mail.png)
	
Now, I know what you are thinking. When you read the word API or Application Programming Interface, it’s just an acronym that spells something that makes no sense. However, past the perplexing words, you can uncover an amazing tool that makes web scraping effortless. In summary, an API is a messenger between you and the database you are wanting to collect data from. You ask it for certain things, and it will grab those preferences from the database. Isn’t that so much nicer than reading HTML code and digging to find where to web scrape from?! With that excitement, we are going to use this interface to scrape weather from the website [Virtual Crossing], which houses weather data from all over. 
		

## Ethical?

First and foremost, I want to show that what I have collected was legal and ethical. Virtual Crossing’s website had no explicit statement that the database wasn’t allowed to be collected. Furthermore, they have a user-friendly query system to filter what you would like in your dataset. I believe it is safe to assume that they want people to collect the data they have. 


## Virtual Crossing

Virtual Crossing is one of the leading providers of weather data, which has an easy and low-cost system. What makes their system so painless is their query builder. Below is a screenshot of what the builder looks like. You can manually pick location, variables, and duration. When finished it creates a unique API URL for you to use based on your account API key they provide. Easy as that! 

![Virtual Crossing: Query Builder](/assets/images/VirtualCrossing.png)

## Code

Below is a piece of the code that contains the API URL. I am using Python with the library’s requests, pandas, and config. From here you can change, which location to look at and what days to range from. For your information, if you have a free account, you can only do a small query so make sure your range of dates is small. 

```
#Defining Parameters 
City = "Provo"
State = "Utah"
Begin_Time = "/2022-01-01"
End_Time = "/2022-10-16"

#Accessing hidden api key
API_Key = config.api_key


#Combining it all into the api url 
url = "https://weather.visualcrossing.com/VisualCrossingWebServices/rest/services/timeline/" + City + "%2C%20" + State + Begin_Time + End_Time + "?unitGroup=us&elements=datetime%2Ctempmax%2Ctempmin%2Ctemp%2Cfeelslikemax%2Cfeelslikemin%2Cfeelslike%2Chumidity%2Cprecip%2Cprecipprob%2Cwindspeed%2Ccloudcover%2Csolarradiation%2Csolarenergy%2Cuvindex%2Csunrise%2Csunset%2Cconditions&key=" + API_Key + "&contentType=json"

```

## Take The Data By Storm!

![Lightning](/assets/images/storm.png)

Now from here, I want to analyze and explore what Provo’s weather is like for this particular year. I hope you are excited as I am to discover what we can find with the computer knowledge we have! Look at the dataset and see what you can find in our your exploratory data analysis!


