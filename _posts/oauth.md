---
layout: post
title: OAuth 2.0 with the Google APIs Client Library for Python
categories: dev-resources
---

<h2>Reference Material</h2>
<ul>
	<li><a href="https://developers.google.com/api-client-library/python/guide/aaa_oauth">Google API Client Library for Python</a></li>
</ul>


<h4>What is OAuth 2.0?</h4>
It is the authorisation protocol used by Google APIs. 

<h4>What is OAuth 2.0 used for?</h4>
Used to provide authorised API access to allow you to access private user data. Before you can access the user data, the user must first grant your app permission to do so.

<h4>How does OAuth 2.0 work at a basic level?</h4>
When building your OAuth 2.0 authorisation, there are two key fields which will be implemented:
<ol>
	<li>Client ID; and</li>
	<li>Client secret</li>
</ol>

Using these two pieces of information, we can authenticate a user.

NB: There are three types of client IDs:
<ol>
	<li><a href="https://developers.google.com/identity/protocols/OAuth2WebServer">Web application client IDs</a></li>
	<li><a href="https://developers.google.com/identity/protocols/OAuth2InstalledApp">Installed application client IDs</a> (e.g. mobile and desktop apps)</li>
	<li><a href="https://developers.google.com/identity/protocols/OAuth2ServiceAccount">Server account client IDs</a></li>
</ol>


<h2>Implementing OAuth 2.0 in Python</h2>

<h4>Step 1: Create a Flow</h4>
{% highlight ruby %}
FLOW = OAuth2WebServerFlow 
import requests 
import csv # to export the scraped results to CSV (if desired)
{% endhighlight%}


<h4>Create a Project on the Google API Console</h4>
SET-UP THE API ON GOOGLE
1. Open the Library page of the console https://console.developers.google.com/apis/dashboard and create a new project (or open one up)
2. Click Create Credentials > OAuth client ID
3. Complete the form on screen and set the Web application to Python
	For testing, you can specify URIs that refer to the local machine (e.g. http://localhost:8080) as the redirect URI
4. After creating the credentials, download the client_secret.json file from the API Console and securly store the file in a location that only your app can access

Before you start implementing OAuth 2.0 authorization, we recommend that you identify the scopes that your app will need permission to access.


<h4>Install Dependencies</h4>
{% highlight ruby %}
# Google APIs client Library for Python
pip install --upgrade google-api-python-client

# The google-auth, google-auth-oauthlib and google-auth-httplib2 for user authorisation
pip install --upgrade google-auth google-auth-oauthlib google-auth-httplib2

# The Flask Python web app framework
pip install --upgrade flask

# The requests HTTP library
pip install --upgrade requests
{% endhighlight%}


<h4>Obtaining OAuth 2.0 Access Tokens</h4>

STEP 1: Set authorisation parameters
Creates an authorisation request.







--------------------------------
USING FLASK-DANCE WITH OAUTH 2.0

1. pip install flask-dance --> Install flask-dance
2. pip install flask-dance[sqla] --> Install the sqla extension
3. 




