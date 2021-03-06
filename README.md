# guardian_scraper
This is a python project that scrape articles from [guardian](https://www.theguardian.com/au), and then expose the data through an API

## Installation and Configuration


### python dependencies

First, clone the project and go to the project directory. 

```
git clone https://github.com/Ahsanul08/guardian_scraper.git
cd guardian_scraper
```

Before running this project, user needs to make sure all the dependencies are installed. It can be installed with `pip` like below:

```
pip install -r requirements.txt
```

### Database configuration

The project is designed in a way, that it chooses the database (local or remote) based on an environment variable (`ENV`).

If the `ENV` is set to `DEV`, it will be considered as a local deployment, so the local database will be used. However, if `ENV` is not set, production URL would be used. 

You can set `ENV` from terminal, by:

```
export ENV=DEV
```

To add permanently, add it to the `.bashrc` file.

N.B. For security reason, the production database URI is not given in code. You can go to `guardian_crawler/guardian/settings.py` (For crawler) and `guardian_scraper/guardian_api/conf/settings.py` (for API) to put the right database URI.

## Usage

Inside the directory, two directory named `guardian_crawler` and `guardian_api` holds the crawler and API code respectively. Let's see one by one,how to deploy them.

### Crawler

Enter into the project with

```
cd guardian_crawler
```

And, run the following command. 


```
scrapy crawl guardian
```

It will crawl the last 2 days (default) of data of all category from guardian. However, the no of day can be controlled from  outside though a parameter. 


For example:

```
scrapy crawl guardian -a num_of_days=30
```

The above command will crawl last 30 days of data. 


### Flask API

To run the API, first go inside the project directory.

```
cd guardian_api
```

And, run the application with 

```
python run.py
```

As described above, based on `ENV` set or not, the database, port and running mode will be adjusted. 

So, if you run in production, you might need to add `sudo` with the command, as the production app run on port `80`.

At local deployment however, the app will be visible at http://localhost:5000


## Testing

To run the tests of `API`, you need to go to the directory `guardian_api`, and run:

```
python3 -m pytest tests
```

For the scraper, the common testing methodology is to serve an offline page and check the parser against it. However, scrapy provides a testing utility called `scrapy contracts`, which is used in our crawler project. 

To run this, you need to go to the directory `guardian_crawler`, and run:

```
scrapy check
```

## API Documentation

In this API, no addition/modification/deletion is not allowed (`POST`, `PUT`, `UPDATE`, `DELETE`). Only data retrieval (`GET`) is possible through few endpoints. 

### URL (Retrieval of collection of Articles)

  `/articles`

- #### Methods

  `GET`

- #### URL Params

  #### optional
  
  `keyword=[alphanumeric]`
    
   Provides a keyword based search on the whole list of articles. If not provided, the article list is served (paginated) 

  `date=[alphanumeric]`
    
   Provides articles of a specific date. If not provided (or invalid value), the filter isn't considered.  

  `category=[alphanumeric]`
    
   Provides articles of a specific category. If not provided (or invalid value), the filter isn't considered.  

  `sub-category=[alphanumeric]`
    
   Provides articles of a specific sub-category. If not provided (or invalid value), the filter isn't considered.  

  `author=[alphanumeric]`
    
   Provides articles of a specific author. If not provided (or invalid value), the filter isn't considered.  


   `offset=[int]`
   
   Works as pagination offset. 0 is considered as value when not provided. 
   
   `limit[int]`
   
   Works as pagination page size. 10 is default when not provided.
   
- #### Success Response:
    
    * #### Code: 200
      #### Content : 
      
      ```
      {
        "articles": [
          {
            "url": "https://www.theguardian.com/world/2018/oct/07/haiti-hit-by-59-magnitude-earthquake-causing-injuries-and-building-damage", 
            "content": "An earthquake that struck off .....", 
            "sub_category": "World news", 
            "category": "News", 
            "creation_date": "2018-10-07", 
            "id": "0a202ac6ea81550e0f0f6"
          }
        ], 
        "previous_url": "http://host/path?keyword=haiti&offset=0&limit=5",
        "next_url": "http://host/path?keyword=haiti&offset=10&limit=5"
      }
      ```` 
      Pagination URLs are served as `previous_url` and `next_url`.
      
      
- #### Error Response:
    
    * #### Code: 404
      #### Content : 
      
      ```
      {}      
      ``` 
      
- #### Sample Requests:
          
     ```
        # To get articles having trump keyword
        curl -XGET http://ec2-34-245-40-21.eu-west-1.compute.amazonaws.com/articles?keyword=trump
        # To get all the articles (with pagination) in DB
        curl -XGET http://ec2-34-245-40-21.eu-west-1.compute.amazonaws.com/articles
        # To get a specific page of articles in DB
        curl -XGET http://ec2-34-245-40-21.eu-west-1.compute.amazonaws.com/articles?offset=10&limit=10
        # To get a specific page of article about trump in DB
        curl -XGET http://ec2-34-245-40-21.eu-west-1.compute.amazonaws.com/articles?keyword=trump&offset=10&limit=10              ``` 

### URL (Retrieve a specific article by Article ID)

  `/article/<article_id>`

- #### Methods

  `GET`

- #### URL Params

      NO
      
- #### Success Response:
    
    * #### Code: 200
      #### Content : 
      
      ```
       {
         "url": "https://www.theguardian.com/money/2018/oct/05/thames-water-meter-refuse", 
         "content": "Thames Water is insisting we have a meter. Can we refuse? ...", 
         "headline": "Can I say no to a water meter?", 
         "author": "Anna Tims", 
         "sub_category": "Money", 
         "category": "Lifestyle", 
         "creation_date": "2018-10-05", 
         "id": "5bf55014ac6ea81550e103a6"
       }
      ```       
      
- #### Error Response:
    
    * #### Code: 404
      #### Content : 
      
      ```
       {}      
      ``` 
      
- #### Sample Request:
          
     ```
        # To get a specific article
        curl -XGET http://ec2-34-245-40-21.eu-west-1.compute.amazonaws.com/article/5bf55014ac6ea81550e103a6
     ``` 

## Limitations

-  Though the API provides certail fields to use as filter, only `keyword` field is indexed (As that was the priority), so other filters should be a bit slower. Idially, using elasticsearch would perform better, as keys are indexed there.
-  The search by `keyword`, by no means a full text search, rather as the name suggests a single keyword search.
-  Pagination could be implemented better rather than just immediate next and previous page.
-  No HTTP verbs other than `GET` is allowed, no state change of the server.
-  Search Fields like `author`, `category` could be normalized to have a better search, rather than an exact match. But left as lower priority.   
  

