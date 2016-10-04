![ThemeKit](https://github.com/luckymarmot/ThemeKit/raw/themekit-initial/ThemeKit.png)
========

![macOS](https://img.shields.io/badge/os-macOS 10.10+-green.svg?style=flat)
![Swift3](https://img.shields.io/badge/swift-3.0.x-green.svg?style=flat)
![macOS](https://img.shields.io/badge/release-1.0-blue.svg?style=flat)
![MIT](https://img.shields.io/badge/license-MIT-lightgray.svg)
![CocoaPods](https://img.shields.io/badge/dep-CocoaPods-orange.svg)
![Carthage](https://img.shields.io/badge/dep-Carthage-orange.svg)

## Summary

*ThemeKit* is a lightweight theming library completly written in Swift 3 that provides theming capabilities to both Swift 3 and Objective-C macOS applications.

*ThemeKit* is brought to you by the [Paw](https://paw.cloud) team.

## Table of Contents
* [Summary](#summary)
* [Features](#features)
* [Installation](#installation)
* [Usage](#usage)
  * [Simple Usage](#simple-usage)
  * [Advanced Usage](#advanced-usage)
     * [Observing theme changes](#observing-theme-changes)
     * [Manually theming windows](#manually-theming-windows)
     * [NSWindow extension](#nswindow-extension)
* [Theme-aware Assets](#theme-aware-assets)
* [Creating Themes](#creating-themes)
  * [Native Themes](#native-themes)
  * [User Themes](#user-themes)
* [FAQ](#faq)
* [License](#license)

## Features

- Written in Swift 3
- Optional configuration, none required
- Neglected performance impact
- Themes:
  - [`LightTheme`](https://paw.cloud/opensource/themekit/docs/Classes/LightTheme.html) (default macOS appearance)
  - [`DarkTheme`](https://paw.cloud/opensource/themekit/docs/Classes/DarkTheme.html)
  - [`SystemTheme`](https://paw.cloud/opensource/themekit/docs/Classes/SystemTheme.html) (default theme). Dynamically resolves to `ThemeKit.lightTheme` or `ThemeKit.darkTheme`, depending on the *"System Preferences > General > Appearance"*.
  - Support for custom themes ([`Theme`](https://paw.cloud/opensource/themekit/docs/Classes/Theme.html))
  - Support for user-defined themes ([`UserTheme`](https://paw.cloud/opensource/themekit/docs/Classes/UserTheme.html))
- Theme-aware assets:
  - [`ThemeColor`](https://paw.cloud/opensource/themekit/docs/Classes/ThemeColor.html): colors that dynamically change with the theme
  - [`ThemeGradient`](https://paw.cloud/opensource/themekit/docs/Classes/ThemeGradient.html): gradients that dynamically change with the theme
  - [`ThemeImage`](https://paw.cloud/opensource/themekit/docs/Classes/ThemeImage.html): images that dynamically change with the theme
  - Optional override of `NSColor` named colors (e.g., `labelColor`) to dynamically change with the theme

## Installation
There are multiple options to inlcude *ThemeKit* on your project:

- [CocoaPods](https://cocoapods.org): add to your `Podfile`:

  ```
  use_frameworks!
  pod 'ThemeKit', '~> 1.0'
  ```
  
- [Carthage](https://github.com/Carthage/Carthage):

  ```
  github "luckymarmot/ThemeKit"
  ```
  
- Manually:
  - Either add `ThemeKit.framework` on your project
  - Or, manually add source files from the `ThemeKit\` folder to your project

## Usage

### Simple Usage
At its simpler usage, applications can be themed with a single line command:

```swift
func applicationWillFinishLaunching(_ notification: Notification) {
	
	/// Apply the dark theme
	ThemeKit.darkTheme.apply()
	
	/// or, the light theme
	//ThemeKit.lightTheme.apply()
	
	/// or, the 'system' theme, which dynamically changes to light or dark, 
	/// respecting *System Preferences > General > Appearance* setting.
	//ThemeKit.systemTheme.apply()
	
}
```

### Advanced Usage

The following code will define which windows should be automatically themed ([`WindowThemePolicy`](https://paw.cloud/opensource/themekit/docs/Classes/ThemeKit/WindowThemePolicy.html)) and add support for user themes ([`UserTheme`](https://paw.cloud/opensource/themekit/docs/Classes/UserTheme.html)):

```swift
func applicationWillFinishLaunching(_ notification: Notification) {

	/// Define default theme.
	/// Used on first run. Default: `SystemTheme`.
	/// Note: `SystemTheme` is a special theme that resolves to `ThemeKit.lightTheme` or `ThemeKit.darkTheme`,
	/// depending on the macOS preference at 'System Preferences > General > Appearance'.
	ThemeKit.defaultTheme = ThemeKit.lightTheme
	
	/// Define window theme policy.
	ThemeKit.shared.windowThemePolicy = .themeAllWindows
	//ThemeKit.shared.windowThemePolicy = .themeSomeWindows(windowClasses: [MyWindow.self])
	//ThemeKit.shared.windowThemePolicy = .doNotThemeWindows
	    
	/// Enable & configure user themes.
	/// Will use folder `(...)/Application Support/{your_app_bundle_id}/Themes`.
	let applicationSupportURLs = NSSearchPathForDirectoriesInDomains(.applicationSupportDirectory, .userDomainMask, true)
	let thisAppSupportURL = URL.init(fileURLWithPath: applicationSupportURLs.first!).appendingPathComponent(Bundle.main.bundleIdentifier!)
	let userThemesFolderURL = thisAppSupportURL.appendingPathComponent("Themes")
	ThemeKit.shared.userThemesFolderURL = userThemesFolderURL
	
	/// Change the default light and dark theme, used when `SystemTheme` is selected.
	//ThemeKit.lightTheme = ThemeKit.shared.theme(withIdentifier: PaperTheme.identifier)!
	//ThemeKit.darkTheme = ThemeKit.shared.theme(withIdentifier: "com.luckymarmot.ThemeKit.PurpleGreen")!
	
	/// NOTE: You don't need to manually select a theme.
	///       Theme selected from last app run is stored on `NSUSerDefaults`
	///       and is automatically applied at startup.
	 
}    
```

#### Observing theme changes

ThemeKit provides the following notifications:

- `Notification.Name.willChangeTheme` is sent when current theme is about to change
- `Notification.Name.didChangeTheme` is sent when current theme did change
- `Notification.Name.didChangeSystemTheme` is sent when system theme did change (System Preference > General)

Example:

```swift
// Register to be notified of theme changes
NotificationCenter.default.addObserver(self, selector: #selector(changedTheme(_:)), name: .didChangeTheme, object: nil)

@objc private func changedTheme(_ notification: Notification) {
	// ...
}
```

Additionaly, the following properties are KVO compliant:

- [`ThemeKit.shared.theme`](https://paw.cloud/opensource/themekit/docs/Classes/ThemeKit.html#/s:vC8ThemeKit8ThemeKit5themePS_5Theme_)
- [`ThemeKit.shared.effectiveTheme`](https://paw.cloud/opensource/themekit/docs/Classes/ThemeKit.html#/s:vC8ThemeKit8ThemeKit14effectiveThemePS_5Theme_)
- [`ThemeKit.shared.themes`](https://paw.cloud/opensource/themekit/docs/Classes/ThemeKit.html#/s:vC8ThemeKit8ThemeKit6themesGSaPS_5Theme__)
- [`ThemeKit.shared.userThemes`](https://paw.cloud/opensource/themekit/docs/Classes/ThemeKit.html#/s:vC8ThemeKit8ThemeKit10userThemesGSaPS_5Theme__)

Example:

```swift
// Register for KVO changes on ThemeKit.shared.effectiveTheme
ThemeKit.shared.addObserver(self, forKeyPath: "effectiveTheme", options: NSKeyValueObservingOptions.init(rawValue: 0), context: nil)

public override func observeValue(forKeyPath keyPath: String?, of object: Any?, change: [NSKeyValueChangeKey : Any]?, context: UnsafeMutableRawPointer?) {
	if keyPath == "effectiveTheme" {
		// ...
   }
}
```


#### Manually theming windows

In case ([`WindowThemePolicy`](https://paw.cloud/opensource/themekit/docs/Classes/ThemeKit/WindowThemePolicy.html)) was NOT set to `.themeAllWindows`, you may need to manually theme a window. You can use our `NSWindow` extension for that:

##### NSWindow Extension

- `NSWindow.theme()`

	Theme window if appearance needs update. Doesn't check for policy compliance.


- `NSWindow.isCompliantWithWindowThemePolicy()`

	Check if window complies to current policy.

- `NSWindow.themeIfCompliantWithWindowThemePolicy()`

	Theme window if compliant to `ThemeKit.shared.windowThemePolicy` (and if appearance needs update).

- `NSWindow.themeAllWindows()`

	Theme all windows compliant to `ThemeKit.shared.windowThemePolicy` (and if appearance needs update).


## Theme-aware Assets

[`ThemeColor`](https://paw.cloud/opensource/themekit/docs/Classes/ThemeColor.html), [`ThemeGradient`](https://paw.cloud/opensource/themekit/docs/Classes/ThemeGradient.html) and [`ThemeImage`](https://paw.cloud/opensource/themekit/docs/Classes/ThemeImage.html) provides colors, gradients and images, respectively, that dynamically change with the current theme.
Additionally, named colors from the `NSColor` class defined on the `ThemeColor` subclass extension will override the system ones, providing theme-aware colors.

For example, a project defines a `ThemeColor.brandColor` color. This will resolve to different colors at runtime, depending on the selected theme:

- `ThemeColor.brandColor` will resolve to `NSColor.blue` if the light theme is selected
- `ThemeColor.brandColor` will resolve to `NSColor.white` if the dark theme is selected
- `ThemeColor.brandColor` will resolve to `rgba(100, 50, 0, 0.5)` for some user-defined theme ([`UserTheme`](https://paw.cloud/opensource/themekit/docs/Classes/UserTheme.html))

Similarly, defining a `ThemeColor.labelColor` will override `NSColor.labelColor` (`ThemeColor` is a subclass of `NSColor`), and *ThemeKit* will allow `labelColor` to be customized on a per-theme basis as well.

### Fallback Assets

ThemeKit provides a simple fallback mechanism when looking up assets in current theme. It will search for assets, in order:

- the asset name, defined in theme (e.g., `myBackgroundColor`)
- `fallbackForegroundColor`, `fallbackBackgroundColor`, `fallbackGradient` or `fallbackImage` defined in theme, depending if asset is a foreground/background color, gradient or image, respectively
- `defaultFallbackForegroundColor`, `defaultFallbackBackgroundColor`, `fallbackGradient` or `defaultFallbackImage` defined internally, depending if asset is a foreground/background color, gradient or image, respectively

However, for overrided system named colors, the fallback mechanism is different and simpler:

- the asset name, defined in theme (e.g., `labelColor`)
- original asset defined in system (e.g., `NSColor.labelColor`)

Please refer to [`ThemeColor`](https://paw.cloud/opensource/themekit/docs/Classes/ThemeColor.html), [`ThemeGradient`](https://paw.cloud/opensource/themekit/docs/Classes/ThemeGradient.html) and [`ThemeImage`](https://paw.cloud/opensource/themekit/docs/Classes/ThemeImage.html) for more information.

## Creating Themes

### Native Themes
For creating additional themes, you only need to create a class that conforms to the [`Theme`](https://paw.cloud/opensource/themekit/docs/Classes/Theme.html) protocol and extends `NSObject`.

Sample theme:

```
import Cocoa
import ThemeKit
	
class MyOwnTheme: NSObject, Theme {
    
    /// Light theme identifier (static).
    public static var identifier: String = "com.luckymarmot.ThemeKit.MyOwnTheme"
    
    /// Unique theme identifier.
    public var identifier: String = MyOwnTheme.identifier
    
    /// Theme display name.
    public var displayName: String = "My Own Theme"
    
    /// Theme short display name.
    public var shortDisplayName: String = "My Own"
    
    /// Is this a dark theme?
    public var isDarkTheme: Bool = false
    
    /// Description (optional).
    public override var description : String {
        return "<\(MyOwnTheme.self): \(themeDescription(self))>"
    }
    
    // MARK: -
    // MARK: Theme Assets
    
    // Here you can define the instance methods for the class methods defined 
    // on `ThemeColor`, `ThemeGradient` and `ThemeImage`, if any. Check
    // documentation of these classes for more details.
}
```

### User Themes
ThemeKit also supports definition of additional themes with simple text files (`.theme` files). Example of a very basic `.theme` file:

```ruby
// ************************* Theme Info ************************* //
displayName = My Theme 1
identifier = com.luckymarmot.ThemeKit.MyTheme1
darkTheme = true

// ********************* Colors & Gradients ********************* //
# define color for `ThemeColor.brandColor`
brandColor = $blue
# define a new color for `NSColor.labelColor` (overriding)
labelColor = rgb(11, 220, 111)
# define gradient for `ThemeGradient.brandGradient`
brandGradient = linear-gradient($orange.sky, rgba(200, 140, 60, 1.0))
 
// ********************* Images & Patterns ********************** //
# define pattern image from named image "paper" for color `ThemeColor.contentBackgroundColor`
contentBackgroundColor = pattern(named:paper)
# define pattern image from filesystem (relative to user themes folder) for color `ThemeColor.bottomBackgroundColor`
bottomBackgroundColor = pattern(file:../some/path/some-file.png)
# define image using named image "apple"
namedImage = image(named:apple)
# define image using from filesystem (relative to user themes folder)
fileImage = image(file:../some/path/some-file.jpg)

// *********************** Common Colors ************************ //
blue = rgb(0, 170, 255)
orange.sky = rgb(160, 90, 45, .5)

// ********************** Fallback Assets *********************** //
fallbackForegroundColor = rgb(255, 10, 90, 1.0)
fallbackBackgroundColor = rgb(255, 200, 190)
fallbackGradient = linear-gradient($blue, rgba(200, 140, 60, 1.0))

```

To enable support for user themes, just need to set the location for them:

```
// Setup ThemeKit user themes folder
ThemeKit.shared.userThemesFolderURL = //...
```

Please refer to [`UserTheme`](https://paw.cloud/opensource/themekit/docs/Classes/UserTheme.html) for more information.


## FAQ

### Can controls be tinted with different colors?
Other than the colors set by the inherited appearance - light (dark text on light background) or dark (light text on dark background) - natively, it is not possible to specify different colors for the text and/or background fills of controls (buttons, popups, etc).

For simple cases, overriding `NSColor` can be sufficient: for example, `NSColor.labelColor` is a named color used for text labels; overriding it will allow to have all labels themed accordingly. You can get a list of all overridable named colors (class method names) with `NSColor.colorMethodNames()`.

For more complex cases, like views/controls with custom drawing, please refer to next question.

### Can I make custom drawing views/controls theme-aware?
Yes, you can! Implement your own custom controls drawing using [Theme-aware Assets](#theme-aware-assets) (`ThemeColor` and `ThemeGradient`) so that your controls drawing will always adapt to your current theme... automatically!

In case needed (for example, if drawing is being cached), you can observe when theme changes to refresh the UI or to perform any theme related operation. Check *"Observing theme changes"* on [Usage](#usage) section above.

### Are images theme-aware as well?
Partially: currently, only pattern images (defined as `NSColor(patternImage:)`) are supported. We hope to add support for theme-aware `NSImage`'s in the future.


### I'm having font smoothing issues!
You may run into font smoothing issues, when you use text without a background. Bottom line is, always specify/draw a background when using/drawing text. 

  1. For controls like `NSTextField`, `NSTextView`, etc:
   
    Specify a background color on the control. E.g.,
    
    ```
    control.backgroundColor = NSColor.black
    ```
    
  2. For custom text rendering:

    First draw a background fill, then enable font smoothing and render your text. E.g.,
    
    ```
    let context = NSGraphicsContext.current()?.cgContext
    NSColor.black.set()
    context?.fill(frame)
    context?.saveGState()
    context?.setShouldSmoothFonts(true)
        
    // draw text...
    
	context?.restoreGState()
    ``` 
    As a last solution - if you really can't draw a background color - you can disable font smoothing which can slightly improve text rendering:
    
    ```
    let context = NSGraphicsContext.current()?.cgContext
    context?.saveGState()
    context?.setShouldSmoothFonts(false)
        
    // draw text...
    
	context?.restoreGState()
   ```
   
  3. For custom `NSButton`'s:

    This is more tricky, as you will need to override private methods. If you are distributing your app on the Mac App Store, you must first check if this is allowed.
    
    a) override the private method `_backgroundColorForFontSmoothing` to return your button background color.
    
    b) if (a) isn't sufficient, you will also need to override `_textAttributes` and change the dictionary returned from the `super` call to provide your background color for the key `NSBackgroundColorAttributeName`.

## License

*ThemeKit* is available under the MIT license. See the [LICENSE](LICENSE) file for more info.