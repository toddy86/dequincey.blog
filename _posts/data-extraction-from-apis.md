---
layout: post
title: Extracting Data from APIs with Python
categories: projects
---

Export daily health and fitness data from the Fitbit API for further analysis using Python.

<!-- more -->
<b>Project goal</b> 

Learn the basic concepts of how to extract data from an API, along with implementing an OAuth 2.0 authentication workflow.

<b>Lessons learnt</b>
<ul>
	<li>What OAuth 2.0 is</li>
	<li>How to implement an OAuth 2.0 flow</li>
	<li></li>
	<li>XXXX</li>
</ul>

<b>Technology Review</b>

XXXX

XXXX

XXXX

---
<h3>Step 1: Review the Fitbit API Documentation</h3>
Fitbit provide developers with an array of different options to work with them, including an SDK for development of watch faces and watch apps, as well as a web API. 

As I want to extract the daily health and fitness data, the <a href="https://dev.fitbit.com/build/reference/web-api/">web API</a> is the service I need to use.

Under the <a href="https://dev.fitbit.com/build/reference/web-api/">Web API Reference page</a> Fitbit lists out the processes needed to be completed under the Quick Start heading.

In summary, we need to:
<ol>
	<li>Sign up for a developer account</li>
	<li>Register our app with Fitbit</li>
	<li>Decide what sort of Oauth 2.0 authorisation flow we want to use (server, client or personal)</li>
	<li>Make a HTTP request to access data</li>
</ol>

The biggest challenges at this stage was understanding the difference between what Fitbit refers to as the Oauth Authentication Type (server, client or personal). There is not a lot of documentation around this, but this <a href="https://community.fitbit.com/t5/Web-API-Development/Intraday-data-now-immediately-available-to-personal-apps/td-p/1014524">Fitbit Community post</a> guided me to choosing the 'personal' type, as this will give me access to intraday time series data (i.e. granular data from the day).

As a newbie to OAuth 2.0, I needed to do some reading up on what exactly it was, how to implement it and so forth. The following links were extremely valuable in this process:
<ul>
	<li><a href="https://developers.google.com/api-client-library/python/guide/aaa_oauth"></a>Google API Client Libraries Guide for Python</li>
	<li></li>
</ul>

The next challenge was figuring out which version of OAuth 2.0 to implement (web application or server account). This took some research, but the end decision was web server for advanced security reasons and the use of HTTP requests.


---
<h3>Step 2: Become familar with the DOM of the website</h3>
Access the webpage you want to scrape and <a href="https://developer.chrome.com/devtools">inspect the DOM</a>. Spend some time becoming familiar with the structure of the DOM and the css attributes of the content you want to scrape.  

The example website I will scrape is the <a href="http://www.wsj.com/mdc/public/page/2_3022-intlstkidx-20180219.html">Yahoo Finance World Indicies</a> page.

---
<h3>Step 3: Create a Python script and import dependencies</h3>
Create a Python script with the .py extension (e.g. "webscraper.py") and import the required dependencies.

{% highlight ruby %}
# Import the required dependencies
from bs4 import BeautifulSoup 
import requests 
import csv # to export the scraped results to CSV (if desired)
{% endhighlight%}

---
<h3>Step 4: Parse the HTML into a BeautifulSoup object</h3>
Grab the website content and create a BeautifulSoup object
{% highlight ruby %}
# Get the webpage content using requests, with additional .text to convert the HTML into a text object
webpage = requests.get('http://www.wsj.com/mdc/public/page/2_3022-intlstkidx-20180219.html').text

# Create a BeautifulSoup object and parse the content using lxml 
soup = BeautifulSoup(webpage, 'lxml')
{% endhighlight %}

Check your work so far by printing the object
{% highlight ruby %}
# Print the soup object to check the website has been successfully grabbed. Use .preittify() to make the results easily readable
print(soup.prettify())
{% endhighlight %}


---
<h3>Step 5: Grab a single elements from the soup object</h3>
Start small and grab a single element from the page. This way we can test that our code works correctly before scaling it up and grabbing multiple elements.


Use the .find() method to find and store the parent element with all of the content you want to scrape.  

{% highlight ruby %}
# As there is more than 1 table in the DOM, we need to grab and store the parent element of the table we are interested in, class='mdcWide'.
mdcWide = soup.find('div', class_='mdcWide')

# Pass the table object into the .find() method abd grab the td from the table object
td = mdcWide.find('td')
{% endhighlight %}

Ensure you are regularly testing your code by printing the results to avoid bugs in the final code.


---
<h3>Step 6: Grab multiple elements from the HTML object</h3>
Once we have established that we are correctly  grabbing a single element from the page, we can then refactor our code and use the .find_all() method to grab all of the elements from the page. 

{% highlight ruby %}
# Create a for loop and uses the .find_all() method to grab all of the rows in the table stored in the mdcWide div
for row in mdcWide.find_all('tr'):
	
	# Try block to handle exceptions
	try:
		# Grab the row name (index name), which has a unique css class (class="text")
		index_name = row.find('td', class_='text').text
		
		# Find all td fields in the row (non-unique field)
		all_td = row.find_all('td')

		# Grab the close price, which is index [3] of the non-unique td fields
		close = all_td[3].text

	# Except clause to continue past any errors / rows which do not contain the index name or close price
	except Exception as e:
		continue

	# Print the results to check code
	print(index_name, close)
{% endhighlight %}

<p>Notes</p>
<ul>
	<li>The singular .find() method is used when finding the index_name as it has a unique class name of class="text"</li>
	<li>The plural .find_all() method is used when finding all of the td elements, as there are multiple, non-unique td elements in each row containing financial data</li>
	<li>The close price data is stored in the 3rd td index</li>
	<li>The printing of the results above is only done to test the code. In the next step we will store the results into a variable for writing to CSV.</li>
</ul>

<h3>Step 7: Exporting results to CSV</h3>
Create csv file and write to it. This syntax needs to be included <u>before</u> the for loop.
{% highlight ruby %}
# Create a csv_file variable for saving results to and set the file name. Set the arguement to write to the CSV ('w')
csv_file = open('web_scraper_results.csv', 'w')

# Create a csv_writer variable and pass in the csv_file to the writer
csv_writer = csv.writer(csv_file)

# Set the header row of the CSV to match the data being scraped
csv_writer.writerow(['Index', 'Close Price'])
{% endhighlight %}


{% highlight ruby %}
# As the last step in the for loop, write the results of each element scraped to the CSV file
csv_writer.writerow([index_name, close])

# At the very end of the script, close the CSV file to ensure memory is managed
csv_file.close()
{% endhighlight %}

---
<h3>Step 8: Run the final script</h3>
Either run the script directly from Sublime (Command + B) or run the script from your terminal.

{% highlight ruby %}
# Import the required dependencies
from bs4 import BeautifulSoup 
import requests 
import csv # to export the scraped results to CSV (if desired)

# Get the webpage content using requests, with additional .text to convert the HTML into a text object
webpage = requests.get('http://www.wsj.com/mdc/public/page/2_3022-intlstkidx-20180219.html').text

# Create a BeautifulSoup object and parse the content using lxml 
soup = BeautifulSoup(webpage, 'lxml')

# Create a csv_file variable for saving results to and set the file name. Set the arguement to write to the CSV ('w')
csv_file = open('web_scraper_results1.csv', 'w')

# Create a csv_writer variable and pass in the csv_file to the writer
csv_writer = csv.writer(csv_file)

# Set the header row of the CSV to match the data being scraped
csv_writer.writerow(['Index', 'Close Price'])


# As there is more than 1 table in the DOM, we need to grab and store the parent element of the table we are interested in, class='mdcWide'.
mdcWide = soup.find('div', class_='mdcWide')

# Pass the table object into the .find() method abd grab the td from the table object
td = mdcWide.find('td')

# Create a for loop and uses the .find_all() method to grab all of the rows in the table stored in the mdcWide div
for row in mdcWide.find_all('tr'):
	
	# Try block to handle exceptions
	try:
		# Grab the row name (index name), which has a unique css class (class="text")
		index_name = row.find('td', class_='text').text
		
		# Find all td fields in the row (non-unique field)
		all_td = row.find_all('td')

		# Grab the close price, which is index [3] of the non-unique td fields
		close = all_td[3].text

	# Except clause to continue past any errors / rows which do not contain the index name or close price
	except Exception as e:
		continue

	# As the last step in the for loop, write the results of each element scraped to the CSV file
	csv_writer.writerow([index_name, close])

# At the very end of the script, close the CSV file to ensure memory is managed
csv_file.close()
{% endhighlight %}

The final script I created does not 100% resemble the above instructions as it includes a refactoring for capturing data from multiple fields, replacing non-numerical data with zeros etc.

The final script can be downloaded <a href="https://github.com/toddy86/basic_webscraper.git">here</a>

---
<h3 id="bs_syntax">BeautifulSoup Syntax</h3>
Source: <a href="https://gist.github.com/yoki/b7f2fcef64c893e307c4c59303ead19a">https://gist.github.com/yoki/b7f2fcef64c893e307c4c59303ead19a</a>

<script src="https://gist.github.com/yoki/b7f2fcef64c893e307c4c59303ead19a.js"></script>

