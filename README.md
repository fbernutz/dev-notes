# Felis dev notes ✌️

My personal list of some things I learned in ...

* iOS, Swift
* CocoaPods
* git
* Android, Kotlin

... and do not want to forget. Some things might be outdated.

- [iOS](#ios)
  - [#24: Deselect selected Row](#24-deselect-selected-row)
  - [#23: Using specific versions for CocoaPods](#23-using-specific-versions-for-cocoapods)
  - [#22: Clean StatusBar](#22-clean-statusbar)
  - [#21: Find unused resources](#21-find-unused-resources)
  - [#20: Speed up WWDC Videos](#20-speed-up-wwdc-videos)
  - [#19: Converting between String and Data](#19-converting-between-string-and-data)
  - [#18: Localizing for plurals and variant widths using a stringsdict file](#18-localizing-for-plurals-and-variant-widths-using-a-stringsdict-file)
  - [#17: Target Environment Simulator](#17-target-environment-simulator)
  - [#16: Delay requests while searching using GCD](#16-delay-requests-while-searching-using-gcd)
  - [#15: WebViewController](#15-webviewcontroller)
  - [#14: Common Mistakes with Custom Fonts](#14-common-mistakes-with-custom-fonts)
  - [#13: IBOutletCollection](#13-iboutletcollection)
  - [#12: `debugPrint` in release environment](#12-debugprint-in-release-environment)
  - [#11: Check for email addresses in String](#11-check-for-email-addresses-in-string)
  - [#10: StackView & Warnings](#10-stackview--warnings)
  - [#09: Find error in plist](#09-find-error-in-plist)
  - [#08: Right and LeftView in UITextField](#08-right-and-leftview-in-uitextfield)
  - [#07: Dispatch Group](#07-dispatch-group)
  - [#06: ATS information](#06-ats-information)
  - [#05: Attributed Text Helper](#05-attributed-text-helper)
  - [#04: UITest helper](#04-uitest-helper)
  - [#03: Configure UIKit elements on initializer](#03-configure-uikit-elements-on-initializer)
  - [#02: Simple animation with UIImage](#02-simple-animation-with-uiimage)
  - [#01: UIImage with black & white effect](#01-uiimage-with-black--white-effect)
- [Xcode](#xcode)
  - [#08: Support new iOS in old Xcode](#08-support-new-ios-in-old-xcode)
  - [#07: Clear Cache for Launch Screen](#07-clear-cache-for-launch-screen)
  - [#06: Open Deep Link in Simulator](#06-open-deep-link-in-simulator)
  - [#05: Select next occurrence](#05-select-next-occurrence)
  - [#04: Adding identifier to constraints](#04-adding-identifier-to-constraints)
  - [#03: Weird problems with IBDesignable](#03-weird-problems-with-ibdesignable)
  - [#02: Set version number without fastlane](#02-set-version-number-without-fastlane)
  - [#01: `po` in debugger](#01-po-in-debugger)
- [git](#git)
  - [#05: Trunk based development and feature toggles](#05-trunk-based-development-and-feature-toggles)
  - [#04: Rebase](#04-rebase)
  - [#03: Squash](#03-squash)
  - [#02: Useful git commands](#02-useful-git-commands)
  - [#01: Ignore already tracked `XCUserstate`](#01-ignore-already-tracked-xcuserstate)
- [HTML](#html)
  - [#01: `<details>` and `<summary>`](#01-details-and-summary)
- [Android](#android)
  - [#03: Logging](#03-logging)
  - [#02: Integrate `ktlint` into project](#02-integrate-ktlint-into-project)
  - [#01: Swift is like Kotlin](#01-swift-is-like-kotlin)

## iOS

### #24: Deselect selected Row

How to deselect the selected row/item so it happens alongside the navigation back animation (including when using the interactive pop gesture):

```swift
extension UITableView {
    func deselectRow(along transitionCoordinator: UIViewControllerTransitionCoordinator?) {
        guard let selectedIndexPath = indexPathForSelectedRow else { return }

        guard let coordinator = transitionCoordinator else {
            deselectRow(at: selectedIndexPath, animated: false)
            return
        }

        coordinator.animate(alongsideTransition: { _ in
            self.deselectRow(at: selectedIndexPath, animated: true)
        }) { (context) in
            if context.isCancelled {
                self.selectRow(at: selectedIndexPath, animated: false, scrollPosition: .none)
            }
        }
    }
}
```

Found [in twitter by @smileyborg](https://twitter.com/smileyborg/status/1279473615553982464).

### #23: Using specific versions for CocoaPods

Have a look at: [http://guides.cocoapods.org/using/the-podfile.html](http://guides.cocoapods.org/using/the-podfile.html).

* `~> 0.1.2` will get you a version up to 0.2 (but not including 0.2 and higher)
* `~> 0.1` will get you a version up to 1.0 (but not including 1.0 and higher)
* `~> 0` will get you a version of 0 and higher (same as if it was omitted)

### #22: Clean StatusBar

See [here](https://www.jessesquires.com/blog/overriding-status-bar-settings-ios-simulator/).

Run:

```bash
// Xcode 11: Use Terminal to set the Status Bar of all booted Simulator devices to the most common variables.
xcrun simctl status_bar booted override --time "9:41" --batteryState charged --batteryLevel 100 --cellularMode active
```

### #21: Find unused resources

See [here](https://www.avanderlee.com/optimization/unused-images-clean-up/).

Use `Fengniao` [GitHub Repo](https://github.com/onevcat/FengNiao) to find unused resources in Xcode. Install it and exclude the folder you want to skip: `fengniao --exclude Pods fastlane SnapshotTests DerivedData` 

### #20: Speed up WWDC Videos

If you want to increase the video play rate of the sessions of WWDC, open Safari Dev Tools and type the following into the console. You'll see the video in 140% speed.

```javascript
document.getElementsByTagName('video')[0].playbackRate = 1.4;
```

### #19: Converting between String and Data

Idea from: [https://twitter.com/johnsundell/status/1117812175593197568](https://twitter.com/johnsundell/status/1117812175593197568)

Without optionals:

```swift
// String -> Data
let data = Data("String".utf8)

// Data -> String
let string = String(decoding: data, as: UTF8.self)
```

With optionals:

```swift
// String -> Data?
let data = "String".data(using: .utf8)

// Data -> String?
let optionalString = String(data: data, encoding: .utf8) 
```

### #18: Localizing for plurals and variant widths using a stringsdict file

You can use a stringsdict file to add plural support for your strings or to add different strings for variant widths, e.g. a short expression, like "Hi" for small devices and a longer one, like "Welcome <Name>" for larger devices. Using the stringsdict file removes the extra logic for plural or variant widths support from your code to an easy to read xml file.

#### Plurals example

In practice, you define a key in your `Localizable.stringsdict` and define the values for your plurals. Xcode will automatically define the different keys for the specific language. For German there is `zero`, `one` and `other` but for Russian, there are more plural options. Xcode will automatically generate these keys, so you just have to add the correct values for the different strings.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
    <dict>
        <key>%d movie(s)</key>
        <dict>
            <key>NSStringLocalizedFormatKey</key>
            <string>%#@movies@</string>
            <key>movies</key>
            <dict>
                <key>NSStringFormatSpecTypeKey</key>
                <string>NSStringPluralRuleType</string>
                <key>NSStringFormatValueTypeKey</key>
                <string>d</string>
                <key>zero</key>
                <string>Keine Filme</string>
                <key>one</key>
                <string>Ein Film</string>
                <key>other</key>
                <string>%d Filme</string>
            </dict>
        </dict>
    </dict>
</plist>
```

In your project you can define your string like that:

```swift
static func movies(for counter: Int) -> String {
        return String.localizedStringWithFormat(NSLocalizedString("%d movie(s)", comment: "Movie(s)"), counter)
}
```

_Important: Your key from `NSLocalizedString` has to be the same as in your `Localizable.stringsdict`._

#### Variant width example

To specify different strings for variant widths will sometimes solve some layout issues and this is a better solution than to always use a short string, because there would be much white space on larger devices.

Your `Localizable.stringsdict` can look like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
    <dict>
        <key>welcome.width</key>
        <dict>
            <key>NSStringVariableWidthRuleType</key>
            <dict>
                <key>1</key>
                <string>Hi</string>
                <key>30</key>
                <string>Herzlich Willkommen</string>
            </dict>
        </dict>
    </dict>
</plist>
```

Like in the plurals example, it is important to use the same key in your swift declaration.

```swift
static let welcome = NSLocalizedString("welcome.width", comment: "Title for welcome")
```

There is one thing I don't understand with this. You set a value for the key and the compiler will automatically select the value for UIKit elements, but this did not work like I expected.
With `NSString.variantFittingPresentationWidth(_ width: Int) -> String` you can set the width directly and the value will be returned for the appropriated width. If you call the method with 30 or bigger `Herzlich Willkommen` will be returned and if you call it with 1..<30 `Hi` will be returned. But you need to define when which string will be used. I wrote a helper as extension for String to decide which string will be returned depending on the screen device.

```swift
import UIKit

extension String {
    var forWidth: String {
        let width =
            Int(UIScreen.main.bounds.width) > 320
                ? 30
                : 1
        return (self as NSString).variantFittingPresentationWidth(width)
    }
}
```

In this case only for iPhone 5 or 5s the smaller string will be used. If you know any better solution, please let me know! 😉

You call this helper every time the string is used, so the correct string will be returned from the `Localizable.stringsdict`.

```swift
static let welcome = NSLocalizedString("welcome.width", comment: "Title for welcome").forWidth
```

_Info:
The `variantFittingPresentationWidth` won't work for some UI elements, like `UITabBarItem` and `UIPreviewAction`. So in these cases the correct string will be returned when you print them, but they are not displayed. This felt like an unexpected behaviour for me, so I filed my first bug reports: [rdar://41193537](https://openradar.appspot.com/41193537) and [rdar://41193359](https://openradar.appspot.com/41193359)_

### #17: Target Environment Simulator

Have a look at: [https://swift.org/blog/swift-4-1-released](https://swift.org/blog/swift-4-1-released/)

You can use `#if targetEnvironment(simulator)` to check if your app runs in a simulator. This is really useful, when you want to add some special handling for simulators, like taking a photo with a real device and using a dummy image in simulator.

```swift
import UIKit
class TestViewController: UIViewController, UIImagePickerControllerDelegate {
   // a method that does some sort of image processing
   func processPhoto(_ img: UIImage) {
       // process photo here
   }
   // a method that loads a photo either using the camera or using a sample
   func takePhoto() {
      #if targetEnvironment(simulator)
         // we're building for the simulator; use the sample photo
         if let img = UIImage(named: "sample") {
            processPhoto(img)
         } else {
            fatalError("Sample image failed to load")
         }
      #else
         // we're building for a real device; take an actual photo
         let picker = UIImagePickerController()
         picker.sourceType = .camera
         vc.allowsEditing = true
         picker.delegate = self
         present(picker, animated: true)
      #endif
   }
   // this is called if the photo was taken successfully
   func imagePickerController(_ picker: UIImagePickerController, didFinishPickingMediaWithInfo info: [String : Any]) {
      // hide the camera
      picker.dismiss(animated: true)
      // attempt to retrieve the photo they took
      guard let image = info[UIImagePickerControllerEditedImage] as? UIImage else {
         // that failed; bail out
         return
      }
      // we have an image, so we can process it
      processPhoto(image)
   }
}
```

### #16: Delay requests while searching using GCD

Have a look at: [https://www.swiftbysundell.com/posts/a-deep-dive-into-grand-central-dispatch-in-swift](https://www.swiftbysundell.com/posts/a-deep-dive-into-grand-central-dispatch-in-swift).

How to use GCD to delay sending requests while searching.

```swift
class SearchViewController: UIViewController, UISearchBarDelegate {
    // We keep track of the pending work item as a property
    private var pendingRequestWorkItem: DispatchWorkItem?

    func searchBar(_ searchBar: UISearchBar, textDidChange searchText: String) {
        // Cancel the currently pending item
        pendingRequestWorkItem?.cancel()

        // Wrap our request in a work item
        let requestWorkItem = DispatchWorkItem { [weak self] in
            self?.resultsLoader.loadResults(forQuery: searchText)
        }

        // Save the new work item and execute it after 250 ms
        pendingRequestWorkItem = requestWorkItem
        DispatchQueue.main.asyncAfter(deadline: .now() + .milliseconds(250), execute: requestWorkItem)
    }
}
```

### #15: WebViewController

iOS 8 and lower: Use `WKWebView`

iOS 9 or newer: Use `SFSafariViewController`

```swift
import SafariServices

guard let url = URL(string: "") else { return }
let safariController = SFSafariViewController(url: url)
present(safariController, animated: true, completion: nil)
```

Websites can be...

* shared (+)
* opened in safari (+)
* displayed in reader mode directly (+)

Websites will be...

* displayed in a different design. It will feel like something outside your app (-)

### #14: Common Mistakes with Custom Fonts

Have a look at: [https://codewithchris.com/common-mistakes-with-adding-custom-fonts-to-your-ios-app/](https://codewithchris.com/common-mistakes-with-adding-custom-fonts-to-your-ios-app/)

### #13: IBOutletCollection

Have a look at: [https://medium.com/@abhimuralidharan/what-is-an-iboutletcollection-in-ios-78cfbc4080a1](https://medium.com/@abhimuralidharan/what-is-an-iboutletcollection-in-ios-78cfbc4080a1).

`IBOutletCollection` can be used to configure an array of outlets.

```swift
@IBOutlet var buttons: [UIButton]! {
    didSet {
        for button in buttons {
            button.backgroundColor = .black
        }
    }
}
```

### #12: `debugPrint` in release environment

Have a look at: [http://swiftiostutorials.com/use-nslog-debugging/](http://swiftiostutorials.com/use-nslog-debugging/)

`debugPrint` has nothing to do with debug and release environments. It just gives you some information more than the normal `print`.

To solve this, you can overwrite the `print` function with the following code and even add a `releasePrint` to print in release environment:

```swift

func releasePrint(_ object: Any) {
    Swift.print(object)
}

func print(_ object: Any) {
    #if DEBUG
    Swift.print(object)
    #endif
}
```

To use the `DEBUG` flag in code, you have to set the custom flag "DEBUG" in Build Settings > Active Compilation Conditions > Debug.

### #11: Check for email addresses in String

Have a look at: [http://developer.bombbomb.com/blog/2017/10/24/swift-data-detection/](http://developer.bombbomb.com/blog/2017/10/24/swift-data-detection/).

```swift
import UIKit

extension String {
    // Get email addresses in a string, discard any other content.
    func emailAddresses() -> [String] {
        var addresses = [String]()

        do {
            let detector = try NSDataDetector(types: NSTextCheckingResult.CheckingType.link.rawValue)
            let matches = detector.matches(in: self, options: [], range: NSMakeRange(0, self.count))
            for match in matches {
                if let matchURL = match.url,
                    let matchURLComponents = URLComponents(url: matchURL, resolvingAgainstBaseURL: false), matchURLComponents.scheme == "mailto" {
                    let address = matchURLComponents.path
                    addresses.append(String(address))
                }
            }
        } catch {
            return []
        }

        return addresses
    }
}

let string = "ios@test.com is an email address"
string.emailAddresses() //results in "ios@test.com"
```

### #10: StackView & Warnings

Have a look at: [http://stackoverflow.com/a/37447714/6716961](http://stackoverflow.com/a/37447714/6716961)

`Do not make layer changes in UIStackView!`

### #09: Find error in plist

`plutil <PATH_TO_FILE>`

### #08: Right and LeftView in UITextField

You can use a `rightView` or/and `leftView` in `UITextField`.

#### When is it useful?

Well, it depends.

When you want to use a custom element as "side element" of the text field and want to define it in storyboard, you just do not have to use the `rightView` parameter. This is really useful when you have the view configured in code and then you can simply add it to the text field and you do not have to set constraints etc.

```swift
@IBOutlet var customView: UIView!

func textFieldDidBeginEditing(_ textField: UITextField) {
    if textField == password {
        password.rightView = customView
        password.rightViewMode = .always
    }
}
```

#### References

* [ Developer — Manage TextFieldTextViews](https://developer.apple.com/library/content/documentation/StringsTextFonts/Conceptual/TextAndWebiPhoneOS/ManageTextFieldTextViews/ManageTextFieldTextViews.html)
* [ Developer — RightView](https://developer.apple.com/reference/uikit/uitextfield/1619596-rightview)
* [ Developer — RigthViewMode](https://developer.apple.com/reference/uikit/uitextfield/1619607-rightviewmode)

### #07: Dispatch Group

* Create a group
* Enter a group
* Won't finish until every group has left
* Will notify on a specified queue when finished

Very useful with arrays of asynchronous requests and their completion handler.

```swift
let dispatchGroup = DispatchGroup()
var items = [Item]()
for item in items {
    if item.name == "name" {
        dispatchGroup.enter()
        doSomething(item.name) { _ in
            //do something
            dispatchGroup.leave()
        }
    }
}

dispatchGroup.notify(queue: DispatchQueue.main) {
    //do something
}
```

### #06: ATS information

There are the following criteria, for which you have to add an exception:

* http
* lower TLS version than 1.2
* cipher suites that do not support forward secrecy

#### Define the problem

* Run in command line `nscurl --ats-diagnostics <url> --verbose`
* Check url on the [ssltest](https://www.ssllabs.com/ssltest/) site from ssllabs.
* You can test your [browsers capabilities](https://www.ssllabs.com/ssltest/viewMyClient.html) in ssllabs too.

#### Use exception

Available keys:

* `NSIncludesSubdomains`
* `NSExceptionAllowsInsecureHTTPLoads`
* `NSExceptionRequiresForwardSecrecy`
* `NSExceptionMinimumTLSVersion`
* `NSThirdPartyExceptionAllowsInsecureHTTPLoads`
* `NSThirdPartyExceptionMinimumTLSVersion`
* `NSThirdPartyExceptionRequiresForwardSecrecy`

##### add in `info.plist`

```xml
<key>NSAppTransportSecurity</key>
<dict>
<key>NSExceptionDomains</key>
    <dict>
        <key>url.com</key>
        <dict>
            <key>NSThirdPartyExceptionMinimumTLSVersion</key>
            <string>TLSv1.0</string>
            <key>NSThirdPartyExceptionRequiresForwardSecrecy</key>
            <false/>
            <key>NSTemporaryExceptionAllowsInsecureHTTPLoads</key>
            <true/>
            <key>NSIncludesSubdomains</key>
            <true/>
        </dict>
    </dict>
</dict>
```

#### References

Getting Ready for ATS Enforcement in 2017

* [https://developer.apple.com/library/ios/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW33](https://developer.apple.com/library/ios/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW33)
* [https://ste.vn/2015/06/10/configuring-app-transport-security-ios-9-osx-10-11/](https://ste.vn/2015/06/10/configuring-app-transport-security-ios-9-osx-10-11/)
* [http://stackoverflow.com/questions/30720813/cfnetwork-sslhandshake-failed-ios-9](http://stackoverflow.com/questions/30720813/cfnetwork-sslhandshake-failed-ios-9)
* [http://www.heise.de/security/artikel/Zukunftssicher-Verschluesseln-mit-Perfect-Forward-Secrecy-1923800.html](http://www.heise.de/security/artikel/Zukunftssicher-Verschluesseln-mit-Perfect-Forward-Secrecy-1923800.html)
* [http://timekl.com/blog/2015/08/21/shipping-an-app-with-app-transport-security/](http://timekl.com/blog/2015/08/21/shipping-an-app-with-app-transport-security/)
* [http://www.neglectedpotential.com/2015/06/working-with-apples-application-transport-security/](http://www.neglectedpotential.com/2015/06/working-with-apples-application-transport-security/)

### #05: Attributed Text Helper

#### non-breaking

* non breaking space: `\u00a0`
* non breaking hyphen: `\u2011`

To use this in `Localizable.strings`, you have to use all caps (like `\U00A0`). Found [here](https://stackoverflow.com/questions/5105053/iphone-uilabel-non-breaking-space#comment24881735_15696729).

#### soft hyphen

Have a look at: [http://stackoverflow.com/questions/33727659/how-to-perform-hyphenation-on-a-uilabel-sensitive-to-a-language-different-from-t](http://stackoverflow.com/questions/33727659/how-to-perform-hyphenation-on-a-uilabel-sensitive-to-a-language-different-from-t)

```swift
let title = "Title with Hyphen"

let paragraphStyle = NSMutableParagraphStyle()
paragraphStyle.hyphenationFactor = 1.0

let paragraphAttributes = [NSAttributedStringKey.paragraphStyle: paragraphStyle]
let attributedStringWithHyphens = NSAttributedString(
    string: title,
    attributes: paragraphAttributes
)

let label = UILabel()
label.attributedText = attributedStringWithHyphens
```

### #04: UITest helper

* `print(app.debugDescription)`
* Testing AutoLayout: Edit Scheme -> Options -> Double Length Pseudolanguage
* use internal functions in test classes with `@testable import ProjectName`

### #03: Configure UIKit elements on initializer

```swift
lazy var resultSearchController: UISearchController  = {
	let resultSearchController = UISearchController(searchResultsController: nil)
	resultSearchController.dimsBackgroundDuringPresentation = false
	resultSearchController.isActive = false
	resultSearchController.searchBar.sizeToFit()
	resultSearchController.searchResultsUpdater = self
	return resultSearchController
}()
```

### #02: Simple animation with UIImage

```swift
@IBOutlet weak var loadingScene: UIImageView!

loadingScene.animationImages = [
    UIImage(named: "01")!,
    UIImage(named: "02")!,
    UIImage(named: "03")!,
    UIImage(named: "04")!,
    UIImage(named: "06")!
]

loadingScene.animationDuration = 1
loadingScene.startAnimating()
```

### #01: UIImage with black & white effect

Have a look at: [ Developer - CoreImageFilterReference](https://developer.apple.com/library/content/documentation/GraphicsImaging/Reference/CoreImageFilterReference/#//apple_ref/doc/filter/ci/CIPhotoEffectMono).

```swift
func convertToBlackAndWhite() -> UIImage {
	let filter = CIFilter(name: "CIPhotoEffectMono")!
	filter.setDefaults()
	filter.setValue(CoreImage.CIImage(image: self)!, forKey: kCIInputImageKey)
	return UIImage(CGImage: CIContext(options:nil).createCGImage(filter.outputImage!, fromRect: filter.outputImage!.extent))
}
```

## Xcode

### #08: Support new iOS in old Xcode

Currently: Using iOS 14 devices with Xcode 11.5 (instead of Xcode 12)

See this [gist by @steipete](https://gist.github.com/steipete/d9b44d8e9f341e81414e86d7ff8fb62d).

### #07: Clear Cache for Launch Screen

The system caches resources from the launch screen. When changing the launch screen, this can be annoying.

```swift
import UIKit

public extension UIApplication {
    func clearLaunchScreenCache() {
        do {
            try FileManager.default.removeItem(atPath: NSHomeDirectory()+"/Library/SplashBoard")
        } catch {
            print("Failed to delete launch screen cache: \(error)")
        }
    }
}
```

Found [here](https://rambo.codes/posts/2019-12-09-clearing-your-apps-launch-screen-cache-on-ios).

### #06: Open Deep Link in Simulator

To test your apps deep links, you can open a deep link in the simulator like this:

```bash
xcrun simctl openurl booted "https://your-deep-link"
```

Found [in twitter by @alex_v_bush](https://twitter.com/alex_v_bush/status/1156917502942822401).

### #05: Select next occurrence

1. select the thing you want to edit
2. press-and-hold ⌥⌘E to "select next occurrence" until everything’s selected
3. type in the replacement; all of ‘em get changed

Found [in twitter by @davedelong](https://twitter.com/davedelong/status/1147229851373326336).

### #04: Adding identifier to constraints

You can add identifier to constraints in code or in Storyboard. This makes it easier when you need to debug.

```swift
let constraint = topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor)
constraint.identifier = "ViewXYToTop"
```

Found [in twitter by @twostraws](https://twitter.com/twostraws/status/1082939762363850752).

### #03: Weird problems with IBDesignable

When you have some weird problems with IBDesinable, try `killall ibtoold` and test again.

Another option is to read out the logs under `~/Library/Logs/DiagnosticReports` and they are named `IBDesignablesAgentCocoaTouch_*.crash`.

### #02: Set version number without fastlane

Have a look at: [ Automating Version and Build Numbers Using agvtool](https://developer.apple.com/library/archive/qa/qa1827/_index.html)

```ruby
##set version number to specific version
agvtool new-marketing-version <your_specific_version>

##updating build number
agvtool next-version -all

##set build number to specific version
agvtool new-version -all <your_specific_version>
```

Otherwise, you can use fastlane for that too: [https://docs.fastlane.tools/actions/increment_version_number/](https://docs.fastlane.tools/actions/increment_version_number/)

### #01: `po` in debugger

`po` in Xcode debugger means `print object`

```
test = 0
po [dictionary[@"test"] boolValue] //prints nil
print [dictionary[@"test"] boolValue] // prints NO
```

## git

### #05: Trunk based development and feature toggles

This was a new term for me, as I am mostly working with `git flow` at the moment. So I wrote together some points what trunk based development is and what I _think_ when it might be a good choice.

#### git flow

These are some points I am thinking of when working with `git flow`.

| 👍                                             | 👎                                                                    |
|------------------------------------------------|-----------------------------------------------------------------------|
| separate features                              | large merge conflicts (when feature branch is used for a longer time) |
| independent from other developers              | frequently merging from main branch in feature branch                 |
| "standard" CI possible                          | refactoring can be difficult                                              |
| good visualized code reviews in gitlab with MR | interaction of features is tested when features are already merged    |

Info: _"standard" CI means build a new dev-version-app when merged on develop and prod-version with tags._

#### What is trunk based development?

= working on one main branch
- activating features with feature toggles when they are finished
- different features can be activated manually to test their interaction
- always pull the latest changes before committing

#### trunk based

| 👍                                                           | 👎                                                    |
|--------------------------------------------------------------|-------------------------------------------------------|
| nearly synchronous development                               | different CI process                                  |
| less merge conflicts                                         | many commits from different features (may look messy) |
| timely correct history                                       | code review process is not so smooth                  |
| easily testing interactions of features (during development) |                                                       |
| faster releases (no merge before release)                    |                                                       |

Info: _The "messy" commits can be solved with adding a ticket number in front of the commit message, e.g. `[ABC-01] Update README`. To build and deploy an application you could use manual git triggers with options to build and release a develop or a release version._

#### How can code reviews be integrated in process?

- pushing changes on temporary branch (naming convention?)
- do the code review (in gitlab with a MR — which will not be merged — or directly in Xcode)
- fix discussions on temporary branch with additional commits
- merge temporary branch on master with `git merge --squash` to have all the changes in one commit and set a meaningful commit message
- delete the temporary branch (local and remote)

#### How can features be activated?

- release version:
    - activated from outside with a backend (e.g. with [firebase remote config](https://firebase.google.com/docs/remote-config/)?)
    - activated with a local configuration, like xml-file
- develop version:
    - activated with "secret gestures", like five-finger-tap, etc.
    - activated with a local configuration, like xml-file
- when features are controlled from outside the app
    - features can be released (or stopped) without an app update
    - easy A/B Tests possible

#### Feature toggles in Swift

This could look something like this:

```swift
enum Feature {
    case login
    case logout

    var ticketNumber: String {
        switch self {
        case .login:
            return "DS-01"
        case .logout:
            return "DS-02"
        }
    }

    var isActive: Bool{
        switch self {
        case .login:
            return true
        case .logout:
            return false
        }
    }
}

if Feature.login.isActive {
    print("new login")
} else {
    print("old login")
}
```

#### Additional Information & ToDos

- CI-Process
    - actions like `analyze` and `tests` are running all the time
    - build and deploy actions: using manual git triggers to release new apps (with option to release develop or release version)
- Is there a limit for the number of developers for using trunk based development?
- Xcode 10 has a better integrated source control which will be helpful with trunk based development
- Code review takes some time? (can lead to problems with every process)
- Working with "Mono-Repositories": conflicts or inconsequences will be found even earlier
- Duplicated code during refactoring?
- Old features may be forgotten to be removed when new features are activated permanently
- Architectural refactoring will be difficult

### #04: Rebase

`Do not force push! There is another solution most of the time.`

Have a look at: [github.com/k88hudson/git-flight-rules](https://github.com/k88hudson/git-flight-rules)

#### When to rebase?

When you want to apply commits from another branch, like `master`, on top of your current local branch, like your feature branch.

#### How to rebase?

1. Get the latest changes from `master` branch with `git pull master`.
2. Checkout the branch you want to rebase with `git checkout <branch>`.
3. Run `git rebase master`.
4. (Solve possible conflicts and run `git rebase --continue`)
6. Run `git fetch` to see changes in SourceTree

To cancel the rebase process early, run `git rebase --abort`.

#### You have pushed your feature branch and want to use rebase?

Do not use it, because you would have to use a `force push` and this will destroy the state of the repository for every other developer who has checked out the rebased branch before. Instead, you can use a simple merge to get the latest changes into your branch.

### #03: Squash

`Do not force push! There is another solution most of the time.`

Have a look at: [github.com/k88hudson/git-flight-rules](https://github.com/k88hudson/git-flight-rules)

#### When to squash commits?

When you want to merge a branch with just one clean commit.

This is the case when working in feature branches. When you want to merge your feature, the exact steps you made are not relevant anymore most of the time (but they can be, of course). So while working on a feature you can commit wildly and you can try things out and revert them later. Basically, you can focus on the feature. When the feature is finished and about to be merged, you think about a meaningful commit message and with `git merge <feature-branch> --squash` you can commit all the changes in only one commit.

#### How to squash commits?

1. You checkout the branch you want to integrate the new feature, with `git checkout <branch>`.
2. Use `git merge <feature-branch> --squash` to squash all the commits from the feature branch. You now have all the changes unstaged.
3. You can test your application once again and when everything is running, choose a meaningful commit message and commit all the changes on the branch.
4. Now you can push the feature with `git push` and delete the feature branch on local and remote side.

There is a similar option for this directly integrated in GitHub as well. So you can merge features from a pull request with one single commit.

### #02: Useful git commands

* `git reflog` to see the history of the latest commands, every command has a number
* `git reset HEAD@{<number>}` to get back to the state of when the command was executed.
* `git bisect` to find the commit which introduced a bug.

```bash
git bisect start
git bisect bad # current version not working
git bisect good adf71de61 # worked in this commit
```

* `git checkout develop -- <path/to/file>` to reset a file to the state on develop.

Two useful websites:

* A git choose-your-own-adventure: [On undoing, fixing, or removing commits in git](http://sethrobertson.github.io/GitFixUm/fixup.html)
* Oh shit git: [ohshitgit.com](http://ohshitgit.com/)

### #01: Ignore already tracked `XCUserstate`

1. Quit Xcode
2. Navigate to the project directory and execute the following commands
```ruby
ls -la #make sure you see the .git and .gitignore files
git rm --cached <projectFolder>.xcodeproj/project.xcworkspace/xcuserdata/<myUserName>.xcuserdatad/UserInterfaceState.xcuserstate
git commit -m "Removed file that shouldn't be tracked"
```
3. Change `ProjectFolder` in the example above to the name of your project
4. Relaunch Xcode and do a commit and verify that the `xcuserstate` file is no longer listed.

## HTML

### #01: `<details>` and `<summary>`

Use `<details>` and `<summary>` in GitHub to collapse large information in PR descriptions or comments.

```HTML
<details>
    <summary>Screenshot</summary>

    ![name](link)

</details>
```

Found [in twitter by @ashfurrow](https://twitter.com/ashfurrow/status/1100781360288014336).

## Android

### #03: Logging

Have a look at the official documentation: [https://developer.android.com/reference/android/util/Log](https://developer.android.com/reference/android/util/Log)

Import the `Log` class with `import android.util.Log`. There are different log leves, `d` is for Debug and `e` for error. 

Two examples how to log something: 

```kotlin
android.util.Log.d("feli", "feli: Logging is nice!")
```

```java
android.util.Log.e("feli", "feli: " + e.getMessage(), e);
```

You see the logs printed in the Logcat section in Android Studio or print them out to your console with: 

```bash
adb logcat -s feli
```

More information about how to use `logcat`: [https://developer.android.com/studio/command-line/logcat](https://developer.android.com/studio/command-line/logcat)

### #02: Integrate `ktlint` into project

To enforce similar code style and conventions, it's a good idea to integrate a linter, like [`ktlint`](https://github.com/pinterest/ktlint). 

Add the following to `app/build.gradle`:

```gradle
repositories {
    jcenter()
}

configurations {
    ktlint
}

dependencies {
    //...
    ktlint "com.pinterest:ktlint:0.32.0"
}

task ktlint(type: JavaExec, group: "verification") {
    description = "Check Kotlin code style."
    classpath = configurations.ktlint
    main = "com.pinterest.ktlint.Main"
    args "src/**/*.kt"
    // to generate report in checkstyle format prepend following args:
    // "--reporter=plain", "--reporter=checkstyle,output=${buildDir}/ktlint.xml"
    // see https://github.com/pinterest/ktlint#usage for more
}
check.dependsOn ktlint

task ktlintFormat(type: JavaExec, group: "formatting") {
    description = "Fix Kotlin code style deviations."
    classpath = configurations.ktlint
    main = "com.pinterest.ktlint.Main"
    args "-F", "src/**/*.kt"
}
```

To check code style, run:

```bash
./gradlew ktlint
```

To run formatter:

```bash
./gradle ktlintFormat
```

For further information, see the [official installation guide for gradle](https://github.com/pinterest/ktlint#-with-gradle).

### #01: Swift is like Kotlin

To see the parallels, have a look at: [http://nilhcem.com/swift-is-like-kotlin/](http://nilhcem.com/swift-is-like-kotlin/)
