---
layout: single
classes: wide 
title: NodeJS - Consume REST API on IBM i (AS400)
header:
  og_image: /assets/images/forposts/currency_header.png
categories: 
 - ibmi 
 - NodeJS
 - Webservice 
tags: 
 - REST
 - JSON
 - JavaScript
 - as400
permalink: /:year/:month/:title:output_ext
---
![Currency-Query]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/currency_header.png){: .align-center}  
In my last [article](https://www.anandk.dev/2020/08/Webservice-IBMi-AS400.html), we saw how to consume a RESTFul API using native SQL functions. This article gives another method for it by using NodeJS on the IBM i.   
{: style="text-align: justify;"}
NodeJS has been available on IBM i from 2009, yes, it's that old! 
>(Correction! Thanks to [Kevin Adler](https://twitter.com/kadler_ibm) from IBM, "while NodeJS was created in 2009, it has only been available on IBM i since 2014." )  

I assume you know JavaScript basics, but still have provided enough details for beginners**.** 

Let's see the approach, the structure, and the tools needed to write such an application.

## The API we will be consuming

### Open Exchange Rates

Many business applications need currency rates regularly if the clients they are serving are from various countries.  I have chosen this API because it is easy to understand and is easy to update in DB2. 
{: style="text-align: justify;"}
First, we have to sign-up for the API by visiting this [URL](https://openexchangerates.org/signup/free).  The free plan allows 1000 requests per month for a default base ('USD') currency. The rates are refreshed every 30 minutes. This should be good enough for our application. Please signup and get your API-Key which will be needed in the application. 
{: style="text-align: justify;"}
This API returns currency rates in JSON format, like below (please refer my [earlier](https://www.anandk.dev/2020/08/Webservice-IBMi-AS400.html) article for a brief background on JSON). 
{: style="text-align: justify;"}
```json
{
  "disclaimer": "Usage subject to terms: https://openexchangerates.org/terms",
  "license": "https://openexchangerates.org/license",
  "timestamp": 1600678800,
  "base": "USD",
  "rates": {
    "AED": 3.672951,
    "AFN": 76.72753,
    "ALL": 104.631252,
    "AMD": 481.616228,
    "ANG": 1.792053,
    "AOA": 618.42,
    "ARS": 75.306,
    "AUD": 1.373749,
.................
}
```

## NodeJS libraries and ecosystem

NodeJS is a powerful engine based on Google's V8 engine for JavaScript, it is JavaScript outside of a web browser, for server-side programming. It is again enhanced by its vast array of open-source libraries, which are available on a registry [www.npmjs.com](http://www.npmjs.com). 
{: style="text-align: justify;"}
Sometimes, it becomes challenging to decide which library to use for a given function as there are many to choose from. I go with one, which has a large number of weekly downloads and its collaborator is updating/fixing it regularly, so those which have a strong support backbone. This can be checked from its [npmjs page](https://www.npmjs.com/package/needle) and [github page](https://github.com/tomas/needle).  
{: style="text-align: justify;"}
These libraries can be used by first installing them in our project directory and then by simply using the `require` keyword in JavaScript code, which is similar to how we use copybooks in RPGLE programs. 
{: style="text-align: justify;"}
```jsx
var needle = require('needle');
```

For this application, we will need 2 libraries, one for connecting to DB2 and the other for getting (GET) HTTP request to consume the API. We will see those two below.
{: style="text-align: justify;"}
### [idb-pconnector - Promise-based DB2 Connector for IBM i](https://www.npmjs.com/package/idb-pconnector)

I will be using idb-pconnector but,  IBM has recently released its [ODBC driver](https://github.com/IBM/ibmi-oss-examples/blob/master/odbc/odbc.md#installation-on-ibm-i) that can be used from Windows, macOS or Linux as well as from IBM i. Going forward, this ODBC library will be used extensively, as ODBC is a standardized API to interact with various DBMS. 
{: style="text-align: justify;"}
At the time of writing this article, I did not have any IBM i server that has this driver installed, hence I will be using idb-pconnector for our DB2 needs. NodeJS has its ODBC library [here](https://www.npmjs.com/package/odbc). 
{: style="text-align: justify;"}
idb-pconnector has various ways that make it easy to connect to the database and run SQL statements or call Stored Procedures. This is a promise-based version of the existing [idb-connector library](https://www.npmjs.com/package/idb-connector). (without `p` in connector)
{: style="text-align: justify;"}
The documentation for this library can be found [here](https://github.com/IBM/nodejs-idb-pconnector/blob/master/docs/README.md). 
{: style="text-align: justify;"}
We will visit briefly on what are promises, async, await functions of JavaScript so that this is easy to understand. 
{: style="text-align: justify;"}
### [Needle - HTTP Client on NodeJS](https://www.npmjs.com/package/needle)

Needle, as per its creator is, "The leanest and most handsome HTTP client in the Nodelands."   
{: style="text-align: justify;"}
We will be using its GET HTTP request from our API to get the currencies we need in the above-mentioned JSON format.  
{: style="text-align: justify;"}
## Some JavaScript Basics

### Callback/Promise/Async/Await

I highly recommend that you go through details of this topic [here](https://javascript.info/async). In this article we will just touch upon the concepts to give some idea as to how the code is behaving.
{: style="text-align: justify;"}
Let's say when we call multiple programs one by one in a CL program, those are being called **synchronously**, one after the other. The control transfers from one statement to another. On the other hand, when we use the SBMJOB command, the control comes back immediately, giving us the freedom to take the next step while the submitted job is doing its stuff, this is an **asynchronous** way in the JavaScript world!
{: style="text-align: justify;"}
But if some of our later code is dependent on our submitted job completion, then we have to write extra code to wait for its completion (like checking data queue, or data area), or write code to execute this functionality within the submitted program after completion of its intended function, in JavaScript it is called as callbacks.
{: style="text-align: justify;"}
Let's say, we call Program A with its parameter along with one more parameter as the name of program B, which is a callback function. Program A will complete its functionality and will call program B with the appropriate parameters.  This way the reusability of Program A increases as it dynamically calls a program for which it gets the name in a parameter. We also get the flexibility to write Program B each time based on our current needs. 
{: style="text-align: justify;"}
There are some challenges with callbacks called as [callback hell](https://javascript.info/callbacks#pyramid-of-doom), hence JavaScript has more advanced methods to do the same called as Promise based Async and Await, which we will be using in our application.
{: style="text-align: justify;"}
Promise is a special JavaScript object, which gives one of the two outputs, either a result or an error. It is called as Resolved or Rejected. 
{: style="text-align: justify;"}
We are using Async function that returns a promise, which is either resolved (we get the desired result) or rejected (we get an error), based on the outcome, we decide our next step.
{: style="text-align: justify;"}
Here we use the JavaScript keyword `await`, which waits for the promise to be resolved or rejected, and if it is rejected, we write `try` and `catch` block to monitor for any errors.
{: style="text-align: justify;"}
In our case with database update, we `await` for the execution of our SQL statement; it may go well or may end up in error, which is monitored in a `try` / `catch` block. The errors are displayed on the screen using `console.log` like `DSPLY` from where we are executing our application.
{: style="text-align: justify;"}
As I said, this is a vast topic and cannot be covered in just a few paragraphs. We will concentrate on demonstrating how to build NodeJS application on IBM i, so please revisit the articles on it for which a [link](https://javascript.info/async) is given.
{: style="text-align: justify;"}
## How to start writing NodeJS application?
We will see some basic steps needed to start writing a NodeJS application. These steps ensure that our application becomes portable and deployable on multiple systems by issuing few commands.   
NodeJS comes with its package manager called npm, short for Node Package Manager. This package manager is used to install the libraries as discussed in the previous section, also is a command-line utility to start creating our application.
{: style="text-align: justify;"}
**Note**: We will be writing this application in IBM i's PASE environment, so we need to get into it. There are two options, first, call `QP2TERM` program from the command line to get to the $ prompt of this environment, second, use an SSH session as described beautifully by Aaron Bartell  on [ITJungle](https://www.itjungle.com/2014/09/17/fhg091714-story01/).
{: style="text-align: justify;"}
I will be using the SSH session, as it is very convenient to issue commands or retrieve earlier issued commands and navigate through the file system. 
{: style="text-align: justify;"}
### NodeJS Version

I have written and tested this application on PUB400's IBM i, which is at Version 7.4 for IBM  i. NodeJS version can be checked using the below command. Some of the functionality in the application  may not work below this version. (QP2TERM SHELL)

```bash
$ node -v         
  v12.18.2
```  

Let's start with creating our application. Full application with source code is given, still, if you want to do it manually follow the below steps. 

### 1. Create a directory

We will create a directory in IFS (/home/YOUR_ID/) to store all the sources and downloaded Node libraries. I will call it `currency`.  Once created, I will change my current directory to `currency`.
{: style="text-align: justify;"}
```bash
# I am currently in my /home/USERID (home directory)
$ mkdir currency
$ cd currency
```

### 2. Initialize the application

When I am in my application directory, I will start initializing the application by using the Node package manager with command `npm init`. With this, npm creates two files for us; first, it will create package.json and when we start installing open-source libraries, it will create package-lock.json. These two files are kept updated with the list of installed libraries, their versions, and commands needed by the application.
{: style="text-align: justify;"}
```bash
$ npm init
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.

See `npm help init` for definitive documentation on these fields
and exactly what they do.

Use `npm install <pkg>` afterwards to install a package and
save it as a dependency in the package.json file.

Press ^C at any time to quit.
package name: (currency)
version: (1.0.0)
description: Fetch Currency API
entry point: (index.js) app.js
test command: node app.js 'USD'
git repository:
keywords:
author: www.anandk.dev
license: (ISC)
About to write to /home/anandk/currency/package.json:

{
  "name": "currency",
  "version": "1.0.0",
  "description": "Fetch Currency API",
  "main": "app.js",
  "scripts": {
    "test": "node app.js 'USD'"
  },
  "author": "www.anandk.dev",
  "license": "ISC"
}

Is this OK? (yes)

```

Once `npm init` command is issued, it starts asking you questions about the application.

I entered application-specific answers like name, description, entry point, test command, author, and kept the rest blank. It is fine if you enter nothing, but would be good to enter the name and description of the application. Finally, it asks if the generated JSON is OK, when confirmed the package.json file is created. 
{: style="text-align: justify;"}
These package.json and package-lock.json files are useful when you want to install our ready application once you download it from [here](https://github.com/anand-khekale/IBMi) on the IBM i. After downloading in the appropriate directory you just have to issue command `npm install` and all the required dependencies will be installed and the application will be ready to use. 
{: style="text-align: justify;"}
### 3. Installing the required libraries

As stated earlier, we will be using two libraries. We will now install those so that, those can be used in our programs. 
{: style="text-align: justify;"}
First, let's install our IBM i specific library.

```bash
$ npm install idb-pconnector
```

Once this library is installed, we will proceed with installing the HTTP client library Needle

```bash
$ npm install needle
```

After successful installation, I will just verify, if the package.json is updated with dependencies and a package-lock.json file is created. 
{: style="text-align: justify;"}
### 4. Start writing our programs

We have everything now to start writing our programs. The basic directory structure I have created is as below.  The full source is available on GitHub [here](https://github.com/anand-khekale/IBMi). 
{: style="text-align: justify;"}
```bash
/home/ANANDK/currency$ tree
.
├── app.js
├── db
│   ├── createDb.js
│   └── updateDb.js
├── node_modules
├── package-lock.json
├── package.json
└── utils
    ├── formatDate.js
    └── getCurrency.js
```

Please note, `node_module` is the directory created by npm when we start installing the individual libraries; we are not supposed to touch/change that directory. 
{: style="text-align: justify;"}
Let's go through our sources one by one. I have added enough comments so that additional explanation is not needed. 
{: style="text-align: justify;"}
### 5. Create table CURRENCY - createDb.js

Well, we can create the table using DDS, DDL, or using STRSQL, but I just created this source, which will be run for the first time, just to showcase, this can be done from NodeJS as well. 
{: style="text-align: justify;"}
```jsx
// import required class - DBPoll from idb-pconnector for connection to DB2
const { DBPool } = require('idb-pconnector');

// Keep the SQL statements ready
const sql = `CREATE OR REPLACE TABLE APKHEKALE1.CURRENCY(
                Date Timestamp(0),
                BCURR CHAR(3),
                USD Decimal(12, 4),
                EUR Decimal(12, 4),
                GBP Decimal(12, 4),
                INR Decimal(12, 4),
                SGD Decimal(12, 4)) RCDFMT RCURRENCY`;

const label = `LABEL ON COLUMN APKHEKALE1.CURRENCY (
                Date TEXT IS 'Date fetched',
                BCURR TEXT IS 'Base Currency',
                USD  TEXT IS 'United States Dollar',
                EUR  TEXT IS 'Euro',
                GBP  TEXT IS 'Pound sterling',
                INR  TEXT IS 'Indian Rupee',
                SGD  TEXT IS 'Singapore Dollars')`;

const colhdg = `LABEL ON COLUMN APKHEKALE1.CURRENCY (
                Date  IS 'Date fetched',
                BCURR IS 'Base Currency',
                USD   IS 'United States Dollar',
                EUR  IS 'Euro',
                GBP  IS 'Pound sterling',
                INR  IS 'Indian Rupee',
                SGD  IS 'Singapore Dollars')`;
// Write async - promise based function so that await can be used
async function createDB() {

    try {
        const pool = new DBPool(); //Create a new instance of pool
        const connection = pool.attach(); // create the connection
        const statement = connection.getStatement(); //create new instance of statement
        // execute and await each statement, if something goes wrong, we will be in error block. 
        await statement.prepare(sql);
        await statement.execute();
        await statement.prepare(label);
        await statement.execute();
        await statement.prepare(colhdg);
        await statement.execute();

        await pool.detach(connection); //Once done, detach/close the connection 
    } catch (error) {
        console.log(error);
    }
}

//call the function and catch errors, if any.
createDB().catch((error) => {
    console.error(error);
})
```

Once the source is ready, while we are in the `currency/db` directory, we will issue `node createDb.js` command to create the table. (QP2TERM SHELL)
{: style="text-align: justify;"}
```bash
$ cd currency/db
$ node createDb.js 
```

### 6. Our entry point of application - app.js

```jsx
// import our getCurrency function from utils directory, this is similar to /COPY in RPGLE
const getCurrency = require('./utils/getCurrency');
// import our updateDb function from db directory
const updateDb = require('./db/updateDb');

const baseCurr = process.argv[2]; // accept base currency from command line

//If base currency is not entered, give error and exit the program.
if (!baseCurr) { return console.log('Base Currency not entered') }

/* Call getCurrency function passing base currency and a callback function. 
   The getCurrency will give a call to callback with appropriate parameters,
   if it is successful it will have currency data, else it will have error, but not both. */
getCurrency(baseCurr, (error, currencyData) => {
    if (error) {
        console.log(error);
    } else {
        // call updateDb for updating the data into DB2 with currency rates. 
        updateDb(currencyData).catch((error) => {
            console.error(error);
        })
    }
})
```

### 7. Get (fetch) the Currency data - getCurrency.js

```jsx
// Import Needle library that we installed, to issue HTTP GET request.
var needle = require('needle');
// Assign API key that we received from https://openexchangerates.org/signup/free
const API_KEY = 'YOUR-KEY-FROM-OPEN-EXCHANGE-RATES';

const getCurrency = (baseCurr, callback) => {
    const url = `https://openexchangerates.org/api/latest.json?app_id=${API_KEY}&base=${baseCurr}`
    needle.get(url, function (error, response) {
        if (error) { //Check if there are any communication errors
            callback('Unable to connect to currency services', undefined)
        } else if (response.body.error) { // Check if we received any error from API
            callback({ "error": response.body.description }, undefined);
        } else if (!error && response.statusCode == 200) { // All went well and we have the needed data
            callback(undefined, response.body); // Call the callback with no error and data
        }
    })
}
/* Since we will be using getCurrency from outside this source, we need to export it, similarly as
   we do with RPGLE procedures                                                                     */
module.exports = getCurrency;
```

### 8. Update the database - updateDb.js

```jsx
//import only required class - DBPool from idb-pconnector 
const { DBPool } = require('idb-pconnector');
//get our utility which formats the date for us 
const formatDate = require('../utils/formatDate');

//Write async function as we will use promise based await while updating the database
async function updateDb(currencyData) {

    // Format the date in IBM i timestamp format as CCYY-MM-DD-HH.MM.SS
    const timestamp = formatDate(currencyData.timestamp)

    // Select required currency to be updated into database from the JSON
    const baseCurr = currencyData.base;
    const currUSD = currencyData.rates.USD;
    const currEUR = currencyData.rates.EUR;
    const currGBP = currencyData.rates.GBP;
    const currINR = currencyData.rates.INR;
    const currSGD = currencyData.rates.SGD;

    // Create a database connection and update the database. 
    try {
        // Connect DB2 using a pool of connection. This is useful for scalability.
        const pool = new DBPool();
        const sql = `Insert into APKHEKALE1.CURRENCY values (?,?,?,?,?,?,?) WITH NONE`;
        const params = [timestamp, baseCurr, currUSD, currEUR, currGBP, currINR, currSGD];
        await pool.prepareExecute(sql, params); // prepare and execute the statement 
    } catch (error) { // if something goes wrong, like SQL error, catch those here. 
        console.log(error);
    }
}
//Export the module, so that it can be used from outside this source. 
module.exports = updateDb;
```

### 9. Running the application

There are two ways you can call the node application, from your SSH or QP2TERM session, or; from the CL/RPGLE program. We will see it one by one. 
{: style="text-align: justify;"}
### a. Running from PASE environment

From your SSH/QP2TERM session, change your current directory to `currency`.  Make sure that the PF is created in your library, use the below command to run the application.
{: style="text-align: justify;"}
```bash
$ node app.js 'USD'

```

Command run from QP2TERM session (CALL QP2TERM)  
![QP2TERM-View]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/currency_qp2term.png){: .align-center}  

Here we are using node to run our first entry point, that is app.js, and passing 'USD' as the base currency. Once the application runs fine, we will be back to our $ prompt. 
{: style="text-align: justify;"}
We can also utilize the package.json file which got created when we started building our application, we provided our `test` script to be `node app.js 'USD'` . Enter the command as shown below (QP2TERM SHELL).  

```bash
$ cd currency
$ npm run test 
> currency@1.0.0 test /home/ANANDK/currency
> node app.js 'USD'
$
```

We can check if our CURRENCY table got updated to make sure everything is working as expected (I ran this multiple times on different days, hence these many entries).
{: style="text-align: justify;"}
![Currency-Query]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/currency_header.png){: .align-center}

We can test our validation for a base currency, without passing it as a parameter, see below. 
![Base-Curr-Validation]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/currency_validation.png){: .align-center}

### b. Running from CL programs

To run any program in the PASE environment, I feel inclined to writing a small shell script to take care of navigating to the correct directory. The script is as below named as runcurrency.sh.
{: style="text-align: justify;"}
```bash
#!/QOpenSys/usr/bin/sh
# This line with '#' is a comment, the above line with '#!' is a Shebang, by which the shell knows which shell to use.
# We have used 'sh' shell on the IBM i which is mostly available by default, alternatively one may use 
# /QOpenSys/pkgs/bin/bash (to check if it's available, do a cd /QOpenSys/pkgs/bin/ if there are no errors, bash is 
# available on your IBM i)
# For more information visit - https://www.ibm.com/support/knowledgecenter/ssw_ibm_i_74/rzalf/rzalfpase.htm

# The below line runs our application by navigating to the app.js in the directory structure. 
node /home/ANANDK/currency/app.js 'USD'
```

Whenever a shell script is written in a file, the file needs to be converted into an executable, this is done by issuing the below command (from QP2TERM shell prompt).
{: style="text-align: justify;"}
```bash
$ cd currency
$ chmod +x runcurrency.sh 
```

Then it becomes easy to call this shell script from a CL program.

```bash
PGM                                             
QSH        CMD('/home/ANANDK/currency/runcurrency.sh')
ENDPGM
```

### 10. Installing the application

You can directly install this application by downloading the folder from GitHub [here](https://github.com/anand-khekale/IBMi). Once the folder is downloaded follow the below steps.
{: style="text-align: justify;"}
1. From a text editor change the below sources.
    - Change the API Key in getCurrency.js - Replace 'YOUR-KEY-FROM-OPEN-EXCHANGE-RATES' with the key you obtained from the API provider.
    - Change library in createDb.js
    - Change library in updateDb.js
    - Change path with your ID, in runcurrency.sh ==> /home/**YourID**/currency/app.js
    - Change path with your ID in RUNNODE.CLLE ==> /home/**YourID**/currency/runcurrency.sh
2. After updating the sources, upload the folder `currency` into your IFS  `/home/YourID/` directory.  IBM's Access Client Solution has  `Actions→Integrated File System→Actions→Upload` to upload the files.  

    I use [VSCODE](https://code.visualstudio.com/) with [SSHFS](https://marketplace.visualstudio.com/items?itemName=Kelvin.vscode-sshfs) extension, and write code directly into IFS on the IBM i from my computer. I will write an article explaining how to do this.

3. Install node libraries
    - Call QP2TERM, on $ prompt execute the below one after the other (package.json and package-lock.json files are necessary for this step). This  step will take some time and will provide details of libraries installed.
    {: style="text-align: justify;"}
    ```bash
    $ cd currency
    $ npm install

    ```

4. Make the `runcurrency.sh` as executable from QP2TERM shell, while you are in `currency` directory.

    ```bash
    $ chmod +x runncurency.sh 
    ```

5. Create our database table (PF) running createDb.js  (QP2TERM shell). 

    ```bash
    $ cd currency/db
    $ node createDb.js
    $
    ```

6. Create RUNNODE CLLE program (From CL CMD), copy and compile the CL source.

    ```
    CPYFRMSTMF FROMSTMF('/home/**YOURID**/currency/RUNNODE.CLLE') TOMBR('/QSYS.lib/**YOURLIB**.lib/QCLSRC.file/RUNNODE.mbr') MBROPT(*REPLACE)
    ```

    Run the application as per instructions given in earlier section. 

## Conclusion

I hope this article gave you some ideas on how to create and use NodeJS to consume a RESTful API. The development time is considerably lower, as NodeJS has various libraries at our disposal to quickly write any application.
{: style="text-align: justify;"}
Special thanks to [PUB400](http://www.pub400.com) for hosting free IBM i with latest stuff on it. 
{: style="text-align: justify;"}
As always, request you to get in touch with me on [LinkedIn](https://www.linkedin.com/in/anandkdev)/[Twitter](https://twitter.com/anandkdev) or [email](mailto:anand.khekale@gmail.com) with your valuable feedback. Thanks for reading!
{: style="text-align: justify;"}