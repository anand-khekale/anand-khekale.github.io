---
layout: single
classes: wide 
title: Consume a REST Web service on IBM i (AS400) using SQL
header:
  og_image: /assets/images/forposts/Weather_screen.png
categories: 
 - ibmi 
 - rpgle
 - SQL 
tags: 
 - REST
 - JSON
 - Free format
 - as400
permalink: /:year/:month/:title:output_ext
---
![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/Weather_screen.png){: .align-center}    
In this post, we will see how we can build a weather app natively on IBM i using SQL, RPGLE and HTTPGETCLOB, JSON_TABLE functions. The app will consume RESTful APIs to get the data in JSON format. Db2 for i has both the SQL functions available from V7.2 onwards; this functionality will work if you are on or above that version of the OS. 
{: style="text-align: justify;"}

### First, some background about JSON.
JSON is JavaScript Object Notation, a format in which data is exchanged in "Key":"Value" pair enclosed in curly braces {}.
Below is the simple example of student information in JSON. 
{: style="text-align: justify;"}
{% highlight JSON %}
{
  "Student1": {
    "Mathematics": 80,
    "Physics": 95,
    "Computers": 85
  }
}
{% endhighlight %}  

As can be seen, if we need to get marks in Physics from the JSON, we have to navigate it as `Student1.Physics` to access it's value and we get 95. 

There are various ways JSON can be presented, sometimes it contains array of information nested for a key element. 
{% highlight JSON %}
{
    "Name" : "John Doe",
    "Phones" : [
      {"Office": 25252525},
      {"Home": 35353535}
    ]
}
{% endhighlight %}    
Here, there are 2 entries for Key `"Phones"`, hence if we want only Office phone number, we have to navigate with array notation as `Phones[0].Office`. The array index starts at zero. We will be using similar JSON in our application, hence this background.
{: style="text-align: justify;"} 

### Second, Web Services we will be using.
To get started, we will start by using two web services for our application. 
1. **Mapbox Web service** [Link](https://account.mapbox.com/auth/signup/):<br/>
The Mapbox Web service accepts City/Area as input and provides Latitude, Longitude and details about the area in a JSON format. [See the JSON Response](https://raw.githubusercontent.com/anand-khekale/IBMi/master/WeatherApp/mapbox.json)
2. **Weatherstack Web service** [Link](https://weatherstack.com/signup/free):<br/>
The Weatherstack API needs Latitude, Longitude for it to give weather forecast. This also gives JSON output to consume. [See the JSON Response](https://github.com/anand-khekale/IBMi/blob/master/WeatherApp/weatherstack.json) 
  
These APIs are free to sign-up. Once signed-up, we need to register/create an app on both APIs and get API Key or Token, also the URL. This API Key will be needed for our request to be identified and processed. We will be using the API keys provided by respective APIs while processing our requests.

### Third, the building blocks of our application. 
**DB2 for i** has various HTTP functions available in library `SYSTOOLS`. We will be using mainly below 3 functions.   
- **HTTPGETCLOB**  
This function accepts the URL of the web service and sends a `HTTP GET` request. If the request is successful, it gives back the default response provided by the web service provider. It may be in XML or JSON or any other HTTP supported format.  
HTTPGETCLOB is used as below:  
``HTTPGETCLOB(<Web URL>, <HTTP Header>)``

- **URLENCODE**  
This function is used to construct the URL needed by HTTPGETCLOB. The encoding of the URL needs to be correct for the web service to process it. Some characters are escaped while sending the request (e.g. `space` is replaced by either `+` sign or `%20`.  e.g. Los Angeles becomes Los%20Angeles). URLENCODE comes handy for this and is highly recommended.  
URLENCODE is used as below:  
``URLENCODE(<Value to be encoded>, <Encoding format, e.g.UTF-8 which is default>)``  

- **JSON_TABLE**  
JSON_TABLE parses the JSON that is returned by the HTTPGETCLOB function. Using this function, we can extract the fields we are interested in from the response.  
JSON_TABLE is used as below:  
``Select * from JSON_TABLE(<JSON-context-item>, '$' COLUMNS (Column-Name Data-type PATH column-path-expression)) AS X``

### Fourth, building our SQL procedures.
We will be creating two SQL procedures for our two APIs. 
- *getGeoCode*
- *getForecast*  

SQL procedures can be created using `RUNSQLSTM` or using *Run SQL Script* option of IBM's Access Client Solution.  
We will directly jump into the code, so that, it is clear.  

**Watch out!** `RUNSQLSTM` has given me issues with EBCDIC conversion of `$` and `[ ]`. At the moment, I don't seem to have solution. I have tried changing the job CCSID to 37, still, it didn't work. Using ACS's Run SQL Script is encouraged to create the procedures. 
{: .notice--warning}

The first SQL procedure is to fetch the location coordinates for the city passed, the code goes as below,

{% highlight SQL linenos %}
CREATE OR REPLACE PROCEDURE YourLibrary.getGeoCode(
                IN  City      CHAR(50),
                IN  APIKEY    VARCHAR(200),
                OUT Latitude FLOAT,
                OUT Longitude FLOAT,
                OUT Place     VARCHAR(100)
            )
        LANGUAGE SQL
        RESULT SETS 0
BEGIN

SELECT Lat, Lon, PName
    INTO Latitude, Longitude, Place
    FROM JSON_TABLE(
SYSTOOLS.HTTPGETCLOB(
  'https://api.mapbox.com/geocoding/v5/mapbox.places/' CONCAT 
          SYSTOOLS.URLENCODE(TRIM(City), '') CONCAT 
          '.json?access_token=' CONCAT
          SYSTOOLS.URLENCODE(TRIM(APIKEY),'') CONCAT
          '&limit=' CONCAT
          SYSTOOLS.URLENCODE('1',''), NULL
  ),
  '$'
  COLUMNS
  (Lat    FLOAT     path '$.features[0].center[1]',
   Lon    FLOAT     path '$.features[0].center[0]',
   PName  CHAR(100) path '$.features[0].place_name'
  ) error on error
  ) as x;
  END
{% endhighlight %}  

Here, we will see how the code is functioning,  

| Line   | Explanation |
|--------|:-------|
|1-9| We are using standard SQL create procedure with all the input and output parameters we need.|
|12-14| To use JSON_TABLE, we have to use select statement with the fields we are interested in. We are selecting Latitude, Longitude and Place details.|
|15-22| HTTPGETCLOB expects web-url and optional HTTP header as it's parameters, such as `SYSTOOLS.HTTPGETCLOB(URL,null)`, here we are not sending any header related information in our request, hence we have set it to NULL. The URL is constructed using URLENCODE as described above.|
|23|We are telling JSON_TABLE from where to extract information from the JSON. `$` means we want it from the root of it i.e. from first curly brace.|
|24-27| We select the columns we are interested in, declare the data type and provide the path to the field. This is similar to what we saw above about how to navigate a JSON while extracting office phone number.|  
|28|We use `Error on error` so that, if there are any issues processing the request, those are reported.|

Our second procedure is similar with the required input/output fields and the web service URL.  
{% highlight SQL linenos %}
CREATE OR REPLACE PROCEDURE YourLibrary.getForecast(
                IN  Latitude FLOAT,
                IN  Longitude FLOAT,
                IN  APIKEY    VARCHAR(200),
                OUT W_Description VARCHAR(500),
                OUT CurrentTemp FLOAT
            )
        LANGUAGE sql
        RESULT SETS 0
BEGIN

SELECT W_Desc, Temp
    INTO W_Description, CurrentTemp
    FROM JSON_TABLE(
SYSTOOLS.HTTPGETCLOB(
  'http://api.weatherstack.com/current?access_key=' CONCAT 
          SYSTOOLS.URLENCODE(TRIM(APIKEY), '') CONCAT 
          '&query=' CONCAT
          SYSTOOLS.URLENCODE(Latitude,'') CONCAT
          ',' CONCAT
          SYSTOOLS.URLENCODE(Longitude,''), NULL
  ),
  '$'
  COLUMNS
  (W_Desc  VARCHAR(500) path '$.current.weather_descriptions[0]',
   Temp    FLOAT        path '$.current.temperature'
  ) error on error
  ) as x;
  END
{% endhighlight %}

### Fifth, writing SQLRPGLE to use SQL procedures.
We will be writing simple DSPF and SQLRPGLE to demonstrate the functionality. I will not post the whole RPGLE program, but will just give a snapshot of main subroutine to get the idea. The source code is hosted at GitHub and links are given in the below section.  

![image-center]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/sqlrpg.png){: .align-center}  

Our application is ready to consume the web service! We can add error processing, capturing HTTP response status codes, etc. I kept the scope limited, so as to demonstrate how everything fits into the whole picture. 

### UPDATE (7th Sept.): Some observations and feedback received on this article. 

Thanks to the feedback received on this article, it is noted that, for some reason `SYSTOOLS.HTTPGETCLOB` gives `java.net.SocketException` for the first time the program/procedure is run. If this error is ignored and the program retried, it seems to work afterwards. Any suggestions/feedback is welcome on this issue. 
This is tested on IBM i V 7.4.
{: style="text-align: justify;"}  

#### Source codes
1. [Weather.sqlrpgle](https://github.com/anand-khekale/IBMi/blob/master/WeatherApp/weather.sqlrpgle)
2. [Weather DSPF](https://github.com/anand-khekale/IBMi/blob/master/WeatherApp/WEATHERDS.dspf)
3. [getGeoCode.sql](https://github.com/anand-khekale/IBMi/blob/master/WeatherApp/getGeoCode.sql)
4. [getForecast.sql](https://github.com/anand-khekale/IBMi/blob/master/WeatherApp/getForecast.sql)
5. [MapBox JSON Response](https://raw.githubusercontent.com/anand-khekale/IBMi/master/WeatherApp/mapbox.json)
6. [Weatherstack JSON Response](https://github.com/anand-khekale/IBMi/blob/master/WeatherApp/weatherstack.json)

#### Additional reading
- [JSON_TABLE](https://developer.ibm.com/articles/i-json-table-trs/)
- [HTTPGETCLOB](https://developer.ibm.com/tutorials/dm-1105httprestdb2/)
- [URLENCODE](https://www.ibm.com/support/knowledgecenter/en/SSEPEK_12.0.0/sqlref/src/tpc/db2z_udf_urlencode.html)

Feel free to reach me on [Twitter](https://twitter.com/anandkhekale). 