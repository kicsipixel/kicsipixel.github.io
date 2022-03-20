---
title: "Very simple view-based NSTableView in Swift 4.2 using Model"
date: 2018-11-17
tags: ["macOS", "Cocoa", "Xcode", "Swift", "NSTableView"]
author: ["Szabolcs Tóth"]
cover:
    image: /images/model.png
draft: false
---

This tutorial is very similar to the previous one, the only difference that now, we will use a Person model, that will make you life easier.

So, the final will be the same:
![final][NSTableview1]

**Step 1. Create a Cocoa Application project.**
![create][NSTableview1-1]
![name][NSTableview1-2]

**Step 2. Add NSTableView to the Main Storyboard and customize a little bit**
![addingtable][NSTableview1-3]

- Increase the number of columns from 2 to 3.
![addingcolumn][NSTableview1-4]

- Click on the header and name the columns
![header][NSTableview1-5]
- We need to add identifier to the columns and cells.
![columns][NSTableview1-6]
![cells][NSTableview1-7]
You should repeat this to the Last name(lastName) and Mobile number(mobileNumber) columns and cells as well.

**Create an IBOutlet and connect it to your tableview but don’t forget the dataSource and delegate either.**

![outlet][NSTableview1-8]

``` swift
@IBOutlet var tableView: NSTableView!
```
<br />

![delegate][NSTableview1-9]


So, instead of creating an Array with Dictionary, we add a new file.
![model][NSTableview2-1]
``` swift
import Cocoa

struct Person {
    let firstName: String
    let lastName: String
    let mobileNumber: String
}
```

We have the Person model, we need to add some values to it, but first create an array.

``` swift
@IBOutlet var tableView: NSTableView!
var people: [Person] = []
```

``` swift
override func viewDidLoad() {
	super.viewDidLoad()
	
	let person1 = Person.init(firstName: "Ragnar", lastName: "Lothbrok", mobileNumber: "555-1234")
	let person2 = Person.init(firstName: "Bjorn", lastName: "Lothbrok", mobileNumber: "555-3412")
	let person3 = Person.init(firstName: "Harald", lastName: "Finehair", mobileNumber: "555-4512")
	people.append(person1)
	people.append(person2)
	people.append(person3)
}
```

We need to implement the following two methods:
- How many rows will we need? <br />
``` numberOfRows(in tableView: NSTableView) -> Int
```
- and what will be the values of the cells <br />
```tableView(_ tableView: NSTableView, viewFor tableColumn: NSTableColumn?, row: Int) -> NSView?
 ```

It is considered to be a good practice to separate delegates from the main code as extension, so the following won't be added directly to the ```ViewController```.

``` swift
func numberOfRows(in tableView: NSTableView) -> Int {
      return (people.count)
  }
```

We would like to match the NSDictionary key to the column identifier and give the NSDictionary value to the cell. 

``` swift
func tableView(_ tableView: NSTableView, viewFor tableColumn: NSTableColumn?, row: Int) -> NSView? {
  let person = people[row]
  
  guard let cell = tableView.makeView(withIdentifier: tableColumn!.identifier, owner: self) as? NSTableCellView else { return nil }
  
  if (tableColumn?.identifier)!.rawValue == "firstName" {
      cell.textField?.stringValue = person.firstName
  } else if (tableColumn?.identifier)!.rawValue == "lastName" {
      cell.textField?.stringValue = person.lastName
  } else {
      cell.textField?.stringValue = person.mobileNumber
  }
  
  return cell
}
```
It is very important that your NSDictionary keys must match with the identifiers of your columns otherwise it won’t work.

I copy the full code here below.
``` swift
mport Cocoa

class ViewController: NSViewController {
    
  @IBOutlet var tableView: NSTableView!
  var people: [Person] = []

  override func viewDidLoad() {
    super.viewDidLoad()
    
    let person1 = Person.init(firstName: "Ragnar", lastName: "Lothbrok", mobileNumber: "555-1234")
    let person2 = Person.init(firstName: "Bjorn", lastName: "Lothbrok", mobileNumber: "555-3412")
    let person3 = Person.init(firstName: "Harald", lastName: "Finehair", mobileNumber: "555-4512")
    people.append(person1)
    people.append(person2)
    people.append(person3)
  }
}

extension ViewController: NSTableViewDataSource, NSTableViewDelegate {
    
  func numberOfRows(in tableView: NSTableView) -> Int {
      return (people.count)
  }

  func tableView(_ tableView: NSTableView, viewFor tableColumn: NSTableColumn?, row: Int) -> NSView? {
      let person = people[row]
      
      guard let cell = tableView.makeView(withIdentifier: tableColumn!.identifier, owner: self) as? NSTableCellView else { return nil }
      
      if (tableColumn?.identifier)!.rawValue == "firstName" {
          cell.textField?.stringValue = person.firstName
      } else if (tableColumn?.identifier)!.rawValue == "lastName" {
          cell.textField?.stringValue = person.lastName
      } else {
          cell.textField?.stringValue = person.mobileNumber
      }
      
      return cell
  }
}
```

Source code: [Github](https://github.com/kicsipixel/Cocoa-Samples/tree/master/NSTableView1)


[NSTableview1]:   /images/NSTableView1.png
[NSTableview1-1]: /images/NSTableView1-1.png
[NSTableview1-2]: /images/NSTableView1-2.png
[NSTableview1-3]: /images/NSTableView1-3.png
[NSTableview1-4]: /images/NSTableView1-4.png
[NSTableview1-5]: /images/NSTableView1-5.png
[NSTableview1-6]: /images/NSTableView1-6.png
[NSTableview1-7]: /images/NSTableView1-7.png
[NSTableview1-8]: /images/NSTableView1-8.gif
[NSTableview1-9]: /images/NSTableView1-9.gif
[NSTableview2-1]: /images/NSTableView2-1.png