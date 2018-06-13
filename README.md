# Felis dev notes ✌️

My personal list of things I once learned in iOS, Swift, CocoaPods and git (etc.) and do not want to forget. The list is not ordered by any kind of type – only thought about some things and wrote them down.

## Table of contents

[[iOS] Target Environment Simulator](#ios-target-environment-simulator)\
[[Xcode] Weird problems with IBDesignable](#xcode-weird-problems-with-ibdesignable)\
[[iOS] Delay requests while searching using GCD](#ios-delay-requests-while-searching-using-gcd)\
[[iOS] WebViewController](#ios-webviewcontroller)\
[[iOS] Common Mistakes with Custom Fonts](#ios-common-mistakes-with-custom-fonts)\
[[iOS] IBOutletCollection](#ios-iboutletcollection)\
[[iOS] `debugPrint` in release environment](#ios-debugprint-in-release-environment)\
[[Xcode] Set version number without fastlane](#xcode-set-version-number-without-fastlane)\
[[iOS] Check for email addresses in String](#ios-check-for-email-addresses-in-string)\
[[cocoapods] Using versions for pods](#cocoapods-using-versions-for-pods)\
[[git] Rebase](#git-rebase)\
[[git] Squash](#git-squash)\
[[git] Useful git commands](#git-useful-git-commands)\
[[iOS] StackView & Warnings](#ios-stackview--warnings)\
[[iOS] Find error in plist](#ios-find-error-in-plist)\
[[iOS] Right and LeftView in UITextField](#ios-right-and-leftview-in-uitextfield)\
[[iOS] Dispatch Group](#ios-dispatch-group)\
[[Xcode] po in debugger](#xcode-po-in-debugger)\
[[iOS] ATS information](#ios-ats-information)\
[[iOS] Attributed Text Helper](#ios-attributed-text-helper)\
[[iOS] UITest helper](#ios-uitest-helper)\
[[iOS] Configure UIKit elements on initializer](#ios-configure-uikit-elements-on-initializer)\
[[iOS] Simple animation with UIImage](#ios-simple-animation-with-uiimage)\
[[iOS] UIImage with black & white effect](#ios-uiimage-with-black--white-effect)\
[[git] Ignore already tracked `XCUserstate`](#git-ignore-already-tracked-xcuserstate)

# [iOS] Target Environment Simulator
Have a look at: [https://swift.org/blog/swift-4-1-released](https://swift.org/blog/swift-4-1-released/)

You can use `#if targetEnvironment(simulator)` to check if your app runs in a simulator. This is really usefull, when you want to add some special handling for simulators, like taking a photo with a real device and using a dummy image in simulator. 

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

# [Xcode] Weird problems with IBDesignable
When you have some weird problems with IBDesinable, try `killall ibtoold` and test again. 

Another option is to read out the logs under `~/Library/Logs/DiagnosticReports` and they are named `IBDesignablesAgentCocoaTouch_*.crash`.

# [iOS] Delay requests while searching using GCD
Have a look at: [https://www.swiftbysundell.com/posts/a-deep-dive-into-grand-central-dispatch-in-swift](https://www.swiftbysundell.com/posts/a-deep-dive-into-grand-central-dispatch-in-swift).

How to use GCD to delay sending requests while searching.

```swift
classSearchViewController: UIViewController, UISearchBarDelegate {
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

# [iOS] WebViewController
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

# [iOS] Common Mistakes with Custom Fonts
Have a look at: [https://codewithchris.com/common-mistakes-with-adding-custom-fonts-to-your-ios-app/](https://codewithchris.com/common-mistakes-with-adding-custom-fonts-to-your-ios-app/)

# [iOS] IBOutletCollection
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

# [iOS] `debugPrint` in release environment
Have a look at: [http://swiftiostutorials.com/use-nslog-debugging/](http://swiftiostutorials.com/use-nslog-debugging/)

`debugPrint` has nothing to do with debug and release environments. It just gives you some information more than the normal `print`. 

# [Xcode] Set version number without fastlane
Have a look at: [ Automating Version and Build Numbers Using agvtool](https://developer.apple.com/library/archive/qa/qa1827/_index.html)
```ruby
##set version number to specific version
agvtool new-marketing-version <your_specific_version>

##updating build number
agvtool next-version -all

##set build number to specific version
agvtool new-version -all <your_specific_version>
```
Otherwise you can use fastlane for that too: [https://docs.fastlane.tools/actions/increment_version_number/](https://docs.fastlane.tools/actions/increment_version_number/)

# [iOS] Check for email addresses in String
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

# [cocoapods] Using versions for pods
Have a look at: [http://guides.cocoapods.org/using/the-podfile.html](http://guides.cocoapods.org/using/the-podfile.html).

* `~> 0.1.2` will get you a version up to 0.2 (but not including 0.2 and higher)
* `~> 0.1` will get you a version up to 1.0 (but not including 1.0 and higher)
* `~> 0` will get you a version of 0 and higher (same as if it was omitted)

# [git] Rebase

`Do not force push! There is another solution most of the time.`

## When to rebase?
Imagine, you are working on a feature branch, which is not pushed to remote yet (important) and you want to get the latest changes from another branch, like `master`.

## How to rebase?
1. Get the latest changes from `master` branch with `git pull master`.
2. Checkout the branch you want to rebase with `git checkout <branch>`.
3. Run `git rebase master`.
4. (Solve possible conflicts and run `git rebase --continue`)
6. Run `git fetch` to see changes in Sourcetree

To cancel the rebase process early, run `git rebase --abort`.

## You have pushed your feature branch and want to use rebase?
Do not use it, because you would have to use a `force push` and this will destroy the state of the repository for every other developer who has checked out the rebased branch before. Instead, you can use a simple merge to get the latest changes into your branch.

# [git] Squash

`Do not force push! There is another solution most of the time.`

## When to squash commits?
When you are working on a feature for some time and when you want to merge it, the exact steps you made are most of the time not relevant anymore (but they can be, of course). So while working on a feature you can commit wildly and you can try things out and revert than later, you can focus on the feature. When the feature is finished and about to be merged, you think about a meaningful commit message and can commit all the changes in one commit.

## How to squash commits?
1. You checkout the branch you want to integrate the new feature with `git checkout <branch>`.
2. Use `git merge <feature-branch> --squash` to squash all the commits from the feature branch. You now have all the changes unstaged. 
3. You can test your application once again and when everythin is running, think about a meaningful commit message and commit all the changes on the branch.
4. Now you can push the feature with `git push` and delete the feature branch on local and remote side.

There is a similar option for this directly integrated on GitHub as well. So you can merge features from a pull request with one commit.

# [git] Useful git commands
* `git reflog` to see the history of the latest commands, every command has a number
* `git reset HEAD@{<number>}` to get back to the state of when the command was executed.

# [iOS] StackView & Warnings
Have a look at: [http://stackoverflow.com/a/37447714/6716961](http://stackoverflow.com/a/37447714/6716961)

`Do not make layer changes in UIStackView!`

# [iOS] Find error in plist
`plutil <PATH_TO_FILE>`

# [iOS] Right and LeftView in UITextField
You can use a `rightView` or/and `leftView` in `UITextField`.

## When is it useful?
Well, it depends. 

When you want to use a custom elemt as "side element" of the text field and want to define it in storyboard, you just do not have to use the `rightView` parameter. This is really useful when you have the view configured in code and then you can simply add it to the text field and you do not have to set constraints etc.

```swift
@IBOutlet var customView: UIView!

func textFieldDidBeginEditing(_ textField: UITextField) {
	if textField == password {
		password.rightView = customView
		password.rightViewMode = .always
	}
}
```

## References
* [ Developer - Manage TextFieldTextViews](https://developer.apple.com/library/content/documentation/StringsTextFonts/Conceptual/TextAndWebiPhoneOS/ManageTextFieldTextViews/ManageTextFieldTextViews.html)
* [ Developer - RightView](https://developer.apple.com/reference/uikit/uitextfield/1619596-rightview)
* [ Developer - RigthViewMode](https://developer.apple.com/reference/uikit/uitextfield/1619607-rightviewmode)

# [iOS] Dispatch Group
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

# [Xcode] `po` in debugger
`po` in Xcode debugger means `print object`

```
test = 0
po [dictionary[@"test"] boolValue] //prints nil
print [dictionary[@"test"] boolValue] // prints NO
```

# [iOS] ATS information
There are the following criteria, for which you have to add an exception:
* http
* lower TLS version than 1.2
* cipher suites that do not support forward secrecy

## Define the problem
* Run in command line `nscurl --ats-diagnostics <url> --verbose`
* Check url on the  [ssltest](https://www.ssllabs.com/ssltest/) site from ssllabs.
* You can test your [browsers capabilities](https://www.ssllabs.com/ssltest/viewMyClient.html) in ssllabs too.

## Use exception
Available keys:
* `NSIncludesSubdomains`
* `NSExceptionAllowsInsecureHTTPLoads`
* `NSExceptionRequiresForwardSecrecy`
* `NSExceptionMinimumTLSVersion`
* `NSThirdPartyExceptionAllowsInsecureHTTPLoads`
* `NSThirdPartyExceptionMinimumTLSVersion`
* `NSThirdPartyExceptionRequiresForwardSecrecy`

### add in `info.plist`
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

## References
Getting Ready for ATS Enforcement in 2017

* [https://developer.apple.com/library/ios/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW33](https://developer.apple.com/library/ios/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW33)
* [https://ste.vn/2015/06/10/configuring-app-transport-security-ios-9-osx-10-11/](https://ste.vn/2015/06/10/configuring-app-transport-security-ios-9-osx-10-11/)
* [http://stackoverflow.com/questions/30720813/cfnetwork-sslhandshake-failed-ios-9](http://stackoverflow.com/questions/30720813/cfnetwork-sslhandshake-failed-ios-9)
* [http://www.heise.de/security/artikel/Zukunftssicher-Verschluesseln-mit-Perfect-Forward-Secrecy-1923800.html](http://www.heise.de/security/artikel/Zukunftssicher-Verschluesseln-mit-Perfect-Forward-Secrecy-1923800.html)
* [http://timekl.com/blog/2015/08/21/shipping-an-app-with-app-transport-security/](http://timekl.com/blog/2015/08/21/shipping-an-app-with-app-transport-security/)
* [http://www.neglectedpotential.com/2015/06/working-with-apples-application-transport-security/](http://www.neglectedpotential.com/2015/06/working-with-apples-application-transport-security/)

# [iOS] Attributed Text Helper
## non-breaking
* non breaking space: `\u00a0`
* non breaking hyphen: `\u2011`

## soft hypen
Have a look at: [http://stackoverflow.com/questions/33727659/how-to-perform-hyphenation-on-a-uilabel-sensitive-to-a-language-different-from-t](http://stackoverflow.com/questions/33727659/how-to-perform-hyphenation-on-a-uilabel-sensitive-to-a-language-different-from-t)

### Swift:
```swift
let title = "Title with Hyphen"

let paragraphStyle = NSMutableParagraphStyle()
paragraphStyle.hyphenationFactor = 1.0

let paragraphAttributes = [NSAttributedStringKey.paragraphStyle: paragraphStyle]
let attributedStringWithHyphens = NSAttributedString(string: title,
                                                     attributes: paragraphAttributes)

let label = UILabel()
label.attributedText = attributedStringWithHyphens
```

# [iOS] UITest helper
* `print(app.debugDescription)`
* Testing Autolayout: Edit Scheme -> Options -> Double Length Pseudolanguage
* use internal functions in test classes with `@testable import ProjectName`

# [iOS] Configure UIKit elements on initializer
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

# [iOS] Simple animation with UIImage
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

# [iOS] UIImage with black & white effect
Have a look at: [ Developer - CoreImageFilterReference](https://developer.apple.com/library/content/documentation/GraphicsImaging/Reference/CoreImageFilterReference/#//apple_ref/doc/filter/ci/CIPhotoEffectMono).

```swift
func convertToBlackAndWhite() -> UIImage {
	let filter = CIFilter(name: "CIPhotoEffectMono")!
	filter.setDefaults()
	filter.setValue(CoreImage.CIImage(image: self)!, forKey: kCIInputImageKey)
	return UIImage(CGImage: CIContext(options:nil).createCGImage(filter.outputImage!, fromRect: filter.outputImage!.extent))
}
```

# [git] Ignore already tracked `XCUserstate`
1. Quit Xcode
2. Navigate to the project directory and execute the following commands
```ruby
ls -la #make sure you see the .git and .gitignore files
git rm --cached <projectFolder>.xcodeproj/project.xcworkspace/xcuserdata/<myUserName>.xcuserdatad/UserInterfaceState.xcuserstate
git commit -m "Removed file that shouldn't be tracked"
```
3. Change `ProjectFolder` in the example above to the name of your project
4. Relaunch Xcode and do a commit and verify that the `xcuserstate` file is no longer listed.
