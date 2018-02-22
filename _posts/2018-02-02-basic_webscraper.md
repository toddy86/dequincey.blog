---
layout: post
title: Web Scraper with Beautiful Soup
categories: projects
---

INSERT DESRIPTION OF WHAT THE PROJECT GOAL/OBJECTIVE WAS AND THE RESULTS 

LEARNING AND BUILDING: TOTAL TIME 
EST. TIME TO DO AGAIN: LESS THAN TOTAL LEARNING TIME
RESOURCES: https://www.youtube.com/watch?v=ng2o98k983k
LESSONS LEARNT:
CREATE A BS CHEATSHEET

CHEATSHEET ITEMS TO INCLUDE
1. requests.get('website') # grab a website 
2. .text .grabs the text from the soup or converts to text
3. .title #grabs the tile from the soup
4. .prettify() # converts a HTML string into typical layout / pretty layout
5. .find('what you want to find') # finds an element which meets a certain criteria
6. .find_all()
7. .['attr_name']
8. .split() --> split a string based on a specific character (e.g. .split('/') )
9. Indexing after a split or return of multiple items done with the typical [3] syntax 


<!-- more -->

<h3>Step 1: Install Required Packages</h3>

All of the below (except for Python) come pre-installed with the Anaconda package, but they can be manually installed via pip if required.

<ul>
	<li>Python</li>
	<li>BeautifulSoup4</li>
	<li>lxml</li>
	<li>requests</li>
</ul>

<h4>Python</h4>
Who doesn't have Python installed?!

<h4>BeautifulSoup4</h4>
Must ensure that you have <u>BeautifulSoup4</u> installed and NOT BeautifulSoup.

{% highlight ruby %}
pip install BeautifulSoup4
{% endhighlight %}

<h4>lxml</h4>
{% highlight ruby %}
pip install lxml
{% endhighlight %}

<h4>requests</h4>
{% highlight ruby %}
pip install requests
{% endhighlight %}



<h3>Step 2: Become familar with DOM structure of the website</h3>
The example website I will be scraping is the Yahoo Finance World Indicies page.

http://www.wsj.com/mdc/public/page/2_3022-intlstkidx-20180219.html

Go to Chrome or other browser of choice and Inspect Element the page and become familiar with the layout of the DOM and the information you would specifically like to scrape. 




<h3>Step 3: Create a Python script and import dependencies</h3>
Create a Python script with the .py extension - in my example I have called the script "webscraper.py"

Then import the required dependencies
{% highlight ruby %}
from bs4 import BeautifulSoup
import 
import csv
{% endhighlight%}




<h3>Step 4: Parse HTML into BeautifulSoup</h3>
Next we need to parse in the HTML into BeautifulSoup (BS) so that we can createa BS object. 

{% highlight ruby %} 
# Get the webpage content using requests, with additional .text to convert the HTML into a text object
webpage = requests.get('http://www.wsj.com/mdc/public/page/2_3022-intlstkidx-20180219.html').text

# Parse the text object to create a BeautifulSoup object
soup = BeautifulSoup(webpage, 'lxml')
{% endhighlight %}

To print the results to ensure the scraper is working as expected, print the soup object and use the prettify() method to format the object so it is readable. 

{% highlight ruby %}
print(soup.prettify())
{% endhighlight %}



<h3>Step 5: Grab a single elements from the HTML object</h3>
Before we grab mutliple elements from a page, the best place to start is by grabbing just one of the element first. This way we can test that our code works correctly before grabbing multiple versions of the element (as this could be hundreds, thousands or tens of thosands of elements).

As there are multiple tables on the page, we need to be more specific than just using table as the find parameter. 


--------------------------------------
TO TIDY UP
Notes:
	- .find() --> class_='', but the underscore is only required for class, as that is a special keyword in python
	- .find('div', id='grab_this_id')
	- .find('div', class_='grab_this_class')
--------------------------------------

In the below we are going to grab and store the table from the HTML site. Then we will complete more grabs on the data we are storing int he table variable

{% highlight ruby %}
# Grab the div with the class='mdcWide' and store this as an object NB: the class arguement must be input as class_, and not just class, as this is a special keyword in Python
mdcWide = soup.find('div', class_'mdcWide')

# Pass the mdcWide object into the .find() method and grab the entire table from the HTML object (unnecessary step, but shown for step by step instructions)
table = mdcWide.find('table')

# Pass the table object into the .find() method abd grab the td from the table object
table_data = table.find('td')

{% endhighlight %}

To test your code, simply use the print() function to print the results of the variable (e.g. row) and review the results.



<h3>Step 6: Grab multiple elements from the HTML object</h3>
Once we have established that we are grabbing the correct single element, we can then use the .find_all() method to grab all of the elements on the page which meet our criteria. 

{% highlight ruby %}
# Create a for loop, which uses the .find_all() method and loops over the table_data results from before and prints out the inner text
for table_data in table.find_all('td'):
	results = table_data.text
	print(results)
{% endhighlight %}

The above will print all of the results as separate line items. This is only done to test your code as you go / show you the results. We will store the results into a variable for output later.




<h3>Step 7: Exporting results to CSV</h3>

{% highlight ruby %}

{% endhighlight %}

<h3></h3>
{% highlight ruby %}
{% endhighlight %}

<h3></h3>
{% highlight ruby %}
{% endhighlight %}

<h3></h3>
{% highlight ruby %}
{% endhighlight %}