# Anti-Civilian Violence in Africa: A Statistical Analysis
Jason Andrews | Clark University | Spring 2024 | IDCE-296, Advanced Vector GIS. 

# Introduction

#### The Dataset
This project is an exploration of the anti-civilian violence [dataset](https://acleddata.com/curated-data-files/#regional
) maintained by the Armed Conflict Location & Event Data Project (ACLED). [ACLED](https://acleddata.com/) is a "non-profit, non-governmental organization" and the "leading source of real-time data on political violence and protest activity around the world." Theirs is widely considered the most comprehensive dataset on global conflict, and is regularly cited by major news outlets, including the New York Times, as in this recent [article](https://www.nytimes.com/interactive/2024/04/20/world/asia/myanmar-civil-war.html#top_questions) about the civil war in Myanmar. The ACLED database includes datasets on "Battles," "Explosions/Remote violence," "Protests," "Riots," "Strategic developements," and "Violence Against Civilians." This analysis uses the Violence against civilians dataset for the continent of Africa, though we do include some analysis of the "Battles" dataset as well.

The complete dataset on anti-civilian violence contains, as of March 2024 when we downloaded it, information on more than 327,000 incidents all over the world. The subset for Africa includes 104,000 incidents, going back to 1997.

#### Research Objective
The goal with this project is to analyze how hotspots of violence against civilians have shifted on the African continent over time, and explore if/how the nature of anti-civilian violence has changed. This will be accomplished through data analysis and visualization in the form of graphs and hot spot mapping. After continent wide analysis and visualization, will zoom in on the region of West Africa in order to have a more granular, subnational understanding of the data. 

# Preliminary Data Exploration
To begin with we'll do some exploration of the entire dataset, which includes every country in the world. We'll do this with Python, mostly using pandas and geopandas. See **FatalitiesAnalysis.ipynb**. The ACLED dataset is updated continuously, and the dataset used here was downloaded on March 22, 2024.

After aggregating total fatalities by country into a new table, we will add a "start_year" field, taken from this handy [chart](https://acleddata.com/acleddatanew/wp-content/uploads/dlm_uploads/2019/01/ACLED_Country-and-Time-Period-coverage_updatedFeb2022.pdf), (See **getStartDate.ipynb**.) which allows us to also calculate a 'years' field, and a 'count_per_year' field, which represents the **average number of civilian fatalities per country per year**. 
```python
# calculate number of years of data collection for each country
fatalities['years'] = current_year - fatalities['Start Year']
# use this to get count per year
fatalities['count_per_year'] = fatalities['fatalities'] / fatalities['years']
# Clean up a couple fields
fatalities['start_year'] = fatalities['Start Year']  # Change 'Start Year' to 'start_year'
fatalities.drop(columns=['Country', 'Start Year'], inplace=True)  # Drop 'Country' column
# Sort the DataFrame by 'count_per_year' in descending order
fatalities = fatalities.sort_values(by='count_per_year', ascending=False)
# Selecting the top 15 rows
top_30_fatalities = fatalities.nlargest(30, 'count_per_year')
top_30_fatalities = top_30_fatalities.reset_index(drop=True)
top_30_fatalities
```

Here are the top 30, ranked by average number of civilian fatalities per year:

![image](https://github.com/andrews-j/CivilianAnalysis_Africa/assets/26927475/b1d41b79-2e69-4451-aa9f-9745807226a4)

I suspect some readers may find this chart surprising. Are people aware of the  consistent level of violence in Mexico, or Brazil?

The lack of parity between countries in how long data has been collected presents an issue when analyzing this data.
Every country in mainland Africa goes back to 1997, Latin America starts in 2018, the rest of the world is a number of different dates.

For example-- Syria started in 2017. How much higher would the rate of civilian fatalities be there if it had started a couple years earlier, like most of the other countries in the Middle East, and included the bulk of their Civil War?

But we'll poke around a bit more. This snippet adds a cumulative fatalities by country field to the dataset:
```python
# Create cumulative fatalities field
df.loc[:, 'event_date'] = pd.to_datetime(df['event_date'])
# Calculate cumulative fatalities
df['cumFatalities'] = df.groupby('country')['fatalities'].cumsum()
```

Which allows us to make this plot, cumulative fatalities in the 15 countries with the highest annual rate of civilian fatalities. This is another case study in why it's analytically challenging to compare countries with different data collection start points:
![image](https://github.com/andrews-j/CivilianAnalysis_Africa/assets/26927475/3422ba88-e90b-47cc-b73f-22fcc715754d)

Some takeaways/thoughts on this chart:
- Is it a coincience that Nigeria and DRC were both quite peaceful for the same ~4 years in the mid 2000s?
- Iraq has generally stabilized
- Absolutely amazing and horrifying how consistent the rate of civilian death is in Mexico.
- At the risk of getting into politics-- ACLED goes through great pains to not include the deaths of combatants in this dataset. The point of the dataset is to estimate violence against civilians, not wartime combatant casualties. However there is one territory where they consider every death a civilian fatality. Even Hamas [admitted](https://www.reuters.com/world/middle-east/israels-six-week-drive-hit-hamas-rafah-scale-back-war-2024-02-19/) that they had lost 6000 fighters, as of mid February. It's odd, because ACLED otherwise takes them at their word regarding casualty numbers. Plus, the organization is well [aware](https://acleddata.com/knowledge-base/indirect-killing-of-civilians/) of the need to grapple with this issue. 
- The Darfur Genocide took place in Sudan from 2003-2005. It appears as though civilians were killed at roughly the same rate in Nigeria between 2014 and 2016 at the height of the Boko Haram insurgency.

Next I made a function to isolate and plot the steepest (deadliest) one year period for each country. 
A few examples:

![image](https://github.com/andrews-j/CivilianAnalysis_Africa/assets/26927475/d68c37f7-2a2a-463a-971c-c4c96f0b5e72)

![image](https://github.com/andrews-j/CivilianAnalysis_Africa/assets/26927475/3e113125-b9df-44e1-bfe4-804ddbb4575d)

My idea is to take the steepest one year period for each country, and use that time frame to do a query on the APIs of various major news outlets. This might allow us to see, controlled for fatalities, how much media attention is given to each conflict.


# Narrowing the Focus to Africa

From here on we will focus solely on Africa, analyzing spatial and temporal trends by country and region. 

### Incidents VS Fatalities

To start with, we can look at anti-civilian violent **incidents** (not fatalities) by region:

![Incidents_year_region](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/a083f9b8-6332-4ae7-8c2e-b8faa1ec95c2)

The number of incidents of violence against civilians in this dataset has absolutely exploded in the last decade or so. 

The next plot looks at civilian **fatalities** by region over time:

![fatalities_region_line](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/28867629-ddca-4f97-a4eb-4da20d53f45f)

They look quite different! And what is going on in Middle Africa and East Africa in the late 90s?

Based on those two graphs we can make a couple of general statements about the data:
- In the late 90s Middle Africa and Eastern Africa had relatively few incidents but very high numbers of fatalities
- West Africa has seen a sharp rise in number of fatalities in the last decade, though without a significant increase in the ratio of fatalities/per incident. 

These inferences are confirmed by calculating and plotting the average number of fatalities per incident by year, split by region.

![FatalitiesPer_ScatterPlot](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/3a2b2832-c831-4a93-afb9-6a6750880bd5)

The average number of fatalities per incident has trended down in pretty much every region, especially Middle Africa. This raises some questions about the data: 

Can this trend be explained largely by increased granularity of reporting? Is it the case that in the past only significant incidents made it into the dataset? 

This seems like a strong possibility. We may be able to answer this question by digging further.

We can also look at a comparison of incident count vs fatalities, by region:

![newplot(3)](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/8d550a6c-f938-4620-bff8-94d9cae48385)

Each dot represents the number of fatalities vs incidents in a region for one year. The dashed line is the 1:1 line. Any point above this line represents a year during which every incident resulted in, on average > 1 fatality in that region.  There are some significant outliers in Middle Africa in the top left quadrent of the graph. This we will have to investigate.

A quick manual examination of the data suggests that the outliers in the upper left quadrant of this plot, years with few incidents but more than 5k fatalities, are explained by several distinct periods of conflict captured by the dataset. 

The Middle Africa points contain data from the end of the Angolan Civil War, specifically the seige of Kuito, which lasted 18 months between 1998 and 1999. There are 10 events listed with 1000 casualties each, which is the maximum number for an individual incident in the dataset. This raises questions about how events are binned, how estimate are made, and how his affects the accuracy of this data at a statistical level. 

### Cumulative Fatalities

Going back to Python we can create a graph showing cumulative fatalities by year for each region.

![image](https://github.com/andrews-j/CivilianAnalysis_Africa/assets/26927475/4958b774-d610-499e-9caa-512a0efd0559)

I was surprised to see that even given the recent spike in West Africa, Eastern Africa still has more total fatalities. This is confirmed by a quick Summary Stats in ArcGIS Pro. 

![image](https://github.com/andrews-j/CivilianAnalysis_Africa/assets/26927475/52d11b9f-0d26-4639-b8c4-613e6bac9af8)

### Continent Scale Data Aggregation: Space Time Cube

One of the most effective ways to visualize such a large dataset, both in geographic and temporal scale, is with ArcGIS Pro's **"Space Time Cube"** feature. This tool has several parameters that effect how the date is visualized. This map aggregates incident count into 2 year, 200 mile bins. This is more of an aesthetic choice than anything based on statistical analysis.

The result is a map that contains quite a lot of information. 

![image](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/35905814-8f9a-46ec-bcd3-47d4054d10cc)

An interactive version of that space time cube map is available [here](https://www.arcgis.com/apps/dashboards/7c5297d514a84e52bb6968c1ee254407). I also brought in the ACLED layer on 'battles' in Africa, so the two may be compared and contrasted. Battles are events where both sides are armed. 

**Note:** If the link doesn't work at first press enter in the address bar.

Space time cubes have the advantage of condensing a large amount of information quite effectively. The disadvantage is there is no way to weight the aggregation by a give field, such as fatalities, which would be interesting. 

### Continent Scale Data Aggregation: Hot Spot Analysis

**Emerging Hot Spot Analysis** is a tool in ArcGIS Pro that provides a 2-D representation of a space time cube dataset, with unique symbology for several different types of hot and cold spot. 

Intially I used the same size blocks as the above space time cube. However, in two dimensions, this does not offer fine enough spatial resolution:

![image](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/0d041955-d428-44a7-ab60-cfe4a2e4f10d)

I settled on cubes of 25 miles x 25 miles for hot spot vizualization, which is 1/16th the size of the 200 x 200 cubes above. This captures relatively fine grained regional detail.

![image](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/b9511a56-3292-4bff-91d5-ed769d04cc01)

These Emerging Hot Spot Analysis visualizations are limited, like the space time cubes, by only aggregating the number of incidents, not fatalities. Additionally, in this data set incident count pretty much has only gone up, which may explain why there are no "former hot spots" represented here. 

# Digging in to one Region: West Africa

There are a lot of different analyses that could be done with this data, depending on what question(s) one is trying to answer. West Africa is a region that I have some familiarity with, so we will take a closer look at the data for this region.
To do this we will first need to divide the allAfrica data by region:

![image](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/37d866d9-2b24-4650-93af-645fd2291260)

And plot fatalities per year by country. 

![WA_fatalities_year_country](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/acef14ba-78f9-4278-ab29-f5f5681a2ece)

Because I am somewhat familiar with West Africa, something catches my eye looking at this chart. Or rather, the lack of something. Where are the Liberian and Sierra Leon civil wars? The Sierra Leon Civil War lasted from 1991 to 2002, and the second Liberian Civil War lasted from 1999 to 2003. They don't register on this plot. 
To be more sure we can view the data disentangled by country:

![WA_fatalities_country_grid](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/3cfa2aff-62e2-4742-aa8c-72f74d206912)

Barely even a blip is registered by either.
We can even pull out fatalities per year just for these two countries, to get precise numbers.

![image](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/2eb26c54-2c5c-4651-8093-9a9b21351713)

These numbers are inexplicably low. 
It takes less than a minute of internet searching to find information that suggests the data here is far from complete.

![Screenshot 2024-04-19 000844](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/cd1c5ba8-c02c-4e3b-aa5b-46ed05bb9801)
**Example 1: The seige of freetown, 1,000 civilian fatalities**


![image](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/536b9108-d0cf-4f60-b07c-ad095aeff623)
**Example 2: 1999 Freetown Massacre, at least 3,500 civilian fatalities.**

These aren't obscure events, or footnotes. They are central moments in two major West African civil wars. And they are individual examples of what must be dozens or hundreds of incidents that are not in this dataset. Between these two wars alone the dataset is probably missing close to 100,000 civilian fatalities. And this is just one area I am vaguely familiar with. How many other areas are missing data from major conflicts? The Ivoirian Civil War seems absent as well.

All of which reinforces the fact that ACLED has a very ambitious mission, and despite the best of efforts, this data will always be incomplete. Is it too incomplete to be useful? I will contend that this is in fact the case for certain time periods in West Africa, and probably certain time periods in other regions of the world, during which it is very difficult to get accurate or complete information. 

However, the vastly increased number of reported incidents per year would seem to suggest that the dataset is getting better with time, probably due to way mobile smart phones have permeated society, allowing better connectivity between communities, citizens, and journalists or other record keepers. It is also worth emphasizing that for all its flaws ACLED almost certainly remains the best dataset on world conflict on offer. 

## Incidents VS Fatalities

One thing I am curious about, at this stage, is the percentage of incidents in the dataset with 0 fatalities. The example of the Angolan Civil War, discussed in the previous section, would seem to suggest that earlier in the dataset only major casualty events were recorded.

We can calculate the percentage of incidents per year with 0 fatalities. I found this easier to do in Python:
```python
WA = df[df['region'] == 'Western Africa']
# Group the data by year and calculate the total number of entries for each year
yearly_totals = WA.groupby('year')['fatalities'].count()
# Group the data by year and calculate the number of entries with fatalities equal to 0 for each year
yearly_0 = WA[WA['fatalities'] == 0].groupby('year')['fatalities'].count()
# Calculate the percentage of entries with fatalities equal to 0 for each year
yearly_percent_0 = (yearly_0 / yearly_totals) * 100
# Print the resulting Series
yearly_percent_0
```
And we can plot it with a Pearson's coefficient.
![image](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/8e16a8c4-997a-4133-9b31-25d4ccfbd5e8)

A correlation of -0.70 means that there is in fact a statistically significant **decrease** in the percentage of events **without a fatality** over the timeframe of the study. There would seem to be two likely possible explainations: Either there is a higher threshold over time for what comprises/gets reported as an incident, or civilian targeting incidents are simply becoming more deadly on average, perhaps due to enhanced access to more lethal weaponry by governments, militias, or terrorist groups in the region. 

## Aggregating Data with Collect Events

Even confining our analysis to West Africa, we still have 26,000 incident points to analyze.
One way to go about visualizing all this data is with 'Collect Events', which will aggregate events by location

![image](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/c5d16cc0-87b6-4bb4-aedb-ad40ce0f5019)

Which allows us to create a map like this:

![image](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/977fc35e-1faa-4698-a148-0a8da6bc5746)

However, this probably leaves the viewer with more questions than answers regarding what exactly comprises a 'location.' Is a location a neighborhood? A city? Both, it turns out, according to the [Quick Guide to ACLED Data](https://acleddata.com/resources/quick-guide-to-acled-data/#s7):

"Locations are coded to named populated places, geostrategic locations, natural locations, or neighborhoods of larger cities. Geo-coordinates with four decimals are provided to assist in identifying and mapping named locations to a central point (i.e. a centroid coordinate) within that location. Geo-coordinates do not reflect a more precise location, like a block or street corner, within the named location."

## Aggregating Data by Administrative region

Perhaps a more effective way to represent this data is by aggregating at subnational administrative units, rather than individual 'location'. This elucidates regional trends within individual countries.

We can pull a West Africa [Administrative Boundaries Level 1](https://data.humdata.org/dataset/west-and-central-africa-administrative-boundaries-levels) from [Humanitarian Data Exchange](https://data.humdata.org/). 
The ACLED dataset already has an 'admin1' field, which means we can simply aggregate this data, and then join it to the admin1 boundaries shapefile. 

The one tricky thing here is that there are some repeat names of admin1 zones across West Africa, so the aggregation and join must be done by both the admin1 and country field.
While the latter could have been accomplished with a spatial join, it's not clear to me how I might have done the aggregation in ArcGIS Pro.
For details on how I did this see WestAfrica.ipynb, but here is the crux of it:

```python
fatalities_admin1 = WA.groupby(['country', 'admin1'])['fatalities'].sum().reset_index()
admin1_merged = WA_admin1.merge(fatalities_admin1, on=['country','admin1'], how='left')
incidents_admin1 = WA.groupby(['country', 'admin1']).size().reset_index(name='incident_count')# Print the summary
admin1_merged = admin1_merged.merge(incidents_admin1, on=['country','admin1'], how='left')
```

The result is a shapefile of admin1 boundries across West Africa with fields for **total incidents** and **total fatalities** in that region for the whole timeframe of the dataset. 

This data is a great use case for a bivariate color symbology, allowing us to quickly understand the ratio of fatalities to incidents by administrative region, and therefore where civilians are more likely to be faced with lethal force. 

![image](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/7beaa386-91a5-49ad-9849-73eb5b780df0)

Unsurpisingly, Northern Nigeria has the highest concentration of zones with both high incident numbers and high fatalities, and the Sahel has clusters of high incident activity too. A scatter plot of this data tells us basically the same thing:

![image](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/336b5ab7-b058-4cf4-84b7-909ecf1145a3)

This data can also be viewed as an interactive [web app](https://clarku.maps.arcgis.com/apps/instant/charts/index.html?appid=1158745ed81d46a19df9ca2a2658ec2d)

## Civilian Fatalities vs Battle Fatalities

ACLED also maintains a [dataset](https://acleddata.com/data-export-tool/) on the occurance of 'battles,' which I referenced and used in an interactive space time cube [map](https://www.arcgis.com/apps/dashboards/7c5297d514a84e52bb6968c1ee254407) earlier in this project readme. Battles are [defined](https://acleddata.com/resources/quick-guide-to-acled-data/) by ACLED simply as "Violent interactions between two organized armed groups." The key here being that _both_ sides are armed, though this could still be a street fight between gangs. As always I have questions about the completeness/validity of this data, but we will work with what we have.

The 'battles' data allows for an analysis of what percent of fatalities by region occur in battle, vs anti-civilian violence. 
Theoretically this will show what regions are more embroiled in outright conflict/war, vs where the violence is more one sided. 

To do this I created a ratio, similar in statistical method to NDVI, but with anti-civilian violence fatalities, vs fatalities in battle.

![image](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/5adc2600-ae2b-43ef-b903-422ed1def5fc)

This is another instance where things were much easier in Python, and it did not feel like good use of time to troubleshoot ArcGIS Pro.

![image](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/20e4bcd9-dc67-4bbd-a63b-2191d6d67d15)

The resulting map represents the ratio as bimodal data. 

Green = more civilian fatalities than combat
Blue = more battle fatalities than civilian

![image](https://github.com/andrews-j/CivilianAnalysis/assets/26927475/d42464a6-2cf5-4d5d-b559-6c52a27db0be)

I'm agnostic as to how useful this really is, or if this is an appropriate use of ratios. However, the map does illuminate real trends:
- There is more green in peaceful countries such as Senegal, Guinea, and Ghana.
- The borders of Burkina Faso can almost be distinguished by the collection of blue polygons. 

However, there are drawbacks:
- The magnitude of violence is totally lost here, and an area with lots of battle and anti-civilian violence fataliies is just tan.
- 27 years is a very long timeframe

# Conclusions and Further Research
I've used a handful of different approaches to analyze and visualize this data. It has been an exploratory exercise, and my research questions have emerged as I've proceeded, rather than having them from the beginning. 

These are some of my observations:

- The best visual for spatial-temporal trends in such a large dataset is probably an interactive space time cube. There's really no better way to represent 27 years of data for the entire continent, comprising > 100,000 points.
- While it is broadly useful, this is a highly flawed and incomplete dataset. The total lack of data on the Liberian and Sierra Leonian Civil Wars are especially galling, but these are just examples. GIGO.
- On that point, specialized/contextual knowledge is (as always) crucial to draw insight from this kind of data.
- There are an infinite number of ways to analyze and represent this data.
- I encountered several things that were difficult or even impossible to do in Arc GIS Pro that could be accomplished quickly in Python.

## Further Work:

- Continuing to hone in on smaller subsets of the data, both spatially and temporally, would lead to better and more meaningful representations of the data. Particularly, focusing on trends within the last 5-10 years.
- Different ways to slice the data. It would be interesting to do an analysis of the ratio of anti-civilian incidents to fatalities by **actor** to see which force has the most lethal encounters with civilians.
- It may also be possible to slice the data by motive, using some kind of keyword in the notes.
