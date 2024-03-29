---
title: "Very simple view-based NSTableView in Swift 4.2"
date: 2018-11-16
tags: ["macOS", "Cocoa", "Xcode", "Swift", "NSTableView"]
author: ["Szabolcs Tóth"]
draft: false
---

It is widely known that while the internet is full with iOS tutorials you hardly find any Cocoa ones. Many things on iOS with CocoaTouch is simpler than on OS X, so it is not obvious that after reading some tutorials, you will be able code on both platform.

I’d like to share what I learn helping others and receiving advises from experienced fellows. My first application is a simple phonebook using view-based NSTableView. On the internet you can find plenty cell-based NSTableView tutorials with single column using NSArray.

This is what we are building:
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


The rest of is very easy now. Let’s have an NSArray with NSDictionary:

``` swift
// our variable
var data:[[String: String]] = [[:]
```

``` swift
// adding people
data = [
 [
  "firstName" : "Ragnar",
  "lastName" : "Lothbrok",
  "mobileNumber" : "555-1234"
 ],
 [
  "firstName" : "Bjorn",
  "lastName" : "Lothbrok",
  "mobileNumber" : "555-3412"
 ],
 [
  "firstName" : "Harald",
  "lastName" : "Finehair",
  "mobileNumber" : "555-4512"
 ]
]
```

We need to implement the following two methods:
- How many rows will we need?

``` swift
numberOfRows(in tableView: NSTableView) -> Int
```
- and what will be the values of the cells?

``` swift
tableView(_ tableView: NSTableView, viewFor tableColumn: NSTableColumn?, row: Int) -> NSView
 ```

It is considered to be a good practice to separate delegates from the main code as extension, so the following won't be added directly to the ```ViewController```.

``` swift
func numberOfRows(in tableView: NSTableView) -> Int {
      return (people.count)
  }
```

We would like to match the NSDictionary key to the column identifier and give the NSDictionary value to the cell. We can do it with ```if / else if / else``` or ```switch```, but let's try to make something smarter/shorter.  

``` swift
func tableView(_ tableView: NSTableView, viewFor tableColumn: NSTableColumn?, row: Int) -> NSView? {
  let person = data[row]

  let cell = tableView.makeView(withIdentifier: (tableColumn!.identifier), owner: self) as? NSTableCellView
  cell?.textField?.stringValue = person[tableColumn!.identifier.rawValue]!

  return cell
}
```
It is very important that your NSDictionary keys must match with the identifiers of your columns otherwise it won’t work.

I copy the full code here below.
``` swift
import Cocoa

class ViewController: NSViewController {

  @IBOutlet var tableView: NSTableView!

  // our variable
  var data: [[String: String]] = [[:]]

  override func viewDidLoad() {
    super.viewDidLoad()
    
    // adding people
    data = [
        [
            "firstName" : "Ragnar",
            "lastName" : "Lothbrok",
            "mobileNumber" : "555-1234"
        ],
        [
            "firstName" : "Bjorn",
            "lastName" : "Lothbrok",
            "mobileNumber" : "555-3412"
        ],
        [
            "firstName" : "Harald",
            "lastName" : "Finehair",
            "mobileNumber" : "555-4512"
        ]
    ]
    
    // reload tableview
    tableView.reloadData()
  }

}

extension ViewController: NSTableViewDataSource, NSTableViewDelegate {
    
  func numberOfRows(in tableView: NSTableView) -> Int {
    return (data.count)
  }

  func tableView(_ tableView: NSTableView, viewFor tableColumn: NSTableColumn?, row: Int) -> NSView? {
    let person = data[row]
    
    guard let cell = tableView.makeView(withIdentifier: tableColumn!.identifier, owner: self) as? NSTableCellView else { return nil }
    cell.textField?.stringValue = person[tableColumn!.identifier.rawValue]!
    
    return cell
  }
}
```

Source code: [Github](https://github.com/kicsipixel/Cocoa-Samples/tree/master/NSTableView)


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