---
title: "Vapor 4 with Tailwind CSS"
date: 2024-01-17T12:04:31+02:00
tags: ["iOS", "macOS", "swift", "server", "git", "vapor", "api", "css", "tailwind"]
author: ["Szabolcs Tóth"]
draft: false
---

Usually developers prefer to segregate backend and frontend. Several frameworks allow to develop frontend and backend together, [Vapor](https://vapor.codes/) is one of them (i.e. [Ruby on Rails](https://rubyonrails.org/), [Django](https://www.djangoproject.com/), etc).


Time to time heated debates start about the "best" method how to style your frontend. Whether writing CSS totally from scratch is the best or using CSS frameworks(i.e. [Bootstrap](https://getbootstrap.com/), [Foundation](https://get.foundation/), [Bulma](https://bulma.io/)). 

My favourite CSS framework is [TailwindCSS](https://tailwindcss.com/), as I hardly do any web development, my CSS skills are very limited. It helps me a lot, if I need to put together HTML+CSS very fast. 

### Create a new Vapor project:

```bash
vapor new tailwind_vapor -n
```

### Add [Leaf](https://github.com/vapor/leaf), the templating language, to `Package.swift`:

```swift
dependencies: [
    // 💧 A server-side Swift web framework.
    .package(url: "https://github.com/vapor/vapor.git", from: "4.89.0"),
    // Leaf templating language
    .package(url: "https://github.com/vapor/leaf.git", from: "4.0.0")
],
targets: [
    .executableTarget(
        name: "App",
        dependencies: [
            .product(name: "Vapor", package: "vapor"),
            // Leaf templating language
            .product(name: "Leaf", package: "leaf")
        ]
    ),
```

Leaf has multiple purposes, one of them is “to generate dynamic HTML pages for a front-end website or generate rich emails to send from an API” as written in the documentation. 

### Create `PagesController.swift`
We will store the routes and templates in this new controller. 

Create the file, `PagesController.swift` under `Sources/App/Controllers`:

```swift
import Leaf
import Vapor

struct PagesController: RouteCollection {
    func boot(routes: RoutesBuilder) throws {
        routes.get(use: index)
    }
    
    func index(_ req: Request) -> EventLoopFuture<View> {
        return req.view.render("index")
    }
}
```

We set the route and ready to render the file called `index.leaf`

### Create `index.leaf` template
Create two new folders `Resources/Views`. So your folder structure will look like this:

```bash
.
├── Dockerfile
├── Package.resolved
├── Package.swift
├── Public
├── Resources
│   └── Views
├── Sources
│   └── App
│       ├── Controllers
│       │   └── PagesController.swift
│       ├── configure.swift
│       ├── entrypoint.swift
│       └── routes.swift
├── Tests
│   └── AppTests
│       └── AppTests.swift
└── docker-compose.yml
```

Add `index.leaf` file to your new folder:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>Vapor 4 and Tailwind CSS</title>
</head>
<body>
  <h1>It works...perfectly 🚀</h1>
</body>
</html>
```

We need to register our new controller and remove the default `/index` route.

### Register the `Pages` controller
Remove the content of `routes` function in the `routes.swift` and update with the following:

```swift
import Vapor

func routes(_ app: Application) throws {
    try app.register(collection: PagesController())
}
```

The last thing you need to add to `import Leaf`  and `app.views.use(.leaf)` to `configure.swift`:

```swift
import Leaf
import Vapor

// configures your application
public func configure(_ app: Application) async throws {
    // uncomment to serve files from /Public folder
    // app.middleware.use(FileMiddleware(publicDirectory: app.directory.publicDirectory))

    // use Leaf
    app.views.use(.leaf)
    // register routes
    try routes(app)
}
```

You can test your application now:
![](/images/vapor_tailwind1.png)


### Add Tailwind CSS
The easiest way to use Tailwind CSS in your project is [SwiftyTailwind](https://github.com/tuist/SwiftyTailwind). It has a simple but understandable documentation.

Update the `Package.swift`:

```swift
dependencies: [
    // 💧 A server-side Swift web framework.
    .package(url: "https://github.com/vapor/vapor.git", from: "4.89.0"),
    // Leaf templating language
    .package(url: "https://github.com/vapor/leaf.git", from: "4.0.0"),
    // Tailwind CSS
    .package(url: "https://github.com/tuist/SwiftyTailwind.git", .upToNextMinor(from: "0.5.0"))
],
targets: [
    .executableTarget(
        name: "App",
        dependencies: [
            .product(name: "Vapor", package: "vapor"),
            // Leaf templating language
            .product(name: "Leaf", package: "leaf"),
            // Tailwind CSS
            .product(name: "SwiftyTailwind", package: "SwiftyTailwind")
        ]
    ),
]
```

#### Create `tailwind.swift` file
Under `Sources/App` create a file called, `tailwind.swift` and add:

```swift
import SwiftyTailwind
import TSCBasic
import Vapor

func tailwind(_ app: Application) async throws {
  let resourecesDirectory = try AbsolutePath(validating: app.directory.resourcesDirectory)
  let publicDirectory = try AbsolutePath(validating: app.directory.publicDirectory)

  let tailwind = SwiftyTailwind()
  try await tailwind.run(
    input: .init(validating: "Styles/app.css", relativeTo: resourecesDirectory),
    output: .init(validating: "styles/app.generated.css", relativeTo: publicDirectory),
    options: .content("\(app.directory.viewsDirectory)/**/*.leaf")
  )
}
```

#### Update `configure.swift`
First uncomment this line to use Public folder:

```swift
app.middleware.use(FileMiddleware(publicDirectory: app.directory.publicDirectory))
```
Then add `try await tailwind(app)` to use the earlier created `tailwind` function. 

Your final `configure.swift` file looks:

```swift
// configures your application
public func configure(_ app: Application) async throws {
    // uncomment to serve files from /Public folder
    app.middleware.use(FileMiddleware(publicDirectory: app.directory.publicDirectory))
    
    // use Leaf
    app.views.use(.leaf)
    
    // use Tailwind CSS
    try await tailwind(app)
    
    // register routes
    try routes(app)
}
```

#### Add stylesheet to `index.leaf`
Add the following to the `<head>` section: 

```html
<link rel="stylesheet" href="/styles/app.generated.css" />
```

Your final `index.leaf`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Vapor 4 and Tailwind CSS</title>
    <link rel="stylesheet" href="/styles/app.generated.css" />
  </head>
  <body>
    <div class="flex h-screen justify-center items-center bg-black">
      <div class="text-center">
        <h1 class="text-xl text-white">It works...perfectly 🚀</h1>
      </div>
    </div>
  </body>
</html>
```

#### Add `app.css`
Add the initial `app.css` to `/Resources/Styles` folder:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

If you run your application now, you will see:
![](/images/vapor_tailwind2.png)

Final folder structure looks like:

```bash
.
├── Dockerfile
├── Package.resolved
├── Package.swift
├── Public
│   └── styles
│       └── app.generated.css
├── Resources
│   ├── Styles
│   │   └── app.css
│   └── Views
│       └── index.leaf
├── Sources
│   └── App
│       ├── Controllers
│       │   └── PagesController.swift
│       ├── configure.swift
│       ├── entrypoint.swift
│       ├── routes.swift
│       └── tailwind.swift
├── Tests
│   └── AppTests
│       └── AppTests.swift
└── docker-compose.yml
```

With the help of Tailwind CSS we can create beautiful webpages, without writing a lots of CSS codes.

Source code is [here](https://github.com/kicsipixel/vapor_tailwindcss).
