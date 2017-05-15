![Eureka: Elegant form builder in Swift](Eureka.jpg)

<p align="center">
<a href="https://travis-ci.org/xmartlabs/Eureka"><img src="https://travis-ci.org/xmartlabs/Eureka.svg?branch=master" alt="Build status" /></a>
<img src="https://img.shields.io/badge/platform-iOS-blue.svg?style=flat" alt="Platform iOS" />
<a href="https://developer.apple.com/swift"><img src="https://img.shields.io/badge/swift3-compatible-4BC51D.svg?style=flat" alt="Swift 3 compatible" /></a>
<a href="https://github.com/Carthage/Carthage"><img src="https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat" alt="Carthage compatible" /></a>
<a href="https://cocoapods.org/pods/Eureka"><img src="https://img.shields.io/cocoapods/v/Eureka.svg" alt="CocoaPods compatible" /></a>
<a href="https://raw.githubusercontent.com/xmartlabs/Eureka/master/LICENSE"><img src="http://img.shields.io/badge/license-MIT-blue.svg?style=flat" alt="License: MIT" /></a>
<a href="https://codebeat.co/projects/github-com-xmartlabs-eureka"><img alt="codebeat badge" src="https://codebeat.co/badges/16f29afb-f072-4633-9497-333c6eb71263" /></a>
</p>

Made with ❤️ by [XMARTLABS](http://xmartlabs.com). This is the re-creation of [XLForm] in Swift.

* **PlacePickerRow** 
<img src="Example/Media/RowGifs/GooglePlacesCustomRow.gif" width="300" alt="Screenshot of Place Picker Row"/>

You need 3 libraries. You can check <a href="https://developers.google.com/places/ios-api/start">here</a> for Google documentation.

```ruby
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '9.0'
use_frameworks!

pod 'Eureka'
pod 'GoogleMaps'
pod 'GooglePlaces'
pod 'GooglePlacePicker'
```

After installing the necessary libraries. You need to set API keys in AppDelegate file:

```swift
import GoogleMaps
import GooglePlaces

let apiKey = "AIzaSyBTFypi6rTHlqqqnaHILPaNDlhdej4D7ms"
GMSServices.provideAPIKey(apiKey)
GMSPlacesClient.provideAPIKey(apiKey)
```

I wanted to create a model. Which is for storing location and the place name inside the row.

```swift
public class PlaceModel : Equatable{
    public static func ==(lhs: PlaceModel, rhs: PlaceModel) -> Bool {
        return lhs.location.latitude == rhs.location.latitude &&
            lhs.location.longitude == rhs.location.longitude &&
            lhs.placeName == rhs.placeName
    }

    var placeName: String?
    var location: CLLocationCoordinate2D = CLLocationCoordinate2D(latitude: 0, longitude: 0)
}
```

Then create custom cells with the PlaceModel

```swift
public class PlacePickerCell<T: Equatable> : Cell<T>, CellType {
    required public init(style: UITableViewCellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)
        
    }
    required public init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
    }
    public override func setup() {
        super.setup()
        
    }
    public override func update() {
        super.update()
    }
    public override func didSelect() {
        super.didSelect()
        row.deselect()
    }
}


public class _PlacePickerRow : Row<PlacePickerCell<PlaceModel>>, NoValueDisplayTextConformance {
    public var noValueDisplayText: String?
    
    required public init(tag: String?) {
        super.init(tag: tag)
        noValueDisplayText = "Not Picked Yet"
        displayValueFor = {
            return $0?.placeName // display placeName from PlaceModel if a place is picked
        }
    }
    
}

public final class PlacePickerRow<T> : _PlacePickerRow, RowType where T: Equatable {
    
    var center = CLLocationCoordinate2D(latitude: 0, longitude: 0)
    
    required public init(tag: String?) {
        super.init(tag: tag)
    }
    
    // when custom row is selected promt Google Place Picker
    public override func customDidSelect() {
        super.customDidSelect()
        if !isDisabled {
            let northEast = CLLocationCoordinate2D(latitude: center.latitude + 0.001, longitude: center.longitude + 0.001)
            let southWest = CLLocationCoordinate2D(latitude: center.latitude - 0.001, longitude: center.longitude - 0.001)
            let viewport = GMSCoordinateBounds(coordinate: northEast, coordinate: southWest)
            let config = GMSPlacePickerConfig(viewport: viewport)
            let placePicker = GMSPlacePicker(config: config)
            
            placePicker.pickPlace(callback: {(place, error) -> Void in
                if let error = error {
                    print("Pick Place error: \(error.localizedDescription)")
                    return
                }
                
                if let place = place, let value = self.value {
                    
                    value.placeName = place.name
                    value.location = place.coordinate
		    self.center = place.coordinate
                    
                    self.updateCell()
                } else {
                    // handle the error
                }
            })
        }
    }
}
```
And you can add it to your form this way:

```swift
+++ Section("Where")
    <<< PlacePickerRow<PlaceModel>("PlacePickerRow") { (row : PlacePickerRow<PlaceModel>) -> Void in
	row.title = "Pick a Place"
	row.value = PlaceModel()
	row.center.latitude = 37.788204
	row.center.longitude = -122.411937
    }
```

That's it! 
