---
title: MongoDB Atlas Search Autocomplete with Next.js
description: Learn how to build an autocomplete search feature using MongoDB Atlas Search and Next.js.
date: "2023-04-9"
url: https://www.mongodb.com/
repository: S-Selim/mongoDB-atlas-search-nextjs
published: true
---

## Introduction

This tutorial demonstrates how to build an autocomplete search feature using MongoDB Atlas Search and Next.js. MongoDB Atlas Search provides powerful full-text search capabilities, and Next.js is a popular React framework for building server-rendered applications.

# Overview of the project

In this tutorial, we will walk through the process of building a text search feature for a home rental website. We will leverage Atlas Search to incorporate full-text search capabilities, along with implementing an autocomplete function in the search box. This search functionality will enable users to find homes based on the country they are interested in.
Now, let's explore the technologies we'll be employing for this project.

# MongoDB Atlas Search

Is a comprehensive search and analytics engine incorporated within MongoDB Atlas. This powerful tool allows developers to effortlessly introduce search functionality to their applications, delivering swift and pertinent search results. MongoDB Atlas Search encompasses a range of search features, such as text and faceted search, autocomplete, and geospatial search. Engineered for exceptional scalability and user-friendliness, MongoDB Atlas Search is an invaluable resource for developers seeking to enhance their applications.

# Setting up a MongoDB Account

To start with, let's clone the repository

```bash
git clone https://github.com/S-Selim/mongoDB-atlas-search-nextjs

cd mongoDB-atlas-search-nextjs
```

To configure our MongoDB environment, follow these steps based on the official MongoDB documentation:

1. Register for a free MongoDB account

2. Create a new cluster

3. Add a database user

4. Set up a network connection

5. Import sample data

6. Obtain the connection string

# Determine a database for the project

For this application, we will use the `sample-airbnb` sample data from MongoDB, as it contains relevant entries suitable for our project. Make sure you have the sample data loaded in your cluster by following the steps mentioned above. If you need assistance, refer to the documentation on loading sample data.

Initialize the Node.js backend API
Our frontend will rely on the API provided by the Node.js backend. To set up a connection with your database, create a `.env` file and update it with the connection string.

```bash

cd backend

npm install

touch .env

```

Update .env

```bash

PORT=5050
MONGODB_URI=<CONNECTION_STRING>

```

To initiate the server, you can either use the Node.js executable or, for a more convenient development experience, employ Nodemon. This utility automatically restarts your server when it detects changes to the source code. To learn more about the installation process, refer to the official website.

Execute the following command to run the server:

```bash
npx nodemon .
```

Begin the Next.js frontend application
While the backend is running, let's initiate the frontend of your application. Open a new terminal window and navigate to the `mdbsearch` folder. Next, install all required dependencies for this project and start the project by executing the npm command. Additionally, create a `.env` file and update it with the backend URL.

```bash
cd ../mdbsearch

npm install

touch .env

```

Create .env file, and update as below:

```bash

NEXT_PUBLIC_BASE_URL=http://localhost:5050/

```

Start the application by running.

```bash
npm run dev
```

After the application starts running, you should see the following page

![](/page1.png)

The backend is already connected to the frontend; therefore, we'll need to make a few adjustments to our code throughout this implementation.

Implementing Text Search with MongoDB Atlas Search in Our Application

To enable searching through data in our collection, follow these steps:

# Create a search index

MongoDB's free tier account allows us to create up to three search indexes. From the previously created cluster, click on the "Browse Collections" button, navigate to the "Search" tab, and click on the "Create Index" button on the right side of the page. On the next screen, click "Next" to use the visual editor, add an index name (e.g., `search_home`), select the `listingsAndReviews` collection from the `sample_airbnb` database, and click "Next".

# Refine your index

Click on "Refine Your Index" to specify the fields in our collection that will be used to generate search results. In our case, we'll use the address and `property_type` fields. The address field is an object that has a country property, which is our target.

On this screen, disable the "Enable Dynamic Mapping" option. Under "Field Mapping", click the "Add Field Mapping" button. In the "Field Name" input, type address.country, and make sure the "String" data type is selected. Then, scroll to the bottom of the dialog and click the "Add" button. Create another field mapping for `property_type`, with the data type set to "String".

The index configuration should look like the following:

![](/page2.png)

# Testing the search index

MongoDB offers a search tester that you can use to test your search indexes before integrating them into your application. Click the "QUERY" button in the search index to access the Search Tester screen. Remember, we configured our search index to return results from address.country or property_type. You can test with values like Spain, Brazil, apartment, etc. These values will return results, and you can explore each result document to see where the result is found from these fields.

Try values like "span" and "brasil"; they will return no data result because they don't match exactly. MongoDB recognizes that scenarios like these are likely to happen, so Atlas Search offers a fuzzy matching feature. With fuzzy matching, the search tool searches not only for exact matching keywords but also for matches that might have slight variations. You can find more details on fuzzy search in the documentation.

With the search index created and tested, you can now implement it in your application. First, you need to understand what a MongoDB aggregation pipeline is.

# Integrating the search index into your backend application

Now that you have the search index configured, let's try integrating it into the API used for your project. Open the backend/index.js file, locate the comment "Search endpoint goes here," and update it with the following code. Begin by creating the route needed by your frontend.

```bash

// Search endpoint goes here
app.get("/search/:search", async (req, res) => {
const queries = JSON.parse(req.params.search)
// Aggregation pipeline goes here
});

```

In this endpoint, /search/:search, create a two-stage aggregation pipeline: $search and $project. $search uses the index search_home, which you created earlier. The $search stage structure is based on the query parameter sent from the frontend, while the $project stage returns only the required fields from the $search result.

This endpoint receives the country and property_type so you can start building the aggregation pipeline. There will always be a category property; you can start by adding this:

```bash

// Start building the search aggregation stage
let searcher_aggregate = {
  "$search": {
    "index": 'search_home',
    "compound": {
      "must": [
        // get home where queries.category is property_type
        { "text": {
          "query": queries.category,
          "path": 'property_type',
          "fuzzy": {}
        }},

        // get home where queries.country is address.country
        {"text": {
          "query": queries.country,
          "path": 'address.country',
          "fuzzy": {}
        }}
      ]}
  }
};

```

You might not want to send all the fields back to the frontend, so you can use a projection stage to limit the data you send over.

```bash
app.get("/search/:search", async (req, res) => {
  const queries = JSON.parse(req.params.search)
  // Start building the search aggregation stage
  let searcher_aggregate = { ... };
  // A projection will help us return only the required fields
  let projection = {
    '$project': {
      'accommodates': 1,
      'price': 1,


```

# Implement search feature in frontend application

Navigate to your project folder and open the `mdbsearch/components/Header/index.js` file. Locate the `searchNow` function and update it with the following code.

```bash
// Search function goes here
const searchNow = async (e) => {
  setshow(false);
  let search_params = JSON.stringify({
    country: country,
    category: `${activeCategory}`,
  });
  setLoading(true);
  await fetch(`${process.env.NEXT_PUBLIC_BASE_URL}search/${search_params}`)
    .then((response) => response.json())
    .then(async (res) => {
      updateCategory(activeCategory, res);
      router.query = { country, category: activeCategory };
      setcountryValue(country);
      router.push(router);
    })
    .catch((err) => console.log(err))
    .finally(() => setLoading(false));
};

```

NextFind the `handleChange`

With the above update, you can now explore your application. To start the application,

```bash
run npm run dev

```

Once the page is loaded, choose a property type, and then click on "search country." In the top search bar, type `"Brazil."` Finally, click the search button. You should see the results displayed as shown below.

![](/page3.png)

The search result displays data where address.country is "Brazil" and property_type is "apartment". You can experiment with various search terms like "braz", "brzl", "bral", etc., and you will still get results due to the fuzzy matching feature.

Now, the user experience on the website is good, but you can enhance it further by adding an autocomplete feature to the search functionality.

# Add autocomplete to the search box

Most modern search engines incorporate an autocomplete dropdown that offers suggestions as you type. Users prefer finding the correct match quickly rather than browsing through an endless list of possibilities. This section will show you how to use Atlas Search's autocomplete capabilities to implement this feature in our search box.

In our case, we expect to see country suggestions as we type into the country search input. To implement this, you need to create another search index.

From the previously created cluster, click on the "Browse collections" button and navigate to "Search".
On the right side of the search page, click on the "Create index" button.
On this screen, click "Next" to use the visual editor, add an index name (in our case, country_autocomplete), select the listingsAndReviews collection from the sample_airbnb database, and click "Next".
From this screen, click on "Refine Your Index". We need to toggle off the "Enable Dynamic Mapping" option.
Under "Field Mapping", click the "Add Field Mapping" button. In the "Field Name" input, type address.country, and for "Data Type", make sure "Autocomplete" is selected. Then scroll to the bottom of the dialog and click the "Add" button.
At this point, scroll to the bottom of the screen and click "Save Changes". Then, click the "Create Search Index" button. Wait for MongoDB to create your search index; it usually takes a few seconds to become active.
Once done, you should have two search indexes, as shown below.

![](/page4.png)

# Implement autocomplete API in our backend application

With this done, let's update our backend API as below:
Open the `backend/index.js` file, and update it with the below code:

The above endpoint will return a suggestion of countries as the user types in the search box. In a three-stage aggregation pipeline, the first stage in the pipeline uses the $search operator to perform an autocomplete search on the address.country field of the documents in the country_autocomplete index. The query parameter is set to the user input provided in the URL parameter, and the highlight parameter is used to return the matching text with highlighting.

The second stage in the pipeline limits the number of results returned to one.

The third stage in the pipeline uses the $project operator to include only the address.country field and the search highlights in the output.

# Implement autocomplete in our frontend application

Let's also update the front end as below. From our project folder, open the mdbsearch/components/Header/index.js file. Find the handleChange function, and let's update it with the below code.

```bash

const handleChange = async (e) => {
  setCountry(e.target.value);
  if (e.target.value.length >= 2) {
    setLoading(true);
    await fetch(
      `${process.env.NEXT_PUBLIC_BASE_URL}autocomplete/${e.target.value}`
    )
      .then((response) => response.json())
      .then((res) => {
        setShow(true);
        setSuggestions(res);
      })
      .catch((err) => console.log(err))
      .finally(() => setLoading(false));
  } else {
    setShow(false);
  }
};

```

With the above code, as the user types into the search box, the autocomplete suggestions will be fetched and displayed in a dropdown. The user can then choose a suggestion or continue typing to refine the search.

The above function will make an HTTP request to the `country/autocomplete` endpoint and save the response in a variable.

With our code updated accordingly, let's explore our application. Everything should be working well now. We should be able to search for homes by their country, and we should receive suggestions as we type into the search box. This enhancement provides a more user-friendly experience and enables users to find relevant results more efficiently.

![](/page5.png)

# Summary

To provide a great user experience on a website, you'll agree that it's crucial to make it easy for users to search for what they are looking for. In this guide, we demonstrated how to create a text search for a home rental website using MongoDB Atlas Search. This search allows users to find homes by their country.

MongoDB Atlas Search is a full-text search engine that enables developers to build rich search functionality into their applications, allowing users to search through large volumes of data quickly and easily. Atlas Search also supports a wide range of search options, including fuzzy matching, partial word matching, and wildcard searches. To learn more about MongoDB Atlas Search, check out the official documentation.
