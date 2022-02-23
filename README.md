# [CloudKit](https://github.com/raywenderlich/video-ck-materials)

<img width="803" src="https://user-images.githubusercontent.com/47273077/155275069-45b3d527-24dd-4d05-8a6c-a13d22c4cc0b.png">

<img width="300" src="https://github.com/YamamotoDesu/CloudKit/blob/main/BabiFud/Gif/CloudKitDemo.gif">
<img width="841" alt="スクリーンショット_2022_02_23_16_12" src="https://user-images.githubusercontent.com/47273077/155275258-6ab6210e-0282-4686-b023-a6ecc7b2d526.png">

## 1. Set Up Entitlements  
<img width="606" src="https://user-images.githubusercontent.com/47273077/155251371-744389b8-c91a-4ce6-a6fd-5fbaebb5aed7.png">

![Entitlement](https://user-images.githubusercontent.com/47273077/155252000-c8b8b286-3e36-41f6-a921-96238b7df771.jpeg)

------




## 2. Explore The CloudKit Console  
###  Open The Cloudkit Console 
<img width="1375" src="https://user-images.githubusercontent.com/47273077/155252507-173b813a-3b67-47ab-b6f4-8cb2c790d8b2.png">

### Set Your Container
<img width="376"  src="https://user-images.githubusercontent.com/47273077/155256359-a821002c-8f54-4414-9dc9-cd1b4e3b5b0a.png">

### Create a New Record Type
<img width="762" src="https://user-images.githubusercontent.com/47273077/155256777-318d1b2b-6985-4501-b780-66944563ee57.png">

Example of a Record Type
<img width="855" src="https://user-images.githubusercontent.com/47273077/155256960-bb094f60-aac0-4215-9605-10768bd77133.png">

------



## 3. Set Indexes to The New Record
<img width="479" src="https://user-images.githubusercontent.com/47273077/155257232-65a6a2d4-88fd-4b6c-8860-4097e90ad058.png">
<img width="514" src="https://user-images.githubusercontent.com/47273077/155257378-751c366c-6452-44b4-aa35-a535b336e225.png">

------



## 4. Create CKRecords 
<img width="604" alt="CloudKit_-_Records_と_CloudKit_README_md_at_main_·_YamamotoDesu_CloudKit" src="https://user-images.githubusercontent.com/47273077/155258704-0e6cc74d-1e5f-4448-acec-92c9609bb4b0.png">
 
Example of a CKRecord
<img width="379" src="https://user-images.githubusercontent.com/47273077/155258554-291200d4-3675-4805-b60d-b16873544873.png">

----

## 5. Create CKQueries(Example)  
```swift 
import Foundation
import CloudKit

final class Model {
  static let current = Model()

  private(set) var establishments: [Establishment] = []
  
  func refresh() async throws {
    let query = CKQuery(
      recordType: "\(Establishment.self)",
      predicate: .init(value: true)
    )
  }
}
```

6. Fetch Records(Example)

### Example of the Model
```swift 
import Foundation
import CloudKit

final class Model {
  static let current = Model()

  private(set) var establishments: [Establishment] = []
  
  func refresh() async throws {
    let query = CKQuery(
      recordType: "\(Establishment.self)",
      predicate: .init(value: true)
    )
    
    let database = CKContainer.default().publicCloudDatabase
    let records = try await database.records(matching: query)
      .matchResults.map { try $1.get() }
  }
}
```

### Example of the entity
```swift 
import UIKit
import MapKit
import CloudKit
import CoreLocation

class Establishment {
  enum ChangingTable: Int {
    case none
    case womens
    case mens
    case both
  }
  
  static let recordType = "Establishment"
  private let id: CKRecord.ID
  let name: String
  let location: CLLocation
  let coverPhoto: CKAsset?
  let database: CKDatabase
  let changingTable: ChangingTable
  let kidsMenu: Bool
  let healthyOption: Bool
  let notes: [Note]
  
  init?(record: CKRecord, database: CKDatabase) async throws {
    guard
      let name = record["name"] as? String,
      let location = record["location"] as? CLLocation
    else { return nil }

    id = record.recordID
    self.name = name
    self.location = location
    coverPhoto = record["coverPhoto"] as? CKAsset
    self.database = database
    healthyOption = record["healthyOption"] as? Bool ?? false
    kidsMenu = record["kidsMenu"] as? Bool ?? false

    self.changingTable =
      (record["changingTable"] as? Int).flatMap(ChangingTable.init)
      ?? .none


    notes = []
  }

  func loadCoverPhoto() async throws -> UIImage? {
    nil
  }
}

extension Establishment: Hashable {
  static func == (lhs: Establishment, rhs: Establishment) -> Bool {
    return lhs.id == rhs.id
  }
  
  func hash(into hasher: inout Hasher) {
    hasher.combine(id)
  }
}
```

### Run your App and Check out your received data
<img width="1227"  src="https://user-images.githubusercontent.com/47273077/155268430-e80a947a-ca6e-4bfc-b1b9-cb4e75d81f6e.png">

### Laod Store's Images 
```swift
  func loadCoverPhoto() async throws -> UIImage? {
    guard let fileURL = coverPhoto.flatMap(\.fileURL)
    else { return nil }
    
    return try await Task {
      UIImage(data: try .init(contentsOf: fileURL))
    }.value
  }
```

<img width="490" src="https://user-images.githubusercontent.com/47273077/155269655-570a4cb3-c170-410d-b9d1-098b6564de17.png">

<img width="490" src="https://user-images.githubusercontent.com/47273077/155269691-2cb8176e-307a-4b9c-8187-99dedca0a463.png">


