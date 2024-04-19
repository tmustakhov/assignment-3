# REST API with Redis Caching in Go

## Overview
This project involves building a RESTful API in Go that uses Redis for caching data. The API interacts with a database, but before querying the database, it checks if the requested data is cached in Redis. If not, it retrieves the data from the database, returns it to the client, and stores it in Redis for subsequent requests.

## Objectives
- Set up Redis for caching.
- Connect to a relational database.
- Implement API endpoints that manage caching logic.
- Ensure caching reduces database queries for repeated data requests.

## Requirements
- **Go Programming Language**: All backend code should be in Go.
- **Redis**: Used as the caching layer.
- **Database**: Any relational database like MySQL or PostgreSQL.
- **Libraries/Frameworks**: Suggested frameworks include Gin for routing and GORM or database/sql for ORM operations.
- **Environment Setup**: Setup should be reproducible via local installations or Docker containers.

## Getting Started

### Prerequisites
- Go (version 1.15 or higher)
- Redis server
- A relational database (MySQL or PostgreSQL)
- Docker (optional, for container setup)

## Set up Redis
- **Install Redis locally or run a Redis container:**
  ```bash
  docker run --name redis -p 6379:6379 -d redis

## Set up the Database
- **Create a database and initialize it with the provided schema and data.**
- **Update the database connection settings in your application config.**


## API Documentation

### Endpoints

#### Retrieve a Product
- **GET /product/:id** - Retrieves a product by its ID from the cache or database.
  - **Success Response:**
    - **Code:** 200 OK
    - **Content:** `{ "id": 12, "name": "Product Name", "description": "Product Description", "price": 100 }`
  - **Error Response:**
    - **Code:** 404 Not Found
    - **Content:** `{ "error": "Product not found" }`

#### Create a Product
- **POST /product** - Creates a new product with the provided details.
  - **Request Body:** `{ "name": "New Product", "description": "Details of New Product", "price": 150 }`
  - **Success Response:**
    - **Code:** 201 Created
    - **Content:** `{ "id": 13, "name": "New Product", "description": "Details of New Product", "price": 150 }`
  - **Error Response:**
    - **Code:** 400 Bad Request
    - **Content:** `{ "error": "Invalid input" }`

#### Update a Product
- **PUT /product/:id** - Updates the specified product with new details.
  - **Request Body:** `{ "name": "Updated Product", "description": "Updated Description", "price": 200 }`
  - **Success Response:**
    - **Code:** 200 OK
    - **Content:** `{ "id": 12, "name": "Updated Product", "description": "Updated Description", "price": 200 }`
  - **Error Response:**
    - **Code:** 404 Not Found
    - **Content:** `{ "error": "Product not found" }`

#### Delete a Product
- **DELETE /product/:id** - Deletes a specific product.
  - **Success Response:**
    - **Code:** 204 No Content
    - **Content:** None
  - **Error Response:**
    - **Code:** 404 Not Found
    - **Content:** `{ "error": "Product not found" }`

#### List All Products
- **GET /products** - Retrieves a list of all products.
  - **Success Response:**
    - **Code:** 200 OK
    - **Content:** `[{ "id": 12, "name": "Product One", "description": "Description One", "price": 100 }, { "id": 13, "name": "Product Two", "description": "Description Two", "price": 150 }]`

## Caching Strategy

The caching strategy for this REST API uses Redis to minimize database access for frequently requested data. Here’s how it is implemented:

### Overview
- **Cache Storage**: Redis is used to store serialized versions of data objects. The key for each cache entry is typically the ID of the data (e.g., product ID for products).
- **Cache Lifetime**: Each cache entry has a Time To Live (TTL) which dictates how long the entry stays in the cache before it expires. The TTL can be configured based on the nature of the data and how often it changes.

### Cache Operations

#### Cache Reading
- When the API receives a request (e.g., GET /product/:id), it first checks if the product data for the given ID is available in the Redis cache.
- If the cache contains the data, it is returned immediately without querying the database.
- If the data is not in the cache, the next step is to retrieve it from the database.

#### Cache Writing
- If data is retrieved from the database (because it was not found in the cache or the cache data was expired), it is then serialized and stored in Redis with a set TTL.
- This newly cached data is also returned to the requester.

#### Cache Updating
- When a data object is updated (e.g., PUT /product/:id), the corresponding cache entry should be updated or invalidated.
- If the cache entry exists, it can either be updated with new data or removed to force a database fetch on the next request, ensuring data consistency.

#### Cache Deleting
- When a data object is deleted (e.g., DELETE /product/:id), the corresponding cache entry should be immediately removed to prevent stale data from being served.

### Benefits
- **Reduced Database Load**: By serving repeated requests from the cache, the load on the database is significantly reduced.
- **Improved Response Time**: Retrieving data from the cache is typically faster than querying a database, thus improving the response time for the end-users.
- **Scalability**: Caching allows the API to handle a larger number of requests by reducing the number of database queries, which might otherwise be a bottleneck.


