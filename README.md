# iFoodTrack Dashboard
iFoodTrack Dashboard App (originally called "Food String Tracker") was the original scratching that itch App. It's a macOS App built when Objective-C was king, using AppKit and the Xcode Interface Builder. It utilizes the USDA FoodCentral database.

iFoodTrack Dashboard is organized as a single window dashboard-style App. It presents a library/favorites/custom meals outlineView (left side), a middle detail view, for diary details (top) and chart details (bottom), and a right extended details view describing nutrient data absolute values, averages and mean values. 

It can do quite a lot. Besides looking up library food details, you can save favorites, create custom foods or custom meals, maintain a food diary and track multiple nutrients in a single graph.

The App also can save/open food diary and favorites files, and export data in csv format.
<br></br>

# Technologies Used
## Languages and Frameworks							
* Objective-C Programming language
* Assembly Language
* AppKit framework

## Apple Technologies
* Interface Builder
* Core Graphics
* AutoLayout
* Unit Testing
* Documentation (DocC)

## Other
* Git Version Control
<br></br>

# iFoodTrack Dashboard Animation (placeholder)
<figure>
	<div>
		<video width="500" controls poster="images/screenshots/02a iFTmacOS main window TimeChart light.png" muted preload="auto">
			<source src="videos/iFoodTrack DashboardmacOS_compressed.mp4" type="video/mp4">
			<!-- For non-HTML5 browsers: -->
			Your browser doesn't support the video tag. Click <a href=http://www.firefox.com>here</a> 
			to download the Firefox browser for your operating system.
		</video>
	</div>
</figure>

# A macOS (i.e., Mac OSX) Cocoa/Interface Builder Project...
This project was written entirely without using any dependency and relied on a lot of Objective-C code. Some of the challenges including updating data for the various views in the dashboard main window and doing all the math for graphing the nutrient chart data.
<br></br>

## Sample Code:
### Plotting daily nutrient totals in time-bar charts (placeholder):
```swift
/// Will draw nutrient-specific nutrient total value per date bar of bar graph.
    func drawBarGraphDataSegments(_ cgContext: CGContext?) {
        let lineWidth    = 2.0                              // initialize with zero value
        var y            = lineWidth - 1.0 + xLabelHeight   // start half way up width of axis line
        
        // total the value of all bars ( = "segments") by using reduce to sum them
        guard let maxValue  = segments.map({Double($0.value)}).max() else { return }
        var maxBarHeight    = maxValue > 0 ? maxValue : 100.0   // protects against dividing by zero.
        
        // Check if drv100Percent value is greater than maxValue, then set max to drv100Percent value
        maxBarHeight        = drv100PercentValue >= maxValue ? drv100PercentValue : maxValue
        
        let roundedRect     = FBRoundedRect(radius: 2.5, corners: [.bottomLeft, .bottomRight])
        // To round the top of bar, use bottomLeft and bottomRight, since drawing starts from bottom to top
        
        // Loop through the values array:
        for (index, segment) in segments.enumerated() {
            let i: CGFloat  = CGFloat(index)
            let x: CGFloat  = i * (barWidth + spacing) + offset + yLabelHeight
            let value       = segment.value > 0 ? segment.value : 0 // set to zero if value is negative.
            
            let barHeight   = (value/maxBarHeight * (viewHeight - xLabelHeight))
            if barHeight == 0 { continue }  // If barHeight was zero, then skip drawing the bar! Prevents drawing artifacts on X-Axis.
            
            let bar         = CGRect(x: x, y: y, width: barWidth, height: barHeight)
            var path        = CGPath(rect: .zero, transform: nil)
            path            = roundedRect.path(in: bar) ?? CGPath(roundedRect: bar, cornerWidth: 2.5, cornerHeight: 2.5, transform: .none)
            
            cgContext?.setFillColor(self.nutrientColor.cgColor)
            cgContext?.addPath(path)
            cgContext?.fillPath()
            
            y = lineWidth - 1.0 + xLabelHeight     // reset y
        }
    }
```
<br>
</br>

### A test for validating nutrient value calculations:
```swift
/// Test will test Food Nutrient DRV percent calculation using DRV nutrient defaults. The result should yield 100% for all DRV Nutrient calculations.
    func testCalculateNutValues_recalculateDRVPercent() {
        // Arrange
        let other6NutItemIndexes: [kNutrientNameIndex] = FBDRVSettingsHelper.shared.nutNameIndexes
        
        // MARK: Set up NutIDs:
        // topNutIDs, midNutIDs, and bottomNutIDs split 'other6NutIems' into subarray item pairs:
        var otherNutIDs: (topPair:[kNutrientID], midPair:[kNutrientID], bottomPair:[kNutrientID]) {
            let topPair = other6NutItemIndexes[0...1].map({$0.rawValue})
            let midPair = other6NutItemIndexes[2...3].map({$0.rawValue})
            let btmPair = other6NutItemIndexes[4...5].map({$0.rawValue})
            let NutPairs = [topPair, midPair, btmPair]
            let NutIDPairs = NutPairs.map{$0.map{kNutrientID.allNutrientIDs[$0]}}
            return (NutIDPairs[0], NutIDPairs[1], NutIDPairs[2])
        }
        
        var topNutIDs: [kNutrientID]    { otherNutIDs.topPair } // recompute .topPair
        var midNutIDs: [kNutrientID]    { otherNutIDs.midPair } // recompute .midPair
        var bottomNutIDs: [kNutrientID] { otherNutIDs.bottomPair } // recompute .bottomPair
        
        let otherNutIDsArrays   = [topNutIDs, midNutIDs, bottomNutIDs]
        
        // MARK: Create DRV Food (a token food):
        let nutIDs      = kNutrientID.allCases.map{$0.rawValue}
        let nutNames    = kNutrientName.allCases.map{$0.rawValue}
        let nutNumbers  = kNutrientNameIndex.allNutrientIndexes
        let nutModifers = kNutrientNameIndex.allUnitModifiers
        let nutDRV      = NutrientDRV()

        var nutrients: [FDRNutrient] = []

        // Iterate through all nutrients, based on nutrient name case number (index) in kNutrientNameIndex enum:
        for n in nutNumbers {
            nutrients.append(FDRNutrient(amount: nutDRV.nutrientDRVDict[kNutrientNameIndex(rawValue: n)!], nutrient: FDRNutInfo(id: nutIDs[n], number: "0", name: nutNames[n], rank: 0, unitName: nutModifers[n])))
        }
        
        
        var drvFood  = FDRFood(nutrients: nutrients)
        var drvAbsAmountLabelsArray: [FBNutAmountsAndLabels] = []
        let graphVC = FBGraphVC()
        
//        let expectedOtherNutIDsArrays: [FBNutAmountsAndLabels] = [
//        FBNutAmountsAndLabels(nutAmounts: (drvAmounts: [100, 100], absAmounts: [20, 28]), nutLabels: (drvLabels: ["%", "%"], absLabels: ["g", "g"])),
//        FBNutAmountsAndLabels(nutAmounts: (drvAmounts: [100, 100], absAmounts: [90, 300]), nutLabels: (drvLabels: ["%", "%"], absLabels: ["g", "mg"])),
//        FBNutAmountsAndLabels(nutAmounts: (drvAmounts: [100, 100], absAmounts: [1300, 2300]), nutLabels: (drvLabels: ["%", "%"], absLabels: ["mg", "mg"]))
//        ]
        
        // Act
        // Calculate selected Food amounts:
        for otherNutIDsArray in otherNutIDsArrays {
            guard let nutAmountsAndLabels = drvFood.calculateNutValues(nutrientIDs: otherNutIDsArray) else { return }
            drvAbsAmountLabelsArray.append(nutAmountsAndLabels)
        }
        
        let drvOtherNutIdsAmounts = drvAbsAmountLabelsArray.map{$0.nutAmounts}.map{$0.drvAmounts}
        let expectedDRVOtherNutIdsAmounts = [[100.0, 100.0], [100.0, 100.0], [100.0, 100.0]]
        
        // Assert
        XCTAssert(drvOtherNutIdsAmounts == expectedDRVOtherNutIdsAmounts)
        
    }
```
<br>
</br>


# Sample Screen Shots (Placeholders, include Settings, Custom Foods, Custom Meals Windows)
<table>
<tr>
	<td>
	<img src="images/screenshots/01a iFTmacOS main window Diary light.png" alt="blue marble weather daily forecast" width="500"/>
	</td>
	<td>
	<img src="images/screenshots/01b iFTmacOS main window Diary dark.png" alt="blue marble weather daily forecast" width="500"/>
	</td>
</tr>
<tr>
	<td>
	<img src="images/screenshots/02a iFTmacOS main window TimeChart light.png" alt="blue marble weather daily forecast" width="500"/>
	</td>
	<td>
	<img src="images/screenshots/05a iFTmacOS main window Trash light.png" alt="blue marble weather daily forecast" width="500"/>
	</td>
</tr>
<tr>
	<td>
	<img src="images/screenshots/08a iFTmacOS Settings Appearance light.png" alt="blue marble weather daily forecast" width="500"/>
	</td>
	<td>
	<img src="images/screenshots/06a iFTmacOS Settings Advanced light.png" alt="blue marble weather daily forecast" width="500"/>
	</td>
</tr>
<tr>
	<td>
	<img src="images/screenshots/08a iFTmacOS Settings Appearance light.png" alt="blue marble weather daily forecast" width="500"/>
	</td>
	<td>
	<img src="images/screenshots/012a Settings Window Demo light.png" alt="blue marble weather daily forecast" width="400"/>
	</td>
</tr>
</table>

<!-- ![readme-bluemarbleweather-daily](../images/BMW%20MainView%20Daily.png) -->

## I've implemented the following:
#### Code Structure
* Model-View-Controller Design Pattern
* Delegates and Protocols, NSViewController/NSView
* Navigation Split Views
* OutlineViews
* Parsing a CSV database

#### Testing/Error Handling
* Unit Testing
* Error handling
* Alerts
* Empty states
* Text input validation

#### Customization
* Toolbar and Menu selections
* Settings Window with TabBar subviews
* Light and Dark Mode Selections
* Unit conversions

#### Project Organization
* Code documentation (DocC)
* Help Book manual
* Privacy Manifest
* Group folder organization


# Future Considerations
* Sync with HealthKit, for newer macOS versions
* Implement Core Data
* Re-write using Swift/SwiftUI?




