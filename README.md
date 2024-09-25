# iFoodTrack Dashboard
iFoodTrack Dashboard App (originally called "Food String Tracker") was the original scratching that itch App. It's a macOS App built when Objective-C was king, using AppKit and the Xcode Interface Builder. It utilizes the USDA FoodCentral database.

iFoodTrack Dashboard is organized as a single window dashboard-style App. It presents a library/favorites/custom meals outlineView (left side), a middle detail view, for diary details (top) and chart details (bottom), and a right extended details view describing nutrient data absolute values, averages and mean values. 

It can do quite a lot. Besides looking up library food details, you can save favorites, create custom foods or custom meals, maintain a food diary and track multiple nutrients in a single graph and in the extended-detail right window panel.

The App can also open or save food diary and favorites files, and export data in csv format.
<br></br>

# Technologies Used
## Languages and Frameworks							
* Objective-C Programming language
* InterOp with Swift and Assembly
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


# iFoodTrack Dashboard Animation
<figure>
	<div>
		<video width="500" controls poster="images/screenshots/iFTDash library.png" muted preload="auto">
			<source src="videos/iFTDash Screen Recording_compressed.mp4" type="video/mp4">
			<!-- For non-HTML5 browsers: -->
			Your browser doesn't support the video tag. Click <a href=http://www.firefox.com>here</a> 
			to download the Firefox browser for your operating system.
		</video>
	</div>
</figure>

# A traditional macOS (i.e., Mac OSX) Cocoa/Interface Builder Project...
The motivation for building a great nutrient tracker started with this older Objective-C/Cocoa project, written entirely without external frameworks/APIs. One of the first problems was parsing the many CSV food data files (when JSON was not yet available). Another was doing all the math for graphing the nutrient chart data.
<br></br>


## Here's Sample Code:
### Parsing CSV files:
```objective-c
//MARK: - parseFile:
-(NSMutableArray *)parseFile:(NSURL *)fileURL {
    // ------------------------------------------------------------------------------------------
    // File is broken up into an array of lines (of type NSString).
    // Each line is then broken up into an array of objects (of type NSString) = "Descriptors"
    // ------------------------------------------------------------------------------------------
    myFoodStringFile = [NSString stringWithContentsOfURL:fileURL
                                                encoding:NSASCIIStringEncoding error:nil];
    
    // File is divided up as a series of lines that become elements of the fileLineStringArray array.
    NSArray *fileLineStringArray = [myFoodStringFile componentsSeparatedByString:@"\n"];
    
    // Define Delimiters in file to be parsed.  Only one.  It's the ^ character.
    NSString        *myDelimiters = @"^";
    NSCharacterSet  *customChars = [NSCharacterSet characterSetWithCharactersInString:myDelimiters];
    
    // Define characters to trim from parsed objects. Trim ~ characters from ends of NSString objects.
    // NB: \r is the character for a carriage return (or a "return" character)... this trims objects at end of lines that have both ~ and \r,
    // (otherwise, if only look for ~, \r at end of string will keep the tail end ~ from getting trimmed off)
    NSString        *myTrimmedCharacters = @"~\r";
    NSCharacterSet  *myTrimmer = [NSCharacterSet characterSetWithCharactersInString:myTrimmedCharacters];
    NSString        *trimmedString;
    
    // Array for storing the trimmed fileLineStringArray
    NSMutableArray *fileLineStringArrayTrimmed = [[NSMutableArray alloc] init];
    
    for (NSString *aLineString in fileLineStringArray) {
        // Now, divide up the components of each food string into a "child" array of strings.
        // identify and separate components based on delimiters, and will remove delimiters from component objects.
        NSArray         *componentsOfLineString         = [aLineString componentsSeparatedByCharactersInSet:customChars];
        NSMutableArray  *lineStringComponentsTrimmed    = [[NSMutableArray alloc] init];
        
        printf("#%lu:", (unsigned long)[fileLineStringArray indexOfObject:aLineString]);
        
        for (NSString *foodData in componentsOfLineString) {
            printf(" %s", [foodData UTF8String]);    // use printf so that we can get the component info all on one line
            trimmedString = [foodData stringByTrimmingCharactersInSet:myTrimmer];   // trim ~ characters off ends of string objects
            
            [lineStringComponentsTrimmed addObject:trimmedString];
            
        }
        printf("\n");
        if ([lineStringComponentsTrimmed count]) {
            [fileLineStringArrayTrimmed addObject:lineStringComponentsTrimmed];
        }
    }
    
    return fileLineStringArrayTrimmed;
}
```
<br></br>


### Create the nutrient data lines (bezierPaths) for the chart data:
```objective-c
/// Create bezierPath for Nutrient with tag.
-(NSBezierPath *)createBezierPathforNutrientWithTag:(long int)dataTag {
    // NSLog(@"sortedDateKeyStrings count in createBezierPathforNutrientWithTag:...: %ld", sortedDateKeyStrings.count);
    FoodItem *foodItemNutrientTotalsForDay;
    
    // Use a temporary object to point to the nutrient graph line path object of interest:
    NSBezierPath *graphLinePath = [NSBezierPath bezierPath];
    [graphLinePath setLineWidth:1.0];
    
    // ----------------------------------------------------------------------
    // Use sortedDateKeyStrings to graph data in Ascending Order of Date:
    // ----------------------------------------------------------------------
    // [graphLinePath moveToPoint:graphOrigin_point];    // Don't start with zero, start with first data point instead (see below).
    // NB: Watch out for arithmetic precision.  Remember, result of int + double is int (loses double precision).  Thus, use double variable type for incrementDay
    double incrementDay = 0.0;
    NSPoint firstPoint;
    NSPoint nextPoint;
    
    for (NSString *aDateKey in _limitedSortedDateKeyStrings) {
        // NB: x and y values are both offset by graphOrigin_point values, so that the bezier paths will get drawn WITHIN the graphs x- and y-axis (and start at the "offset" origin).
        foodItemNutrientTotalsForDay = [nutrientPercentRV_byDate_dictionary_static objectForKey:aDateKey];
        
        // Figure out which nutrient total value to extract, and store it in the y-coordinate value for the graphLinePath:
        nextPoint.y  = [foodItemNutrientTotalsForDay nutrientValue_basedOnTag:dataTag] + _graphOrigin_point.y;
        
        if ([aDateKey isEqualTo:[_limitedSortedDateKeyStrings objectAtIndex:0]]) {
            // Plot the first point:
            firstPoint.x = _graphOrigin_point.x;
            firstPoint.y = nextPoint.y;
            [graphLinePath moveToPoint:firstPoint];
        }
        
        else {
            // Plot next point:
            // Extract nutrient value total for that date key:
            incrementDay = incrementDay + _distanceBetween_X_Values; // space each day by X_AXIS_DISTANCE_BETEEN_TICKS number of pixels
            nextPoint.x  = incrementDay + _graphOrigin_point.x;      // start with day "1"
            [graphLinePath lineToPoint:nextPoint];
        }
    }
    
    return graphLinePath;
}
```
<br></br>


### Get maximum line (bezier path) height for the line chart:
```objective-c
/// Get maximum height of bezierPath.
-(double)bezierPathWithMaxHeight {
    double maxHeight            = 0;
    double bezierPath_y_value   = 0;
    long int bezierPathCount;
    NSPoint currentPoint;
    
    for (int tagNumber = 0; tagNumber <= NUMBER_OF_BUTTON_TAGS; tagNumber++) {
        // Test NSBezierPath graph line based on whether associated nutrient button is on:
        // FB_highlightColor_NSButton *aButton = [[FB_highlightColor_NSButton alloc] init];
        FBColorButton *aButton = [myNutrientButtons_static nutrientButtonbasedOnTag:tagNumber];
        
        // Only check bezierPath nutrient graphs that have associated nutrient buttons with an "ON" state
        if (aButton.state == NSControlStateValueOn) {
            
            NSBezierPath *aBezierPath   = [myNutrientGraphLinePaths_static nutrientGraphLinePath_basedOnTag:tagNumber];
            bezierPathCount             = [aBezierPath elementCount];
            
            // Make sure there's elements (dataPoints) making up the bezierPath:
            if ([aBezierPath elementCount]) {
                // pathRect = [aBezierPath bounds];
                // NSLog(@"\n\n--- check aBezierPath max y-value:");
                
                // Scan for Max y-value of bezierPath, and assign maxHeight to it:
                for (int elementIndex = 0; elementIndex <= bezierPathCount - 1; elementIndex++) {
                    NSBezierPathElement bezierPathElementtype = [aBezierPath elementAtIndex:elementIndex
                                                                           associatedPoints:&currentPoint];
                    
                    // Use NSMoveToBezierPathElement to get NSPoint Data from first element
                    // and, use NSMoveToBezierPathElement to get NSPoint Data from subsequent elements
                    if (bezierPathElementtype == NSLineToBezierPathElement || bezierPathElementtype == NSMoveToBezierPathElement) {
                        // NSLog(@"x,y: %.1f, %.1f", currentPoint.x, currentPoint.y);
                        bezierPath_y_value = currentPoint.y;
                    }
                    if (bezierPath_y_value > maxHeight) { maxHeight = bezierPath_y_value; }
                }
            }
        }
    }
    
    return maxHeight;
}
```
<br></br>


# Sample Screen Shots:
<table>
<tr>
	<td>
	<img src="images/screenshots/iFTDash library.png" alt="Main Window showing library" width="500"/>
	</td>
	<td>
	<img src="images/screenshots/iFTDash favorites.png" alt="Custom Foods" width="500"/>
	</td>
</tr>
<tr>
	<td>
	<img src="images/screenshots/iFTDash favorites nutDetails.png" alt="Main Window showing Nutrient Details" width="500"/>
	</td>
	<td>
	<img src="images/screenshots/iFTDash favorites snacks.png" alt="Custom Foods" width="500"/>
	</td>
</tr>
<tr>
	<td>
	<img src="images/screenshots/iFTDash Custom Food Window.png" alt="Custom Meals" width="500"/>
	</td>
	<td>
	<img src="images/screenshots/iFTDash Custom Meals Window.png" alt="Settings" width="500"/>
	</td>
</tr>
</table>


## I've implemented the following:
#### Code Structure
* Model-View-Controller Design Pattern
* Delegates and Protocols, NSViewController/NSView
* Navigation Split Views
* OutlineViews
* Interface Builder Views
* Cocoa Bindings
* CSV Database Parsing

#### Testing/Error Handling
* Unit Testing
* Error Handling
* Alerts
* Empty States
* Text Input Validation

#### Customization
* Toolbar and Menu selections
* Settings Window with TabBar subviews
* Light and Dark Mode Selections
* Unit Conversions
* Custom Threshold values

#### Project Organization
* Code Documentation (DocC)
* Help Book Manual
* Privacy Manifest
* Group Folder Organization


# Future Considerations
* Sync with HealthKit, for newer macOS versions
* Implement Core Data
* Re-write using Swift/SwiftUI...
<br></br>


# FeedBack
<p class="contact-message">If you have any feedback or suggestions you can reach out via <a class="btn" href="mailto:fbotlogic@fbotlogicsolutions.com?subject=Blue Marble Weather Support">Email</a>.</p>



