+++
categories = ["Development"]
date = "2017-01-10T15:17:18Z"
description = "Using open data on house prices from the UK Land Registry to analyse impact of Southern Rail industrial action"
draft = false
tags = ["development", "data", "analytics", "r", "go", "csv", "scraping", "southern rail", "property prices"]
title = "Using data to identify the impact of Southern Rail industrial action"
image = "/post/southernrail/prices.png"

+++

I, like many others, have been affected by the ongoing industrial dispute over Driver Only Operation (DOO) on Southern Railways.  On some days this amounts to delayed or cancelled trains with extended journey times and the inconvenience of standing all the way into London and on others, like today, strikes leave no viable way of getting to work in London at all.

There have been many attempts to articulate the impact of the industrial action such as the use of the [#todayimissed](https://twitter.com/hashtag/todayimissed) hashtag on Twitter (see below), a tounge-in-cheek [video game](http://www.bbc.co.uk/news/uk-england-36825721) and [a recent passenger survey](http://www.huffingtonpost.co.uk/entry/southern-rail-strike-survey_uk_58691a0ee4b0f24da6e921bb) conducted by The Association of British Commuters.  I have heard tales of people losing or missing out on jobs due to continued lateness or based on where they live and, more recently, quite a lot of people moving job or house so they avoid Southern Rail for their commute to/from work.  This got me thinking and I started to wonder whether there was any correlation between the industrial action and property prices in the affected areas.

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/hashtag/todayimissed?src=hash">#todayimissed</a> saying goodnight to my children again <a href="https://twitter.com/hashtag/southernfail?src=hash">#southernfail</a> <a href="https://t.co/hI1dCjLRid">pic.twitter.com/hI1dCjLRid</a></p>&mdash; James Bowman (@JameseBowman) <a href="https://twitter.com/JameseBowman/status/778504806264672256">September 21, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>  

In recent years, the UK government has starting publishing lots of their data, making it freely available to the public and developers to use for third-party applications.  I visited [www.gov.uk](https://www.gov.uk/government/statistical-data-sets/uk-house-price-index-data-downloads-october-2016) and found the UK Land Registry data for house prices available for download as a CSV file.  This data set gives us the monthly average price of completed property transactions for each local authority.  Unfortunately, this is a 'trailing' indicator as most property transactions take months to complete but it is a good place to start.  The data is currently only available up until October 2016 with further data added each month (November's data will be available later this month on the 17th January, December's in February, etc.).  

After downloading the Land Registry data and some [basic manipulation][code] in [R](https://www.r-project.org/about.html) and using [OpenHeatMap](http://www.openheatmap.com/) we can visualise the geographical distribution of property prices (transactions) across Great Britan (as at October 2016).  In this heatmap, the darker areas correspond to higher average property prices in transactions during October and conversely, lighter areas correspond to lower average prices.

<a href="/post/southernrail/uk-house-prices-10-2016.png">
	<img src="/post/southernrail/uk-house-prices-10-2016.png"" alt="UK House Prices 10/2016" class="pure-img">
</a>

We can see from the visualisation that the average house price in and around London is higher than other parts of Britan.  As Southern Rail extends into London it seems likely the average property price in the area served by Southern Rail will be higher than the national average.  For our analysis using the trend of house price movements over time is going to be more useful than absolute values.

Now we have the property prices by area, we need a way of comparing the nationwide prices with those for the area served by Southern Rail.  We need to calculate the area served by Southern or, perhaps more simply, the local authorities which have train stations served by Southern Rail.  Hunting around the internet I struggled to find data on train routes or schedules until I stumbled across the data I needed on Wikipedia.  Unlike most other sources I found, Wikipedia lists the stations served by each train operating company, rather than just the stations they operate.  This is important, because Southern Rail trains serve a number of stations operated by London Underground and other operating companies.

<a href="/post/southernrail/wikipedia-screen-shot.png">
	<img src="/post/southernrail/wikipedia-screen-shot.png"" alt="Infobox Vcard data for each train station on Wikipedia (screenshot from Wikipedia)" class="pure-img">
</a>

Wikipedia has a page for each station which contains relevent detail in the infobox vcard on the right (shown in the screen shot above) including the local authority that we can use to cross reference the Land Registry house price data.  I did not fancy manually trawling wikipedia for the ~200 stations served by Southern so I wrote a simple [Go](https://golang.org/) program to crawl Wikipedia and scrape the data I needed.  

The result was a complete list of stations served by Southern and the associated local authority within which they reside.  The area covered by these local authorities is shown in the map below.

<a href="/post/southernrail/area-served-by-southern.png">
	<img src="/post/southernrail/area-served-by-southern.png"" alt="Area Served by Southern Rail (sourced from Wikipedia)" class="pure-img">
</a>

Unfortunately, there were a lot of inconsistencies in the data scraped from Wikipedia.  Some of the Local Authority names did not match those used in the Land Registry data e.g. `Ashford` and `Borough of Ashford` used in the Land Registry and Wikipedia respectively.  There were also some inconsistencies in the Local Authority names from wiki page to wiki page e.g. `Brighton and Hove` and `Brighton & Hove`.

<a href="/post/southernrail/data-inconsistencies.png">
	<img src="/post/southernrail/data-inconsistencies.png"" alt="Inconsistencies in data scraped from Wikipedia" class="pure-img">
</a>

Some basic string manipulation in our [R script][code] allows us to clean up the wikipedia data and remove any inconsistencies.  Once all of the inconsistencies have been removed and we are satisfied that every Local Authority in the stations dataset references a valid Local Authority from the Land Registry data, we are ready to cross reference the 2 data sets and visualise the results.  I have plotted the average property prices by transaction since January 2016 up to and including October 2016 across local authorities served by Southern Rail and across Great Britan as a whole as a comparison.  The resulting plot is shown below.

<a href="/post/southernrail/prices.png">
	<img src="/post/southernrail/prices.png"" alt="Comparison of average house prices nationwide and in Southern Rail Area" class="pure-img">
</a>

It is clear that, as we predicted, the average price for the area served by Southern Rail is higher than the national average.  The two lines track almost identically except that in October, whilst the national average price continues to rise (albeit slightly), the prices in the area served by Southern Rail declines.  To be specific, the decline in average property price transactions across the Southern Rail area between September and October in 2016 is £2446.66 or approximately 0.6%.

As with any results, they are open to interpretation and we must be careful to consider that this is only a single data point and a trend requires at least 3.  Furthermore, we could argue that although the dispute has been going on since April, it is still too soon to see any impact manifest in property transactions (as it is a trailing indicator).  Finally, and perhaps most importantly, we have identified a correlation and not causation and any change observed in property prices is not necessarily as a direct result of the industrial action.

Whilst it is still too early to draw any conclusions I have really enjoyed playing with the data and will be interested to see what happens in the coming months as more data becomes available.  The code used to process, cleanse, analyse and visualise the data is available on Github [here][code].  I hope you have found my experiences interesting, please share your own experiences or thoughts in the comments section below.

[code]: https://github.com/james-bowman/southerntrains