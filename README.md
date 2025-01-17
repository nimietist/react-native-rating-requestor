# react-native-rating-requestor

A React Native component to prompt users for a rating after positive interactions

The Rating Requestor is a very simple JS module that you simply instantiate and call from time to time, as your user performs actions that result in a "happy path." For example, maybe your users get a smile on their face every time they save money with your app, beat a level, or clear out their inbox. After a certain number of these positive events, it might be a good time to ask the user for a review. 

## Installation

    npm i --save react-native-rating-requestor

You'll probably also want to install the peer dependencies as well: 

    npm i --save @react-native-community/async-storage react-native-store-review

 You may also need to `link` in or `pod install` the native modules from the peer dependencies, which varies depending on the version of React Native you are using.

 *(If anyone has a better way of managing the native modules of these peer dependencies, definitely open an issue and let me know!)*

## Usage

Import and create a new instantiation of the Rating Requestor somewhere in the main portion of your application:

````javascript
    import RatingRequestor from 'react-native-rating-requestor';
    const RatingTracker = new RatingRequestor('[your apps store ID]');

    const MyApp = () => {...};
````
When a positive UX event occurs, let the Rating Requestor know so that it can keep track of these:

````javascript
	if (user_saved_the_world) {
		RatingTracker.handlePositiveEvent();
	}
````

The example above is used without callback. A callback can be provided that reports on result of the handling. The callback accepts two parameters: the first indicates whether the request dialog appeared (boolean), and the second returns the user decision (string: 'decline', 'delay', or 'accept').

````javascript
	if (user_saved_the_world) {
		RatingTracker.handlePositiveEvent(function(didAppear, userDecision) {
			if (didAppear) {
				switch(userDecision)
				{
					case 'decline': console.log('User declined to rate'); break;
					case 'delay'  : console.log('User delayed rating, will be asked later'); break;
					case 'accept' : console.log('User accepted invitation to rate, redirected to app store'); break;
				}
			} else {
				console.log('Request popup did not pop up. May appear on future positive events.');
			} 
		});
  }
````

If enough positive events have occurred (defined by the `timingFunction`, see "Configuration") then a rating dialog will appear. The user can rate the app or decline to rate, in which case they won't be bothered again, or can choose to maybe do so later, in which case the Rating Requestor will keep on tracking positive event counts.

You can also trigger the rating dialog to appear immediately by invoking `RatingTracker.showRatingDialog([callback])`. If you have a "Rate this App" button or link in an about page or something in your app, this would be a good place to use that.

### iOS Native Dialog

When on iOS 10.3+, if the user agrees to leave a review then it will trigger the native star picker dialog (SKStoreReviewController) built in to iOS. This is a definite improvement! However it's not foolproof. Because iOS ultimately makes the decision whether to show the dialog or not, there is a small chance that the user may agree to review the app, but because they've already seen the dialog recently, it may not appear. *Caveat emptor.* In most cases, though, this should work out better than before, because it immediately shows the star buttons instead of linking to the store first.

This is accompished by using [react-native-store](https://github.com/oblador/react-native-store-review), a peer dependency of this package.

## Configuration

All configuration occurs on the construction of a new RatingRequestor. 

````javascript
    const myRR = new RatingRequestor(appStoreId, [ options ]);
````

You *must* pass in a string as the first parameter, which is the app store ID of your application. Optionally, but highly suggested, is a second parameter: a set of options to customize the request dialog and the timing of the dialog. This object follows this pattern:

````javascript
  {
    title: {string},
    message: {string},
    actionLabels: {
      decline: {string},
      delay: {string},
      accept: {string}
    },
    buttonOrder: {
      ios: [buttonTypes],
      android: [buttonTypes],
    },
    shouldBoldLastButton: {boolean},
    storeAppName: {string},
    storeCountry: {string},
    timingFunction: {func(currentCount) => boolean}
  }
````

- `title`: A string used as the title for the dialog (e.g., "Please rate me!")
- `message`: The message you'd like to show the user (e.g., "If you are loving [my app's name], would you please leave me a positive review?")
- `actionLabels`: An object with three properties (all required if you don't want weird blanks or OKs):
  - `decline`: The "no thanks, I don't want to ever rate this" button label
  - `delay`: The "maybe I'll rate this later if I'm feeling charitable" button label
  - `accept`: The "oh my gosh I love this app so much so I'll rate it right now" button label
- `buttonOrder`: An object with platform keys (`ios` and `android` or others), with the value of each key being an array of length 3 containing `buttonType` values, which can be imported from the package. This defines the order the buttons will appear on the dialog. The default order for both platforms is as follows:
  - `[buttonTypes.NEGATIVE_DECLINE, buttonTypes.NEUTRAL_DELAY, buttonTypes.POSITIVE_ACCEPT]`
- `shouldBoldLastButton`: If passed as true, the third button of the modal will have a more prominent style. This option currently only modifies iOS styling
- `storeAppName`: iOS only, the slug that Apple uses in the URL to the app listing. Anything seems to work here, including the default `appName`, but might as well make it match what Apple has, right?
- `storeCountry`: Target a specific coutry's store, defaults to `us`.
- `timingFunction`: A method that takes the current total count of positive events recorded for the app, and returns if the Requestor should display the dialog or not. By default, the timingFunction evaluates as `3^n`, and if `3^n == currentCount` then it returns true/shows the dialog. Source looks like this:

```javascript
timingFunction: function(currentCount) {
    return currentCount > 1 && Math.log(currentCount) / Math.log(3) % 1 == 0;
}
```

## Notes

As of version 2.0.0 this package is compatible with both iOS and Android.

Peer dependencies and linking may be required, depending on your app and version of React Native.

## Releases

For more details, see CHANGELOG.md.

- 4.0.1 - Fix for missing `storeUrl`.
- 4.0.0 - Minimum RN version compatibility now at 0.57+, use community AsyncStorage (props to [@imranariffin](https://github.com/imranariffin)), use iOS native review request dialog (props to [@jasonlfunk](https://github.com/jasonlfunk))
- 3.2.1 - Remove unnecessary `console.log` call. Props to [@nlively](https://github.com/nlively)
- 3.2.0 - Allow iOS country store and app name to be set. Props to [@andreleon](https://github.com/andreleon)
- 3.1.0 - Formalize the `shouldBoldLastButton` option. Props to [@ttargo1](https://github.com/ttargo1)
- 3.0.0 - Allow changing the order of the buttons on different platforms. Impetus and help from [@yihanseattle](https://github.com/yihanseattle), [@olzhasipsy](https://github.com/olzhasipsy), and [@kesha-antonov](https://github.com/kesha-antonov), and update the minimum RN dep to v0.26.
- 2.1.0 - Update the iOS store URL for iOS 11 compatibility thanks to [@rodrigopk](https://github.com/rodrigopk)
- 2.0.0 - Supports Android, requires RN v0.20.0+, and added `showRatingDialog()` thanks to [@maximilianhurl](https://github.com/maximilianhurl).
- 1.1.0 - Added an optional callback to `handlePositiveEvent()` that reports on the result of the handling. Props to [@sercanov](https://github.com/sercanov).
- 1.0.0 - Initial release

## Questions?

Feel free to contact me:

- Twitter: [@jlyman](https://www.twitter.com/jlyman)
- Website: http://www.joshualyman.com/
