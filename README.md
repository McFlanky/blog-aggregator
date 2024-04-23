# Web Server + RSS Scraper
### Why?
Because I wanted to create a real world application that encompasses my skills and learning of the programming language Go into 1 project, to show my expertise in the foundations of creating services!

## 🤖 Introduction

My goal in this project is to show that I can not only create cohesive and comprehensive services, not only in using Go, but using production ready tools and best practices to achieve this goal. This goes to show no matter where you come from like me in front end with JS and basic HTML/CSS, that you can learn and transition to backend development. This RSS Aggregator is built in such a way that it showcases all the fundamentals in web servers and RESTful apis among other system design choices...

## ⚙️ Tech Stack

- Go (Golang)
- PostgreSQL
- SQLc
- Goose
- Thunder Client

## System Design

 Here is a breakdown of the choice of features and design and reasoning behind each:
 ### What we want
 - Users to be able to scrape RSS feeds
 - Users to be able to store those scraped RSS feeds in a database
 - Users can specify which feeds they want to follow/unfollow
 - Users can fetch all latest posts from the RSS feeds they follow

### Detailed Design/Breakdown
1) Create a basic CRUD server with Chi routing/CORS middleware function and add http.Server/port and add multiplexer to it.
2) Create helper functions respondWithJSON/respondWithError with  handler for GET /v1/readiness requests, and handler for GET /v1/err requests.
3) Add PostgreSQL DB using psql to interact with database and sqlc to generate Go code from SQL queries, and goose for running migrations from the same SQL files that sqlc uses.
4) Using these tools we can create queries in an sql/schema directory to create goose migrations like goose down to rollback a migration and goose up to move database from its old state to a new state.
5) Then we use a connection string using psql to connect to our database called 'blogator', and we can run any migration with +goose Up like create table etc. Then we configure sqlc in a yaml file to tell sqlc where to look for our schema directory and to generate go code in our internal/database directory.
6) Then a query is written to generate code and and it creates a new package of go code in internal/database directory.
7) From this we can create a config struct to store shared data that HTTP handlers need access to, then we load in our DB url and open a connection to our DB, and use our generated DB package to create a new *databae.Queries and store it in a config struct!
8) Now we can another endpoint for our RESTful API called POST /V1/users to create a user where we want to get back in JSON as a response an id, created_at, updated-at, and name.
9) Now we want to create an API KEY column to our users table, then create an API KEY for new users, add a new SQL query to get a user by their api key, generate new go code, and then return the current user in another new endpoint called GET /v1/users using "encode(sha256(random()::text::bytea), 'hex')" to generate completely random 256 bit hex values for very good security!
10) Heres the fun part, we create a feeds table with name, url, user_id and ON DELETE CASCADE on the user-id foreign key so that if a user is deleted, all of their feeds are automatically deleted as well! The we add query to create a feed, create auth middleware, then create a handler to create a feed, and test it of course!
11) Create a new endpoint to get ALL FEEDS in the database with no auth, and update that same endpoint using a feed follow many to many relationship so that users can follow many feeds and a feed can be followed by many users!
12) So create a feed follow endoint called POST /v1/feed_follows, with fields id, feed_id, user_id, created_at, updated-at, a DELETE a feed follow endpoint: DELETE /v1/feed_follows/{feedFollowID}, Get all feed follows for a user: GET /v1/feed_follows with auth, and lastly automatically create a feed follow when creating a feed.
13) NOW SCRAPER TIME! Add a last_fetched_at column to the feeds table, and a getNextFeedsToFetch/markFeedFetched query to DB. and a function that can fetch data from a feed with a worker that fetches feeds continuously using go concurrency.
14) LASTLY BUT NOT LEASTLY, add a posts column to the DB and a create post/get posts sql query to DB and updated scraper to save posts. On top of adding a get posts by user HTTP endpoint; GET /v1/posts with auth.
