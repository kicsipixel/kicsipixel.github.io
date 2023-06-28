---
title: "Park API - Server side Swift with Hummingbird"
date: 2023-06-02T12:04:31+02:00
tags: ["iOS", "macOS", "swift", "server", "git", "hummingbird", "api"]
author: ["Szabolcs Tóth"]
draft: false
---

*Special thanks to [Tibor Bödecs](https://twitter.com/tiborbodecs) for his patience and guidence during the writing of this tutorial.*

Server side Swift has been available since end of 2015. The idea was behind the development that you can use the same language for RESTful APIs, desktop and mobile applications. With the evolution of the Swift language, the different Swift web frameworks got more robust and complex. 

That's why I was happy to read [Tib's excellent article](https://theswiftdev.com/beginners-guide-to-server-side-swift-using-the-hummingbird-framework) about a new HTTP server library written in Swift, [Hummingbird](https://github.com/hummingbird-project/hummingbird). I immediately liked the concept of modularity, so decided to create a tutorial to show its simplicity.

We will build a swift server running on SQLite database, which will store playgrounds around the city with name and coordinates. A simple JSON response will look like this:

```json
[
	{
		"latitude": 50.105848999999999,
		"longitude": 14.413999,
		"name": "Stromovka"
	},
	{
		"latitude": 50.0959721,
		"longitude": 14.4202892,
		"name": "Letenské sady"
	}, {
		"latitude": 50.132259369,
		"longitude": 14.46098423,
		"name": "Žernosecká - Čumpelíkova"
	}
]
```


The project will use FeatherCMS's own [Database Component](https://github.com/FeatherCMS/hummingbird-db).

## Step 1. - Init the project

```bash
mkdir parkAPI && cd $_
swift package init --type executable
```

This creates the backbones of our project. One of the most important file and initial point of our project is the  `Package.swift`, the Swift manifest file. [Here](https://theswiftdev.com/the-swift-package-manifest-file/) you can read more about it.

## Step 2. - Create the folder structure
We need to follow a certain guidelines about folder structure, otherwise the compiler won't be able to handle our project. On the picture below, you can find the simplest structure, which follow the [Hummingbird template](https://github.com/hummingbird-project/template). 


```
.
├── Package.swift
├── README.md
└── Sources
    └── parkAPI
        ├── App.swift
        └── Application+configure.swift
```


We will add the `Tests` folder later, when we will have something to test.

## Step 3. - Run the server

Before we are able to run our server, we need to add two packages to the `Package.swift` file:

- Hummingbird
- Swift Argument Parser

Using `.executableTarget` the `@main` will be the enrty point of our application and we can rename `main.swift` to `App.swift`. Paul Hudson wrote a [short article](https://www.hackingwithswift.com/swift/5.4/spm-executable-targets) about it. 

```swift
import PackageDescription

let package = Package(
    name: "parkAPI",
    platforms: [
        .macOS(.v12),
    ],
    dependencies: [
        .package(url: "https://github.com/hummingbird-project/hummingbird", from: "1.5.0"),
        .package(url: "https://github.com/apple/swift-argument-parser",from: "1.2.0"),
    ],
    targets: [
        .executableTarget(
            name: "parkAPI",
            dependencies: [
                .product(name: "ArgumentParser", package: "swift-argument-parser"),
                .product(name: "Hummingbird", package: "hummingbird"),
                .product(name: "HummingbirdFoundation", package: "hummingbird"),
            ],
            swiftSettings: [
                .unsafeFlags(
                    ["-cross-module-optimization"],
                    .when(configuration: .release)
                )
            ]
        )
    ]
)
```

Define the hostname and port in the `App.swift`.

```swift
import ArgumentParser
import Hummingbird

@main
struct App: ParsableCommand {

    @Option(name: .shortAndLong)
    var hostname: String = "127.0.0.1"

    @Option(name: .shortAndLong)
    var port: Int = 8080

    func run() throws {
        let app = HBApplication(
            configuration: .init(
                address: .hostname(hostname, port: port),
                serverName: "parkAPI"
            )
        )
        
        try app.configure()
        try app.start()
        app.wait()
    }
}
```

One last thing remained before we can run our application is to define the `route` in the `Application+configuration.swift`.

```swift
import Hummingbird
import HummingbirdFoundation

public extension HBApplication {

    func configure() throws {
         router.get("/") { _ in
            "The server is running...🚀"
        }
    }
}
```

Run out first Hummingbird server by typing:
```
swift run parkAPI
```


## Step 4. Create API response
Our server will be accessible on the following routes, using different [HTTP methods](https://www.freecodecamp.org/news/http-request-methods-explained/).

- `GET` - `http://hostname/api/v1/parks`: Lists all the parks in the database
- `GET` - `http://hostname/api/v1/parks/:id`: Shows a single park with given id
- `POST` - `http://hostname/api/v1/parks`: Creates a new park
- `PATCH` - `http://hostname/api/v1/parks/:id`: Updates the park with the given id
- `DELETE` - `http://hostname/api/v1/parks/:id`: Removes the park with id from database

### Step 4.1 Add database dependency

Our server will use SQLite database to store all data, so we need to add the [database dependency](https://github.com/FeatherCMS/hummingbird-db) to our manifest file. This will allow the server to communicate to the database.

The updated `Package.swift` file will look like this:

```swift
import PackageDescription

let package = Package(
    name: "parkAPI",
    platforms: [
        .macOS(.v12)
    ],
    dependencies: [
        .package(url: "https://github.com/hummingbird-project/hummingbird", from: "1.5.0"),
        .package(url: "https://github.com/apple/swift-argument-parser",from: "1.2.0"),
        // Database dependency
        .package(url: "https://github.com/feathercms/hummingbird-db", branch: "main")
    ],
    targets: [
        .executableTarget(
            name: "parkAPI",
            dependencies: [
                .product(name: "ArgumentParser", package: "swift-argument-parser"),
                .product(name: "Hummingbird", package: "hummingbird"),
                .product(name: "HummingbirdFoundation", package: "hummingbird"),
                // Database dependencies
                .product(name: "HummingbirdDatabase", package: "hummingbird-db"),
                .product(name: "HummingbirdSQLiteDatabase", package: "hummingbird-db"),
            ],
            swiftSettings: [
                .unsafeFlags(
                    ["-cross-module-optimization"],
                    .when(configuration: .release)
                )
            ]
        )
    ]
)
```

### Step 4.2 Use concurrency in `App.swift`

Add `async` to `function run()` and `await` to `app.configure`.

```swift
import ArgumentParser
import Hummingbird

@main
struct App: AsyncParsableCommand, AppArguments {
    
    @Option(name: .shortAndLong)
    var hostname: String = "127.0.0.1"
    
    @Option(name: .shortAndLong)
    var port: Int = 8080
    
    func run() async throws {
        let app = HBApplication(
            configuration: .init(
                address: .hostname(hostname, port: port),
                serverName: "Hummingbird"
            )
        )
        
        try await app.configure()
        try app.start()
        app.wait()
    }
}
```

  We need to use [`AsyncParsableCommand`](https://apple.github.io/swift-argument-parser/documentation/argumentparser/asyncparsablecommand/) and `AppArguments` protocols.
  
  
### Step 4.3 Create a database configuration file 

Add a new file, with name `DatabaseSetup.swift` file under a new `Database` folder under `Source/parkAPI/`.
  
```swift
import Hummingbird
import HummingbirdSQLiteDatabase

extension HBApplication {
    func setupDatabase() async throws {
        services.setUpSQLiteDatabase(
            storage: .file(path: "./hb-parks.sqlite"),
            threadPool: threadPool,
            eventLoopGroup: eventLoopGroup,
            logger: logger
        )
        
        // Create the database table
        try await db.execute(
            .init(unsafeSQL:
                           """
                           CREATE TABLE IF NOT EXISTS parks (
                               "id" uuid PRIMARY KEY,
                               "latitude" double NOT NULL,
                               "longitude" double NOT NULL,
                               "name" text NOT NULL
                           );
                           """
                 )
        )
    }
}
```
  

### Step 4.4 Call the `setupDatabase` function

Add `try await setupDatabase()` to `Application+configure.swift`.

```swift
import Hummingbird
import HummingbirdFoundation


public protocol AppArguments {}

public extension HBApplication {
    
    func configure() async throws {
        
        // Setup the database
        try await setupDatabase()
        
        // Set encoder and decoder
        encoder = JSONEncoder()
        decoder = JSONDecoder()
       
        // Logger
        logger.logLevel = .debug
        
        // Middleware
        middleware.add(HBLogRequestsMiddleware(.debug))
        middleware.add(HBCORSMiddleware(
            allowOrigin: .originBased,
            allowHeaders: ["Content-Type"],
            allowMethods: [.GET, .OPTIONS, .POST, .DELETE, .PATCH]
        ))
        
        router.get("/") { _ in
            "The server is running...🚀"
        }
        
        // Additional routes are defined in the controller
        // We want our server to respond on "api/v1/parks"
        ParkController().addRoutes(to: router.group("api/v1/parks"))
    }
}
```


### Step 4.5 Create the park model
  Add the `Park.swift` under `/Source/parkAPI/Models`.

```swift
import Foundation
import Hummingbird

struct Park: Codable {
    let id: UUID
    let latitude: Double
    let longitude: Double
    let name: String
    
    init(id: UUID, latitude: Double, longitude: Double, name: String) {
        self.id = id
        self.latitude = latitude
        self.longitude = longitude
        self.name = name
    }
}

extension Park: HBResponseCodable {}
```

### Step 4.6 Create the park controller
 

 The `Controller` receives an input from the users, then processes the user's data with the help of `Model` and passes the results back. Add `ParkController.swift` to a new `Controllers` folder under `Source/parkAPI/`.
 
 ```swift
import Foundation
import Hummingbird
import HummingbirdDatabase

extension UUID: LosslessStringConvertible {
    
    public init?(_ description: String) {
        self.init(uuidString: description)
    }
}

struct ParkController {
    // Define the table in the databse
    let tableName = "parks"
    
    // The routes for CRUD operations
    func addRoutes(to group: HBRouterGroup) {
        group
            .get(use: list)
    }
    
    // Return all parks
    func list(req: HBRequest) async throws -> [Park] {
        let sql = """
                SELECT * FROM parks
        """
        let query = HBDatabaseQuery(unsafeSQL: sql)
        
        return try await req.db.execute(query, rowType: Park.self)
    }
}
 ```
 
In the controller file, define the table of the database you want to use, ideally it is the same as you defined in the `DatabaseSetup.swift` file. 

Use [`HBRouterGroup`](https://hummingbird-project.github.io/hummingbird-docs/documentation/hummingbirdauth/hbroutergroup) to collect all routes under single path.

#### GET - all parks
Start with listing all elements: `.get(use: list)` Where `get` refers to `GET` method and `use` to the function where you describe what supposed to happen, if you call that endpoint.

The `list()` function returns with the array of `Park` model.  
#### GET - park with {id}
Show park with specified id: `.get(":id", use: show)`.

 ``` swift
 func show(req: HBRequest) async throws -> Park? {
        let id = try req.parameters.require("id", as: UUID.self)
        let sql = """
                SELECT * FROM parks WHERE id = :id:
        """
        let query = HBDatabaseQuery(
            unsafeSQL: sql,
            bindings: ["id": id]
        )
        let rows = try await req.db.execute(query, rowType: Park.self)
        
        return rows.first
    }
```


#### POST - create park
Create new park: `.post(options: .editResponse, use: create)`.

```swift
    func create(req: HBRequest) async throws -> Park {
        struct CreatePark: Decodable {
            let latitude: Double
            let longitude: Double
            let name: String
        }
        
        let park = try req.decode(as: CreatePark.self)
        let id = UUID()
        let row = Park(
            id: id,
            latitude: park.latitude,
            longitude: park.longitude,
            name: park.name
        )
        let sql = """
                INSERT INTO
                    parks (id, latitude, longitude, name)
                VALUES
                    (:id:, :latitude:, :longitude:, :name:)
                """
        
        try await req.db.execute(.init(unsafeSQL: sql, bindings: row))
        req.response.status = .created
```

#### PATCH - update park with {id}
Update park with specified id: `‌.patch(":id", use: update)`

```swift
   func update(req: HBRequest) async throws -> HTTPResponseStatus {
        struct UpdatePark: Decodable {
            var latitude: Double?
            var longitude: Double?
            var name: String?
        }
        let id = try req.parameters.require("id", as: UUID.self)
        let park = try req.decode(as: UpdatePark.self)
        let sql = """
                    UPDATE
                        parks
                      SET
                          "latitude" = CASE WHEN :1: IS NOT NULL THEN :1: ELSE "latitude" END,
                          "longitude" = CASE WHEN :2: IS NOT NULL THEN :2: ELSE "longitude" END,
                          "name" = CASE WHEN :3: IS NOT NULL THEN :3: ELSE "name" END
                      WHERE
                          id = :0:
                    """
        try await req.db.execute(
            .init(
                unsafeSQL:
                    sql,
                bindings:
                    id, park.latitude, park.longitude, park.name
            )
        )
        return .ok
    }
```

As in the `DatabaseSetup.swift` file we defined that none of table columns can be `NULL`, we need to check that the request contains all values of only some of them and update the columns respectively. 

#### DELETE - delete park with {id}
Delete park with specified id: `.delete(":id", use: deletePark)`

```swift
    func deletePark(req: HBRequest) async throws -> HTTPResponseStatus {
        let id = try req.parameters.require("id", as: UUID.self)
        let sql = """
                    DELETE FROM parks WHERE id = :0:
                """
        try await req.db.execute(
            .init(
                unsafeSQL: sql,
                bindings: id
            )
        )
        return .ok
    }
}
```


Our final folder structure looks like this:

```
.
├── Package.swift
├── README.md
└── Sources
    └── parkAPI
        ├── App.swift
        ├── Application+configure.swift
        ├── Controllers
        │   └── ParkController.swift
        ├── Database
        │   └── DatabaseSetup.swift
        └── Models
            └── Park.swift
```

### Step 5: Run the API server:
`swift run parkAPI`

You can reach the server on: `http://127.0.0.1:8080`

### Summary
I was impressed how easily and quickly I could build a working API server using [Hummingbird](https://github.com/hummingbird-project/hummingbird) and FeatherCMS's [Database Component](https://github.com/FeatherCMS/hummingbird-db). Building and running the project took very minimal time comparing to Vapor. I highly recommend to try Hummingbird project in case you want something light and modular on Server side Swift.

You can find the source code [here](https://github.com/kicsipixel/parkAPI).