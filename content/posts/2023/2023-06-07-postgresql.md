---
title: "Park API - Using PostgreSQL instead of SQLite"
date: 2023-06-07T12:04:31+02:00
tags: ["iOS", "macOS", "swift", "server", "git", "hummingbird", "api", "postgresql", "docker"]
author: ["Szabolcs Tóth"]
draft: false
---

In the [previous part of the article](https://kicsipixel.github.io/posts/2023/2023-06-02-hummingbird/) we built our first API server based on Swift using [Hummingbird](https://github.com/hummingbird-project/hummingbird) framework. That time we stored our data in SQLite database, which is perfect for rapid prototyping but for production we need something more robust.

[Feather Database Component](https://github.com/FeatherCMS/feather-database) allows us to use PostgreSQL database as well. We need to make small modifications on our code.

### Step 1. - Run PostgreSQL database as container
I prefer to use containers if I know that the development and the production environment most probably are different. 

I have created this simple `docker-compose.yml` :

```yml
version: '3.7'
  
services:
  database:
    image: postgres:15-alpine
    restart: always
    env_file:
      - .env/development/database
    ports:
      - 5432:5432
    volumes:
      - db_data:/var/lib/postgresql/data
volumes:
  db_data:
``` 

 The `POSTGRES_USER`, `POSTGRES_PASSWORD`, and `POSTGRES_DB` environment variables are moved to a separate file under `.env/development` folder. 
 
 The `database` file:
 
 ```
POSTGRES_USER = hummingbird
POSTGRES_PASSWORD = hummingbird_password
POSTGRES_DB = hb-parks
 ```

Run start the container:

```bash 
docker-compose up
```

### Step 2. - Add PostgreSQL database dependency to your `Package.swift` file

We need to replace the SQLite component: `.product(name: "HummingbirdSQLiteDatabase", package: "hummingbird-db")` to PostgreSQL.

```
 // Database dependencies
                .product(name: "HummingbirdDatabase", package: "hummingbird-db"),
                .product(name: "HummingbirdPostgresDatabase", package: "hummingbird-db"),
```

### Step 3. - Configure PostgreSQL database in the `DatabaseSetup.swift` file

Creating and connecting to a PostgreSQL database needs more configuration than SQLite. So these changes need to be applied:

```swift
import Hummingbird
import HummingbirdPostgresDatabase

extension HBApplication {
    func setupDatabase() async throws {
        services.setUpPostgresDatabase(
            configuration: .init(
                host: "localhost",
                port: 5432,
                username: "hummingbird",
                password: "hummingbird_password",
                database: "hb-parks",
                tls: .disable
            ),
            eventLoopGroup: eventLoopGroup,
            logger: logger
        )
                
        try await db.execute(
            .init(unsafeSQL:
                                   """
                                   CREATE TABLE IF NOT EXISTS parks (
                                       "id" uuid PRIMARY KEY,
                                       "latitude" double precision NOT NULL,
                                       "longitude" double precision NOT NULL,
                                       "name" text NOT NULL
                                   );
                                   """
                 )
        )
    }
}
```

Obviously the `.env/development/database` file variables and `setUpPostgresDatabase` init values must be identical.

PostgreSQL database uses `double precision` instead of `double` as data type.

### Step 4. - That's all. You can run your server.
Once the PostgreSQL database is running in a container and you start you server as:

``` bash
swift run parkAPI
``` 

you are able to connect.

### Tools - you can use to make your development process easier

Working with databases is not always easy, specially if you don't have client to test. 

- For basic [CRUD](https://www.freecodecamp.org/news/crud-operations-explained/) operations I use [RaidAPI for Mac](https://paw.cloud/)
- For checking SQLite database - [Base 2](https://menial.co.uk/base/)
- For checking PostgreSQL - [TablePlus](https://tableplus.com/)
