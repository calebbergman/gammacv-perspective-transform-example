# gammacv-perspective-transform-example

# About

This project is to demonstrate the odd behavior GammaCV's perspective transform has on some iOS devices using [Ionic](https://ionicframework.com/) and [Capacitor](https://capacitor.ionicframework.com) (i.e. [WKWebView](https://developer.apple.com/documentation/webkit/wkwebview)). iPhone 6 and iPad Air 2 so far demonstrate the same (undesired) behavior. iPhone 7 and greater perform as desired. iPad Air 2 OS upgraded to iOS 12.4; undesired behavior persists.

## Project setup
```
yarn install
yarn build
npx cap add ios
yarn build:ios
```

## Xcode (MacOS only)

1. Open path/to/this/app/ios/App/App.xcworkspace

2. Assign your Signing Team in the target App's settings (click __App__ in the project explorer window/area, click the _General_ tab, click __App__ under _Targets_, and click the dropdown next to the __Team__ option)

3. Select your debug target in the title bar area

4. Click the run ▶ button

# Output

## Desired

![alt text](https://github.com/calebbergman/gammacv-perspective-transform-example/blob/master/public/output-desired.png?raw=true "Desired transform output")

## Undesired (iPad Air 2 iOS 12.4)

![alt text](https://github.com/calebbergman/gammacv-perspective-transform-example/blob/master/public/output-undesired.jpg?raw=true "Undesired transform output")

## Original

![alt text](https://github.com/calebbergman/gammacv-perspective-transform-example/blob/master/public/doc.png?raw=true "Original document")
