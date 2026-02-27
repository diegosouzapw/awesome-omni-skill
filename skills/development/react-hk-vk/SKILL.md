---
name: react
description: A framework for building native apps for Android, iOS, and more using React
metadata:
  source: llms.txt
  source_url: https://reactnative.dev/llms.txt
  generated: 2026-02-26T08:58:28.692Z
---

# React Native · Learn once, write anywhere

> A framework for building native apps for Android, iOS, and more using React

A framework for building native apps for Android, iOS, and more using React

## Available Resources

### docs

- **Introduction**: This helpful guide lays out the prerequisites for learning React Native, using these docs, and setting up your environment.
  - URL: /docs/getting-started.md

- **Get Started with React Native**: React Native allows developers who know React to create native apps. At the same time, native developers can use React Native to gain parity between native platforms by writing common features once.
  - URL: /docs/environment-setup.md

- **Set Up Your Environment**: In this guide, you'll learn how to set up your environment, so that you can run your project with Android Studio and Xcode. This will allow you to develop with Android emulators and iOS simulators, build your app locally, and more.
  - URL: /docs/set-up-your-environment.md

- **Integration with Existing Apps**: React Native is great when you are starting a new mobile app from scratch. However, it also works well for adding a single view or user flow to existing native applications. With a few steps, you can add new React Native based features, screens, views, etc.
  - URL: /docs/integration-with-existing-apps.md

- **Integration with an Android Fragment**: The guide for Integration with Existing Apps details how to integrate a full-screen React Native app into an existing Android app as an Activity.
  - URL: /docs/integration-with-android-fragment.md

- **Core Components and Native Components**: React Native lets you compose app interfaces using Native Components. Conveniently, it comes with a set of these components for you to get started with right now—the Core Components!
  - URL: /docs/intro-react-native-components.md

- **React Fundamentals**: To understand React Native fully, you need a solid foundation in React. This short introduction to React can help you get started or get refreshed.
  - URL: /docs/intro-react.md

- **Handling Text Input**: TextInput is a Core Component that allows the user to enter text. It has an onChangeText prop that takes a function to be called every time the text changed, and an onSubmitEditing prop that takes a function to be called when the text is submitted.
  - URL: /docs/handling-text-input.md

- **Using a ScrollView**: The ScrollView is a generic scrolling container that can contain multiple components and views. The scrollable items can be heterogeneous, and you can scroll both vertically and horizontally (by setting the horizontal property).
  - URL: /docs/using-a-scrollview.md

- **Using List Views**: React Native provides a suite of components for presenting lists of data. Generally, you'll want to use either FlatList or SectionList.
  - URL: /docs/using-a-listview.md

- **Troubleshooting**: These are some common issues you may run into while setting up React Native. If you encounter something that is not listed here, try searching for the issue in GitHub.
  - URL: /docs/troubleshooting.md

- **Platform-Specific Code**: When building a cross-platform app, you'll want to re-use as much code as possible. Scenarios may arise where it makes sense for the code to be different, for example you may want to implement separate visual components for Android and iOS.
  - URL: /docs/platform-specific-code.md

- **🗑️ Building For TV Devices**: TV devices support has been implemented with the intention of making existing React Native applications work on Apple TV and Android TV, with few or no changes needed in the JavaScript code for the applications.
  - URL: /docs/building-for-tv.md

- **Out-of-Tree Platforms**: React Native is not only for Android and iOS devices - our partners and the community maintain projects that bring React Native to other platforms, such as:
  - URL: /docs/out-of-tree-platforms.md

- **More Resources**: There’s always more to learn: developer workflows, shipping to app stores, internationalization, security and more.
  - URL: /docs/more-resources.md

- **Accessibility**: Create mobile apps accessible to assistive technology with React Native's suite of APIs designed to work with Android and iOS.
  - URL: /docs/accessibility.md

- **AccessibilityInfo**: Sometimes it's useful to know whether or not the device has a screen reader that is currently active. The AccessibilityInfo API is designed for this purpose. You can use it to query the current state of the screen reader as well as to register to be notified when the state of the screen reader changes.
  - URL: /docs/accessibilityinfo.md

- **ActionSheetIOS**: Displays native to iOS Action Sheet component.
  - URL: /docs/actionsheetios.md

- **ActivityIndicator**: Displays a circular loading indicator.
  - URL: /docs/activityindicator.md

- **Alert**: Launches an alert dialog with the specified title and message.
  - URL: /docs/alert.md

- **❌ AlertIOS**: Use Alert instead.
  - URL: /docs/alertios.md

- **Animated**: The Animated library is designed to make animations fluid, powerful, and painless to build and maintain. Animated focuses on declarative relationships between inputs and outputs, configurable transforms in between, and start/stop methods to control time-based animation execution.
  - URL: /docs/animated.md

- **Animated.Value**: Standard value for driving animations. One Animated.Value can drive multiple properties in a synchronized fashion, but can only be driven by one mechanism at a time. Using a new mechanism (e.g. starting a new animation, or calling setValue) will stop any previous ones.
  - URL: /docs/animatedvalue.md

- **Animated.ValueXY**: 2D Value for driving 2D animations, such as pan gestures. Almost identical API to normal Animated.Value, but multiplexed. Contains two regular Animated.Values under the hood.
  - URL: /docs/animatedvaluexy.md

- **Animations**: Animations are very important to create a great user experience. Stationary objects must overcome inertia as they start moving. Objects in motion have momentum and rarely come to a stop immediately. Animations allow you to convey physically believable motion in your interface.
  - URL: /docs/animations.md

- **App Extensions**: App extensions let you provide custom functionality and content outside of your main app. There are different types of app extensions on iOS, and they are all covered in the App Extension Programming Guide. In this guide, we'll briefly cover how you may take advantage of app extensions on iOS.
  - URL: /docs/app-extensions.md

- **Appearance**: The Appearance module exposes information about the user's appearance preferences, such as their preferred color scheme (light or dark).
  - URL: /docs/appearance.md

- **Appendix**: I. Terminology
  - URL: /docs/appendix.md

- **AppRegistry**: Project with Native Code Required
  - URL: /docs/appregistry.md

- **AppState**: AppState can tell you if the app is in the foreground or background, and notify you when the state changes.
  - URL: /docs/appstate.md

- **❌ AsyncStorage**: Use one of the community packages instead.
  - URL: /docs/asyncstorage.md

- **BackHandler**: The Backhandler API detects hardware button presses for back navigation, lets you register event listeners for the system's back action, and lets you control how your application responds. It is Android-only.
  - URL: /docs/backhandler.md

- **BoxShadowValue Object Type**: The BoxShadowValue object is taken by the boxShadow style prop. It is comprised of 2-4 lengths, an optional color, and an optional inset boolean. These values collectively define the box shadow's color, position, size, and blurriness.
  - URL: /docs/boxshadowvalue.md

- **Speeding up your Build phase**: Building your React Native app could be expensive and take several minutes of developers time.
  - URL: /docs/build-speed.md

- **Button**: A basic button component that should render nicely on any platform. Supports a minimal level of customization.
  - URL: /docs/button.md

- **❌ CheckBox**: Use one of the community packages instead.
  - URL: /docs/checkbox.md

- **❌ Clipboard**: Use one of the community packages instead.
  - URL: /docs/clipboard.md

- **Color Reference**: Components in React Native are styled using JavaScript. Color properties usually match how CSS works on the web. General guides on the color usage on each platform could be found below:
  - URL: /docs/colors.md

- **Communication between native and React Native**: In Integrating with Existing Apps guide and Native UI Components guide we learn how to embed React Native in a native component and vice versa. When we mix native and React Native components, we'll eventually find a need to communicate between these two worlds. Some ways to achieve that have been already mentioned in other guides. This article summarizes available techniques.
  - URL: /docs/communication-android.md

- **Communication between native and React Native**: In Integrating with Existing Apps guide and Native UI Components guide we learn how to embed React Native in a native component and vice versa. When we mix native and React Native components, we'll eventually find a need to communicate between these two worlds. Some ways to achieve that have been already mentioned in other guides. This article summarizes available techniques.
  - URL: /docs/communication-ios.md

- **Core Components and APIs**: React Native provides a number of built-in Core Components ready for you to use in your app. You can find them all in the left sidebar (or menu above, if you are on a narrow screen). If you're not sure where to get started, take a look at the following categories:
  - URL: /docs/components-and-apis.md

- **❌ DatePickerAndroid**: Use one of the community packages instead.
  - URL: /docs/datepickerandroid.md

- **❌ DatePickerIOS**: Use one of the community packages instead.
  - URL: /docs/datepickerios.md

- **Debugging Basics**: Debugging features, such as the Dev Menu, LogBox, and React Native DevTools are disabled in release (production) builds.
  - URL: /docs/debugging.md

- **Debugging Native Code**: Projects with Native Code Only
  - URL: /docs/debugging-native-code.md

- **Debugging Release Builds**: Symbolicating a stack trace
  - URL: /docs/debugging-release-builds.md

- **DevSettings**: The DevSettings module exposes methods for customizing settings for developers in development.
  - URL: /docs/devsettings.md

- **Dimensions**: useWindowDimensions is the preferred API for React components. Unlike Dimensions, it updates as the window's dimensions update. This works nicely with the React paradigm.
  - URL: /docs/dimensions.md

- **Document nodes**: Document nodes represent a complete native view tree. Apps using native navigation would provide a separate document node for each screen. Apps not using native navigation would generally provide a single document for the whole app (similar to single-page apps on Web).
  - URL: /docs/document-nodes.md

- **DrawerLayoutAndroid**: React component that wraps the platform DrawerLayout (Android only). The Drawer (typically used for navigation) is rendered with renderNavigationView and direct children are the main view (where your content goes). The navigation view is initially not visible on the screen, but can be pulled in from the side of the window specified by the drawerPosition prop and its width can be set by the drawerWidth prop.
  - URL: /docs/drawerlayoutandroid.md

- **DropShadowValue Object Type**: The DropShadowValue object is taken by the filter style prop for the dropShadow function. It is comprised of 2 or 3 lengths and an optional color. These values collectively define the drop shadow's color, position, and blurriness.
  - URL: /docs/dropshadowvalue.md

- **DynamicColorIOS**: The DynamicColorIOS function is a platform color type specific to iOS.
  - URL: /docs/dynamiccolorios.md

- **Easing**: The Easing module implements common easing functions. This module is used by Animated.timing() to convey physically believable motion in animations.
  - URL: /docs/easing.md

- **Element nodes**: Element nodes represent native components in the native view tree (similar to Element nodes on Web).
  - URL: /docs/element-nodes.md

- **Fabric Native Modules: Android**: Now it's time to write some Android platform code to be able to render the web view. The steps you need to follow are:
  - URL: /docs/fabric-native-components-android.md

- **Native Components**: If you want to build new React Native Components that wrap around a Host Component like a unique kind of CheckBox on Android, or a UIButton on iOS, you should use a Fabric Native Component.
  - URL: /docs/fabric-native-components-introduction.md

- **Fabric Native Components: iOS**: Now it's time to write some iOS platform code to be able to render the web view. The steps you need to follow are:
  - URL: /docs/fabric-native-components-ios.md

- **Fast Refresh**: Fast Refresh is a React Native feature that allows you to get near-instant feedback for changes in your React components. Fast Refresh is enabled by default, and you can toggle "Enable Fast Refresh" in the React Native Dev Menu. With Fast Refresh enabled, most edits should be visible within a second or two.
  - URL: /docs/fast-refresh.md

- **FlatList**: A performant interface for rendering basic, flat lists, supporting the most handy features:
  - URL: /docs/flatlist.md

- **Layout with Flexbox**: A component can specify the layout of its children using the Flexbox algorithm. Flexbox is designed to provide a consistent layout on different screen sizes.
  - URL: /docs/flexbox.md

- **Gesture Responder System**: The gesture responder system manages the lifecycle of gestures in your app. A touch can go through several phases as the app determines what the user's intention is. For example, the app needs to determine if the touch is scrolling, sliding on a widget, or tapping. This can even change during the duration of a touch. There can also be multiple simultaneous touches.
  - URL: /docs/gesture-responder-system.md

- **Get Started Without a Framework**: If you have constraints that are not served well by a Framework, or you prefer to write your own Framework, you can create a React Native app without using a Framework.
  - URL: /docs/getting-started-without-a-framework.md

- **✨ __DEV__**: You can use the DEV pseudo-global variable in the codebase to guard development-only blocks of code.
  - URL: /docs/global-__DEV__.md

- **AbortController**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-AbortController.md

- **AbortSignal**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-AbortSignal.md

- **alert**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-alert.md

- **Blob**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-Blob.md

- **cancelAnimationFrame**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-cancelAnimationFrame.md

- **cancelIdleCallback**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-cancelIdleCallback.md

- **clearInterval**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-clearInterval.md

- **clearTimeout**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-clearTimeout.md

- **console**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-console.md

- **EventCounts**: The global EventCounts class, as defined in Web specifications.
  - URL: /docs/global-EventCounts.md

- **fetch**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-fetch.md

- **File**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-File.md

- **FileReader**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-FileReader.md

- **FormData**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-FormData.md

- **global**: global is a legacy alias for globalThis, as defined in Node.js.
  - URL: /docs/global-global.md

- **Headers**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-Headers.md

- **IntersectionObserver 🧪**: The global IntersectionObserver interface, as defined in Web specifications. It provides a way to asynchronously observe changes in the intersection of a target element with an ancestor element or with a top-level document's viewport.
  - URL: /docs/global-intersectionobserver.md

- **IntersectionObserverEntry 🧪**: The IntersectionObserverEntry interface, as defined in Web specifications. It describes the intersection between the target element and its root container at a specific moment of transition.
  - URL: /docs/global-intersectionobserverentry.md

- **navigator**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-navigator.md

- **performance**: The global performance object, as defined in Web specifications.
  - URL: /docs/global-performance.md

- **PerformanceEntry**: The global PerformanceEntry class, as defined in Web specifications.
  - URL: /docs/global-PerformanceEntry.md

- **PerformanceEventTiming**: The global PerformanceEventTiming class, as defined in Web specifications.
  - URL: /docs/global-PerformanceEventTiming.md

- **PerformanceLongTaskTiming**: The global PerformanceLongTaskTiming class, as defined in Web specifications.
  - URL: /docs/global-PerformanceLongTaskTiming.md

- **PerformanceMark**: The global PerformanceMark class, as defined in Web specifications.
  - URL: /docs/global-PerformanceMark.md

- **PerformanceMeasure**: The global PerformanceMeasure class, as defined in Web specifications.
  - URL: /docs/global-PerformanceMeasure.md

- **PerformanceObserver**: The global PerformanceObserver class, as defined in Web specifications.
  - URL: /docs/global-PerformanceObserver.md

- **PerformanceObserverEntryList**: The global PerformanceObserverEntryList class, as defined in Web specifications.
  - URL: /docs/global-PerformanceObserverEntryList.md

- **PerformanceResourceTiming**: The global PerformanceResourceTiming class, as defined in Web specifications.
  - URL: /docs/global-PerformanceResourceTiming.md

- **process**: 🚧 This page is work in progress.
  - URL: /docs/global-process.md

- **queueMicrotask**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-queueMicrotask.md

- **Request**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-Request.md

- **requestAnimationFrame**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-requestAnimationFrame.md

- **requestIdleCallback**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-requestIdleCallback.md

- **Response**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-Response.md

- **self**: self is an alias for globalThis, as defined in Web specifications.
  - URL: /docs/global-self.md

- **setInterval**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-setInterval.md

- **setTimeout**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-setTimeout.md

- **URL**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-URL.md

- **URLSearchParams**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.\
  - URL: /docs/global-URLSearchParams.md

- **WebSocket**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-WebSocket.md

- **window**: window is an alias for globalThis, as defined in Web specifications.
  - URL: /docs/global-window.md

- **XMLHttpRequest**: 🚧 This page is work in progress, so please refer to the MDN documentation for more information.
  - URL: /docs/global-XMLHttpRequest.md

- **Handling Touches**: Users interact with mobile apps mainly through touch. They can use a combination of gestures, such as tapping on a button, scrolling a list, or zooming on a map. React Native provides components to handle all sorts of common gestures, as well as a comprehensive gesture responder system to allow for more advanced gesture recognition, but the one component you will most likely be interested in is the basic Button.
  - URL: /docs/handling-touches.md

- **Headless JS**: Headless JS is a way to run tasks in JavaScript while your app is in the background. It can be used, for example, to sync fresh data, handle push notifications, or play music.
  - URL: /docs/headless-js-android.md

- **Height and Width**: A component's height and width determine its size on the screen.
  - URL: /docs/height-and-width.md

- **Using Hermes**: Hermes is an open-source JavaScript engine optimized for React Native. For many apps, using Hermes will result in improved start-up time, decreased memory usage, and smaller app size when compared to JavaScriptCore.
  - URL: /docs/hermes.md

- **I18nManager**: The I18nManager module provides utilities for managing Right-to-Left (RTL) layout support for languages like Arabic, Hebrew, and others. It provides methods to control RTL behavior and check the current layout direction.
  - URL: /docs/i18nmanager.md

- **Image**: A React component for displaying different types of images, including network images, static resources, temporary local images, and images from local disk, such as the camera roll.
  - URL: /docs/image.md

- **Image Style Props**: Examples
  - URL: /docs/image-style-props.md

- **ImageBackground**: A common feature request from developers familiar with the web is background-image. To handle this use case, you can use the ` component, which has the same props as `, and add whatever children to it you would like to layer on top of it.
  - URL: /docs/imagebackground.md

- **❌ ImagePickerIOS**: Use one of the community packages instead.
  - URL: /docs/imagepickerios.md

- **Images**: Static Image Resources
  - URL: /docs/images.md

- **Improving User Experience**: Configure text inputs
  - URL: /docs/improvingux.md

- **InputAccessoryView**: A component which enables customization of the keyboard input accessory view on iOS. The input accessory view is displayed above the keyboard whenever a TextInput has focus. This component can be used to create custom toolbars.
  - URL: /docs/inputaccessoryview.md

- **🗑️ InteractionManager**: Avoid long-running work and use requestIdleCallback instead.
  - URL: /docs/interactionmanager.md

- **JavaScript Environment**: JavaScript Runtime
  - URL: /docs/javascript-environment.md

- **Keyboard**: Keyboard module to control keyboard events.
  - URL: /docs/keyboard.md

- **KeyboardAvoidingView**: This component will automatically adjust its height, position, or bottom padding based on the keyboard height to remain visible while the virtual keyboard is displayed.
  - URL: /docs/keyboardavoidingview.md

- **Layout Props**: More detailed examples about those properties can be found on the Layout with Flexbox page.
  - URL: /docs/layout-props.md

- **LayoutAnimation**: Automatically animates views to their new positions when the next layout happens.
  - URL: /docs/layoutanimation.md

- **LayoutEvent Object Type**: LayoutEvent object is returned in the callback as a result of component layout change, for example onLayout in View component.
  - URL: /docs/layoutevent.md

- **Direct Manipulation**: It is sometimes necessary to make changes directly to a component without using state/props to trigger a re-render of the entire subtree. When using React in the browser for example, you sometimes need to directly modify a DOM node, and the same is true for views in mobile apps. setNativeProps is the React Native equivalent to setting properties directly on a DOM node.
  - URL: /docs/legacy/direct-manipulation.md

- **Local libraries setup**: A local library is a library containing views or modules that's local to your app and not published to a registry. This is different from the traditional setup for view and modules in the sense that a local library is decoupled from your app's native code.
  - URL: /docs/legacy/local-library-setup.md

- **Android Native UI Components**: There are tons of native UI widgets out there ready to be used in the latest apps - some of them are part of the platform, others are available as third-party libraries, and still more might be in use in your very own portfolio. React Native has several of the most critical platform components already wrapped, like ScrollView and TextInput, but not all of them, and certainly not ones you might have written yourself for a previous app. Fortunately, we can wrap up these existing components for seamless integration with your React Native application.
  - URL: /docs/legacy/native-components-android.md

- **iOS Native UI Components**: There are tons of native UI widgets out there ready to be used in the latest apps - some of them are part of the platform, others are available as third-party libraries, and still more might be in use in your very own portfolio. React Native has several of the most critical platform components already wrapped, like ScrollView and TextInput, but not all of them, and certainly not ones you might have written yourself for a previous app. Fortunately, we can wrap up these existing components for seamless integration with your React Native application.
  - URL: /docs/legacy/native-components-ios.md

- **Android Native Modules**: Welcome to Native Modules for Android. Please start by reading the Native Modules Intro for an intro to what native modules are.
  - URL: /docs/legacy/native-modules-android.md

- **Native Modules Intro**: Sometimes a React Native app needs to access a native platform API that is not available by default in JavaScript, for example the native APIs to access Apple or Google Pay. Maybe you want to reuse some existing Objective-C, Swift, Java or C++ libraries without having to reimplement it in JavaScript, or write some high performance, multi-threaded code for things like image processing.
  - URL: /docs/legacy/native-modules-intro.md

- **iOS Native Modules**: Welcome to Native Modules for iOS. Please start by reading the Native Modules Intro for an intro to what native modules are.
  - URL: /docs/legacy/native-modules-ios.md

- **Native Modules NPM Package Setup**: Native modules are usually distributed as npm packages, except that on top of the usual JavaScript they will include some native code per platform. To understand more about npm packages you may find this guide useful.
  - URL: /docs/legacy/native-modules-setup.md

- **Using Libraries**: This guide introduces React Native developers to finding, installing, and using third-party libraries in their apps.
  - URL: /docs/libraries.md

- **Linking**: Linking gives you a general interface to interact with both incoming and outgoing app links.
  - URL: /docs/linking.md

- **Linking Libraries**: Not every app uses all the native capabilities, and including the code to support all those features would impact the binary size... But we still want to support adding these features whenever you need them.
  - URL: /docs/linking-libraries-ios.md

- **Metro**: React Native uses Metro to build your JavaScript code and assets.
  - URL: /docs/metro.md

- **Modal**: The Modal component is a basic way to present content above an enclosing view.
  - URL: /docs/modal.md

- **Native Platform**: Your application may need access to platform features that aren’t directly available from react-native or one of the hundreds of third-party libraries maintained by the community. Maybe you want to reuse some existing Objective-C, Swift, Java, Kotlin or C++ code from the JavaScript runtime. Whatever your reason, React Native exposes a powerful set of API to connect your native code to your JavaScript application code.
  - URL: /docs/native-platform.md

- **Navigating Between Screens**: Mobile apps are rarely made up of a single screen. Managing the presentation of, and transition between, multiple screens is typically handled by what is known as a navigator.
  - URL: /docs/navigation.md

- **Networking**: Many mobile apps need to load resources from a remote URL. You may want to make a POST request to a REST API, or you may need to fetch a chunk of static content from another server.
  - URL: /docs/network.md

- **Nodes from refs**: React Native apps render a native view tree that represents the UI, similar to how React DOM does on Web (the DOM tree). React Native provides imperative access to this tree via refs, which are returned by all native components (including those rendered by built-in components like View).
  - URL: /docs/nodes.md

- **Optimizing FlatList Configuration**: Terms
  - URL: /docs/optimizing-flatlist-configuration.md

- **Optimizing JavaScript loading**: Parsing and running JavaScript code requires memory and time. Because of this, as your app grows, it's often useful to delay loading code until it's needed for the first time. React Native comes with some standard optimizations that are on by default, and there are techniques you can adopt in your own code to help React load your app more efficiently. There are also some advanced automatic optimizations (with their own tradeoffs) that are suitable for very large apps.
  - URL: /docs/optimizing-javascript-loading.md

- **Other Debugging Methods**: This page covers how to use legacy JavaScript debugging methods. If you are getting started with a new React Native or Expo app, we recommend using React Native DevTools.
  - URL: /docs/other-debugging-methods.md

- **PanResponder**: PanResponder reconciles several touches into a single gesture. It makes single-touch gestures resilient to extra touches, and can be used to recognize basic multi-touch gestures.
  - URL: /docs/panresponder.md

- **Performance Overview**: A compelling reason to use React Native instead of WebView-based tools is to achieve at least 60 frames per second and provide a native look and feel to your apps. Whenever feasible, we aim for React Native to handle optimizations automatically, allowing you to focus on your app without worrying about performance. However, there are certain areas where we haven't quite reached that level yet, and others where React Native (similar to writing native code directly) cannot determine the best optimization approach for you. In such cases, manual intervention becomes necessary. We strive to deliver buttery-smooth UI performance by default, but there may be instances where that isn't possible.
  - URL: /docs/performance.md

- **PermissionsAndroid**: Project with Native Code Required
  - URL: /docs/permissionsandroid.md

- **PixelRatio**: PixelRatio gives you access to the device's pixel density and font scale.
  - URL: /docs/pixelratio.md

- **Platform**: Example
  - URL: /docs/platform.md

- **PlatformColor**: You can use the PlatformColor function to access native colors on the target platform by supplying the native color’s corresponding string value. You pass a string to the PlatformColor function and, provided it exists on that platform, it will return the corresponding native color, which you can apply in any part of your application.
  - URL: /docs/platformcolor.md

- **Pressable**: Pressable is a Core Component wrapper that can detect various stages of press interactions on any of its defined children.
  - URL: /docs/pressable.md

- **PressEvent Object Type**: PressEvent object is returned in the callback as a result of user press interaction, for example onPress in Button component.
  - URL: /docs/pressevent.md

- **Profiling**: Profiling is the process of analyzing an app's performance, resource usage, and behavior to identify potential bottlenecks or inefficiencies. It's worth making use of profiling tools to ensure your app works smoothly across different devices and conditions.
  - URL: /docs/profiling.md

- **🗑️ ProgressBarAndroid**: Use one of the community packages instead.
  - URL: /docs/progressbarandroid.md

- **Props**: Most components can be customized when they are created, with different parameters. These created parameters are called props, short for properties.
  - URL: /docs/props.md

- **Publishing to Apple App Store**: The publishing process is the same as any other native iOS app, with some additional considerations to take into account.
  - URL: /docs/publishing-to-app-store.md

- **🗑️ PushNotificationIOS**: Use one of the community packages instead.
  - URL: /docs/pushnotificationios.md

- **React Native DevTools**: React Native DevTools is our modern debugging experience for React Native. Purpose-built from the ground up, it aims to be fundamentally more integrated, correct, and reliable than previous debugging methods.
  - URL: /docs/react-native-devtools.md

- **React Native Gradle Plugin**: This guide describes how to configure the React Native Gradle Plugin (often referred as RNGP), when building your React Native application for Android.
  - URL: /docs/react-native-gradle-plugin.md

- **React Node Object Type**: A React Node is one of the following types:
  - URL: /docs/react-node.md

- **Rect Object Type**: Rect accepts numeric pixel values to describe how far to extend a rectangular area. These values are added to the original area's size to expand it.
  - URL: /docs/rect.md

- **RefreshControl**: This component is used inside a ScrollView or ListView to add pull to refresh functionality. When the ScrollView is at scrollY: 0, swiping down triggers an onRefresh event.
  - URL: /docs/refreshcontrol.md

- **Releases Overview**: New React Native releases are shipped every two months, usually resulting in six (6) new minors per year.
  - URL: /docs/releases.md

- **Release Levels**: React Native provides the community with the ability to adopt individual new features as soon as their design and implementation are nearly complete, even before they are included in a stable release. This approach is known as release levels.
  - URL: /docs/releases/release-levels.md

- **Versioning Policy**: This page describes the versioning policy we follow for the react-native package.
  - URL: /docs/releases/versioning-policy.md

- **RootTag**: RootTag is an opaque identifier assigned to the native root view of your React Native surface — i.e. the ReactRootView or RCTRootView instance for Android or iOS respectively. In short, it is a surface identifier.
  - URL: /docs/roottag.md

- **Running On Device**: It's always a good idea to test your app on an actual device before releasing it to your users. This document will guide you through the necessary steps to run your React Native app on a device and to get it ready for production.
  - URL: /docs/running-on-device.md

- **Running On Simulator**: Starting the simulator
  - URL: /docs/running-on-simulator-ios.md

- **🗑️ SafeAreaView**: Use react-native-safe-area-context instead.
  - URL: /docs/safeareaview.md

- **ScrollView**: Component that wraps platform ScrollView while providing integration with touch locking "responder" system.
  - URL: /docs/scrollview.md

- **SectionList**: A performant interface for rendering sectioned lists, supporting the most handy features:
  - URL: /docs/sectionlist.md

- **Security**: Security is often overlooked when building apps. It is true that it is impossible to build software that is completely impenetrable—we’ve yet to invent a completely impenetrable lock (bank vaults do, after all, still get broken into). However, the probability of falling victim to a malicious attack or being exposed for a security vulnerability is inversely proportional to the effort you’re willing to put in to protecting your application against any such eventuality. Although an ordinary padlock is pickable, it is still much harder to get past than a cabinet hook!
  - URL: /docs/security.md

- **❌ SegmentedControlIOS**: Use one of the community packages instead.
  - URL: /docs/segmentedcontrolios.md

- **Settings**: Settings serves as a wrapper for NSUserDefaults, a persistent key-value store available only on iOS.
  - URL: /docs/settings.md

- **Shadow Props**: ---
  - URL: /docs/shadow-props.md

- **Share**: Example
  - URL: /docs/share.md

- **Publishing to Google Play Store**: Android requires that all apps be digitally signed with a certificate before they can be installed. In order to distribute your Android application via Google Play store it needs to be signed with a release key that then needs to be used for all future updates. Since 2017 it is possible for Google Play to manage signing releases automatically thanks to App Signing by Google Play functionality. However, before your application binary is uploaded to Google Play it needs to be signed with an upload key. The Signing Your Applications page on Android Developers documentation describes the topic in detail. This guide covers the process in brief, as well as lists the steps required to package the JavaScript bundle.
  - URL: /docs/signed-apk-android.md

- **State**: There are two types of data that control a component: props and state. props are set by the parent and they are fixed throughout the lifetime of a component. For data that is going to change, we have to use state.
  - URL: /docs/state.md

- **StatusBar**: Component to control the app's status bar. The status bar is the zone, typically at the top of the screen, that displays the current time, Wi-Fi and cellular network information, battery level and/or other status icons.
  - URL: /docs/statusbar.md

- **❌ StatusBarIOS**: Use StatusBar for mutating the status bar.
  - URL: /docs/statusbarios.md

- **Strict TypeScript API (opt in)**: The Strict TypeScript API is a preview of our future, stable JavaScript API for React Native.
  - URL: /docs/strict-typescript-api.md

- **Style**: With React Native, you style your application using JavaScript. All of the core components accept a prop named style. The style names and values usually match how CSS works on the web, except names are written using camel casing, e.g. backgroundColor rather than background-color.
  - URL: /docs/style.md

- **StyleSheet**: A StyleSheet is an abstraction similar to CSS StyleSheets.
  - URL: /docs/stylesheet.md

- **Switch**: Renders a boolean input.
  - URL: /docs/switch.md

- **Systrace**: Systrace is a standard Android marker-based profiling tool (and is installed when you install the Android platform-tools package). Profiled code blocks are surrounded by start/end markers which are then visualized in a colorful chart format. Both the Android SDK and React Native framework provide standard markers that you can visualize.
  - URL: /docs/systrace.md

- **TargetEvent Object Type**: TargetEvent object is returned in the callback as a result of focus change, for example onFocus or onBlur in the TextInput component.
  - URL: /docs/targetevent.md

- **Testing**: This guide introduces React Native developers to the key concepts behind testing, how to write good tests, and what kinds of tests you can incorporate into your workflow.
  - URL: /docs/testing-overview.md

- **Text**: A React component for displaying text.
  - URL: /docs/text.md

- **Text nodes**: Text nodes represent raw text content on the tree (similar to Text nodes on Web). They are not directly accessible via refs, but can be accessed using methods like childNodes on element refs.
  - URL: /docs/text-nodes.md

- **Text Style Props**: Example
  - URL: /docs/text-style-props.md

- **TextInput**: A foundational component for inputting text into the app via a keyboard. Props provide configurability for several features, such as auto-correction, auto-capitalization, placeholder text, and different keyboard types, such as a numeric keypad.
  - URL: /docs/textinput.md

- **Advanced Topics on Native Modules Development**: This document contains a set of advanced topics to implement more complex functionalities of Native Components. It is recommended to first read the Codegen section and the guides on Native Components.
  - URL: /docs/the-new-architecture/advanced-topics-components.md

- **Advanced Topics on Native Modules Development**: This document contains a set of advanced topics to implement more complex functionalities of Native Modules. It is recommended to first read the Codegen section and the guides on Native Modules.
  - URL: /docs/the-new-architecture/advanced-topics-modules.md

- **The Codegen CLI**: Calling Gradle or manually calling a script might be hard to remember and it requires a lot of ceremony.
  - URL: /docs/the-new-architecture/codegen-cli.md

- **Create a Library for Your Module**: React Native has a rich ecosystem of libraries to solve common problems. We collect React Native libraries in the reactnative.directory website, and this is a great resource to bookmark for every React Native developer.
  - URL: /docs/the-new-architecture/create-module-library.md

- **Advanced: Custom C++ Types**: This guide assumes that you are familiar with the Pure C++ Turbo Native Modules guide. This will build on top of that guide.
  - URL: /docs/the-new-architecture/custom-cxx-types.md

- **Direct Manipulation**: It is sometimes necessary to make changes directly to a component without using state/props to trigger a re-render of the entire subtree. When using React in the browser for example, you sometimes need to directly modify a DOM node, and the same is true for views in mobile apps. setNativeProps is the React Native equivalent to setting properties directly on a DOM node.
  - URL: /docs/the-new-architecture/direct-manipulation-new-architecture.md

- **Invoking native functions on your native component**: In the base guide to write a new Native Component, you have explored how to create a new component, how to pass properties from the JS side to the native side, and how to emit events from native side to JS.
  - URL: /docs/the-new-architecture/fabric-component-native-commands.md

- **Measuring the Layout**: Sometimes, you need to measure the current layout to apply some changes to the overall layout or to make decisions and call some specific logic.
  - URL: /docs/the-new-architecture/layout-measurements.md

- **Emitting Events in Native Modules**: In some circustamces, you may want to have a Native Module that listen to some events in the platform layer and then emit them to the JavaScript layer, to let you application react to such native events. In other cases, you might have long running operations that can emits events so that the UI can be updated when those happen.
  - URL: /docs/the-new-architecture/native-modules-custom-events.md

- **Native Modules Lifecycle**: In React Native, Native Modules are singleton. The Native Module infrastructure lazily creates a Native Module the first time it is accessed and it keeps it around whenever the app requires it. This is a performance optimization that allows us to avoid the overhead of creating Native Modules eagerly, at app start, and it ensure faster startup times.
  - URL: /docs/the-new-architecture/native-modules-lifecycle.md

- **Cross-Platform Native Modules (C++)**: Writing a module in C++ is the best way to share platform-agnostic code between Android and iOS. With pure C++ modules, you can write your logic only once and reuse it right away from all the platforms, without the need of writing platform-specific code.
  - URL: /docs/the-new-architecture/pure-cxx-modules.md

- **iOS - Using Swift in Your Native Modules**: Swift is the official and default language for developing native application on iOS.
  - URL: /docs/the-new-architecture/turbo-modules-with-swift.md

- **Using Codegen**: This guide teaches how to:
  - URL: /docs/the-new-architecture/using-codegen.md

- **What is Codegen?**: Codegen is a tool to avoid writing a lot of repetitive code. Using Codegen is not mandatory: you can write all the generated code manually. However, Codegen generates scaffolding code that could save you a lot of time.
  - URL: /docs/the-new-architecture/what-is-codegen.md

- **❌ TimePickerAndroid**: Use one of the community packages instead.
  - URL: /docs/timepickerandroid.md

- **Timers**: Timers are an important part of an application and React Native implements the browser timers.
  - URL: /docs/timers.md

- **ToastAndroid**: React Native's ToastAndroid API exposes the Android platform's ToastAndroid module as a JS module. It provides the method show(message, duration) which takes the following parameters:
  - URL: /docs/toastandroid.md

- **TouchableHighlight**: If you're looking for a more extensive and future-proof way to handle touch-based input, check out the Pressable API.
  - URL: /docs/touchablehighlight.md

- **TouchableNativeFeedback**: If you're looking for a more extensive and future-proof way to handle touch-based input, check out the Pressable API.
  - URL: /docs/touchablenativefeedback.md

- **TouchableOpacity**: If you're looking for a more extensive and future-proof way to handle touch-based input, check out the Pressable API.
  - URL: /docs/touchableopacity.md

- **TouchableWithoutFeedback**: If you're looking for a more extensive and future-proof way to handle touch-based input, check out the Pressable API.
  - URL: /docs/touchablewithoutfeedback.md

- **Transforms**: Transforms are style properties that will help you modify the appearance and position of your components using 2D or 3D transformations. However, once you apply transforms, the layouts remain the same around the transformed component hence it might overlap with the nearby components. You can apply margin to the transformed component, the nearby components or padding to the container to prevent such overlaps.
  - URL: /docs/transforms.md

- **Turbo Native Modules: Android**: Now it's time to write some Android platform code to make sure localStorage survives after the application is closed.
  - URL: /docs/turbo-native-modules-android.md

- **Native Modules**: Your React Native application code may need to interact with native platform APIs that aren't provided by React Native or an existing library. You can write the integration code yourself using a Turbo Native Module. This guide will show you how to write one.
  - URL: /docs/turbo-native-modules-introduction.md

- **Turbo Native Modules: iOS**: Now it's time to write some iOS platform code to make sure localStorage survives after the application is closed.
  - URL: /docs/turbo-native-modules-ios.md

- **Learn the Basics**: React Native is like React, but it uses native components instead of web components as building blocks. So to understand the basic structure of a React Native app, you need to understand some of the basic React concepts, like JSX, components, state, and props. If you already know React, you still need to learn some React Native specific stuff, like the native components. This tutorial is aimed at all audiences, whether you have React experience or not.
  - URL: /docs/tutorial.md

- **Using TypeScript**: TypeScript is a language which extends JavaScript by adding type definitions. New React Native projects target TypeScript by default, but also support JavaScript and Flow.
  - URL: /docs/typescript.md

- **Upgrading to new versions**: Upgrading to new versions of React Native will give you access to more APIs, views, developer tools and other goodies. Upgrading requires a small amount of effort, but we try to make it straightforward for you.
  - URL: /docs/upgrading.md

- **useColorScheme**: The useColorScheme React hook provides and subscribes to color scheme updates from the Appearance module. The return value indicates the current user preferred color scheme. The value may be updated later, either through direct user action (e.g. theme selection in device settings) or on a schedule (e.g. light and dark themes that follow the day/night cycle).
  - URL: /docs/usecolorscheme.md

- **useWindowDimensions**: useWindowDimensions automatically updates all of its values when screen size or font scale changes. You can get your application window's width and height like so:
  - URL: /docs/usewindowdimensions.md

- **Vibration**: Vibrates the device.
  - URL: /docs/vibration.md

- **View**: The most fundamental component for building a UI, View is a container that supports layout with flexbox, style, some touch handling, and accessibility controls. View maps directly to the native view equivalent on whatever platform React Native is running on, whether that is a UIView, `, android.view`, etc.
  - URL: /docs/view.md

- **View Style Props**: Example
  - URL: /docs/view-style-props.md

- **ViewToken Object Type**: ViewToken object is returned as one of the properties in the onViewableItemsChanged callback (for example, in the FlatList component). It is exported by ViewabilityHelper.js.
  - URL: /docs/viewtoken.md

- **VirtualizedList**: Base implementation for the more convenient ` and  components, which are also better documented. In general, this should only really be used if you need more flexibility than FlatList` provides, e.g. for use with immutable data instead of plain arrays.
  - URL: /docs/virtualizedlist.md

- **VirtualView 🧪**: VirtualView is a core component that behaves similar to View.
  - URL: /docs/virtualview.md

### architecture

- **Bundled Hermes**: This page gives an overview of how Hermes and React Native are built.
  - URL: /architecture/bundled-hermes.md

- **Fabric**: Fabric is React Native's new rendering system, a conceptual evolution of the legacy render system. The core principles are to unify more render logic in C++, improve interoperability with host platforms, and to unlock new capabilities for React Native. Development began in 2018 and in 2021, React Native in the Facebook app is backed by the new renderer.
  - URL: /architecture/fabric-renderer.md

- **Glossary**: Dev Menu
  - URL: /architecture/glossary.md

- **About the New Architecture**: Since 2018, the React Native team has been redesigning the core internals of React Native to enable developers to create higher-quality experiences. As of 2024, this version of React Native has been proven at scale and powers production apps by Meta.
  - URL: /architecture/landing-page.md

- **Architecture Overview**: Welcome to the Architecture Overview! If you're getting started with React Native, please refer to Guides section. Continue reading to learn how internals of React Native work!
  - URL: /architecture/overview.md

- **Render, Commit, and Mount**: The React Native renderer goes through a sequence of work to render React logic to a host platform. This sequence of work is called the render pipeline and occurs for initial renders and updates to the UI state. This document goes over the render pipeline and how it differs in those scenarios.
  - URL: /architecture/render-pipeline.md

- **Threading Model**: The React Native renderer distributes the work of the render pipeline across multiple threads.
  - URL: /architecture/threading-model.md

- **View Flattening**: View Flattening is an optimization by the React Native renderer to avoid deep layout trees.
  - URL: /architecture/view-flattening.md

- **Cross Platform Implementation**: The React Native renderer utilizes a core render implementation to be shared across platforms
  - URL: /architecture/xplat-implementation.md

### community

- **Communities**: The React Native Community
  - URL: /community/communities.md

- **Overview**: The React Native Community
  - URL: /community/overview.md

- **Staying up to date**: The React Native Community
  - URL: /community/staying-updated.md

- **Where to get help**: Where to get help
  - URL: /community/support.md

### showcase

- **Who is using React Native?**: Thousands of apps are using React Native, check out these apps!
  - URL: /showcase.md

### contributing

- **Bots Reference**: pull-bot
  - URL: /contributing/bots-reference.md

- **Changelogs in Pull Requests**: The changelog entry in your pull request serves as a sort of "tl;dr do they affect Android? are these breaking changes? is something new being added?
  - URL: /contributing/changelogs-in-pull-requests.md

- **Contribution License Agreement**: You must sign a Contribution License Agreement (CLA) before your pull request can be merged. This a one-time requirement for Meta projects in GitHub. You can read more about Contributor License Agreements (CLA) on Wikipedia.
  - URL: /contributing/contribution-license-agreement.md

- **How to Build from Source**: You will need to build React Native from source if you want to work on a new feature/bug fix, try out the latest features which are not released yet, or maintain your own fork with patches that cannot be merged to the core.
  - URL: /contributing/how-to-build-from-source.md

- **How to Contribute Code**: Thank you for your interest in contributing to React Native! From commenting on and triaging issues, to reviewing and sending PRs, all contributions are welcome. In this document, we'll cover the steps to contributing code to React Native.
  - URL: /contributing/how-to-contribute-code.md

- **How to Open a Pull Request**: These instructions provide the step-by-step process to set up your machine to make contributions to the core React Native repository, and create your first pull request.
  - URL: /contributing/how-to-open-a-pull-request.md

- **How to Report a Bug**: Reporting a bug for React Native is one of the best way to start contributing to the project. We use GitHub issues as the main channel for handling new bug reports.
  - URL: /contributing/how-to-report-a-bug.md

- **How to Run and Write Tests**: Running Tests
  - URL: /contributing/how-to-run-and-write-tests.md

- **Labeling GitHub Issues**: Most of our labels have a prefix that provides a hint of their purpose.
  - URL: /contributing/labeling-github-issues.md

- **Managing Pull Requests**: Reviewing a pull request can take a considerable amount of time. In some cases, the review might require more time to perform than it took someone to write and submit their changes! It's therefore necessary to do some preliminary work to ensure each pull request is in a good state to be reviewed.
  - URL: /contributing/managing-pull-requests.md

- **Contributing Overview**: How to contribute to React Native
  - URL: /contributing/overview.md

- **Triaging GitHub Issues**: Start out by looking at issues that need triage, as identified by the "Needs: Triage" label.
  - URL: /contributing/triaging-github-issues.md

### versions

- **React Native versions**
  - URL: /versions.md

### blog

- **Blog · React Native**: Blog
  - URL: /blog.md

- **First-class Support for TypeScript**: With the release of 0.71, React Native is investing in the TypeScript experience with the following changes:
  - URL: /blog/2023/01/03/typescript-first.md

- **React Native 0.71: TypeScript by Default, Flexbox Gap, and more...**: Today we’re releasing React Native version 0.71! This is a feature-packed release including:
  - URL: /blog/2023/01/12/version-071.md

- **React Native 0.71-RC0 Android outage postmortem**: Now that 0.71 is available, we want to share some key information about the incident that broke Android builds for all React Native versions while releasing the first 0.71 release candidate for React Native & Expo Android builds on November 4th, 2022.
  - URL: /blog/2023/01/27/71rc1-android-outage-postmortem.md

- **React Native 0.72 - Symlink Support, Better Errors, and more**: Today we’re releasing 0.72!
  - URL: /blog/2023/06/21/0.72-metro-package-exports-symlinks.md

- **Package Exports Support in React Native**: With the release of React Native 0.72, Metro — our JavaScript build tool — now includes beta support for the package.json "exports" field. When enabled, it adds the following functionality:
  - URL: /blog/2023/06/21/package-exports-support.md

- **React Native 0.73 - Debugging Improvements, Stable Symlink Support, and more**: Today we're releasing React Native 0.73! This release adds improvements to debugging with Hermes, stable symlink support, Android 14 support, and new experimental features. We are also deprecating legacy debugging features, and are releasing the next pillar of the New Architecture: Bridgeless Mode!
  - URL: /blog/2023/12/06/0.73-debugging-improvements-stable-symlinks.md

- **React Native 0.74 - Yoga 3.0, Bridgeless New Architecture, and more**: Today we're releasing React Native 0.74! This release adds Yoga 3.0, Bridgeless by default under the New Architecture, batched onLayout updates (New Architecture), and Yarn 3 as the default package manager for new projects.
  - URL: /blog/2024/04/22/release-0.74.md

- **Use a framework to build React Native apps**: At React Conf, we updated our guidance on the best tool to get started building React Native apps: a React Native framework - a toolbox with all the necessary APIs to let you build production-ready apps.
  - URL: /blog/2024/06/25/use-a-framework-to-build-react-native-apps.md

- **React Native 0.75 - Support for Percentage Values in Layout, New Architecture Stabilization, Template & init Updates, and more**: Today we are excited to release React Native 0.75!
  - URL: /blog/2024/08/12/release-0.75.md

- **React Native 0.76 - New Architecture by default, React Native DevTools, and more**: Today we are excited to release React Native 0.76!
  - URL: /blog/2024/10/23/release-0.76-new-architecture.md

- **New Architecture is here**: React Native 0.76 with the New Architecture by default is now available on npm!
  - URL: /blog/2024/10/23/the-new-architecture-is-here.md

- **React Native 0.77 - New Styling Features, Android’s 16KB page support, Swift Template**: Today we are excited to release React Native 0.77!
  - URL: /blog/2025/01/21/version-0.77.md

- **React Native Core Contributor Summit 2024 Recap**: Every year, the core contributors in the React Native Community get together with the React Native team to collaboratively shape the direction of this project.
  - URL: /blog/2025/02/03/react-native-core-contributor-summit-2024.md

- **React Native 0.78 - React 19 and more**: Today we are excited to release React Native 0.78!
  - URL: /blog/2025/02/19/react-native-0.78.md

- **React Native 0.79 - Faster tooling and much more**: Today we are excited to release React Native 0.79!
  - URL: /blog/2025/04/08/react-native-0.79.md

- **Moving Towards a Stable JavaScript API (New Changes in 0.80)**: In React Native 0.80, we're introducing two significant changes to React Native's JavaScript API — the deprecation of deep imports, and our new Strict TypeScript API. These are part of an ongoing effort to accurately define our API and offer dependable type safety to users and frameworks.
  - URL: /blog/2025/06/12/moving-towards-a-stable-javascript-api.md

- **React Native 0.80 - React 19.1, JS API Changes, Freezing Legacy Arch and much more**: Today we are excited to release React Native 0.80!
  - URL: /blog/2025/06/12/react-native-0.80.md

- **React Native 0.81 - Android 16 support, faster iOS builds, and more**: Today we are excited to release React Native 0.81!
  - URL: /blog/2025/08/12/react-native-0.81.md

- **React Native 0.82 - A New Era**: Today we're excited to release React Native 0.82: the first React Native that runs entirely on the New Architecture.
  - URL: /blog/2025/10/08/react-native-0.82.md

- **React Native 0.83 - React 19.2, New DevTools features, no breaking changes**: Today we are excited to release React Native 0.83!
  - URL: /blog/2025/12/10/react-native-0.83.md

- **React Native 0.84 - Hermes V1 by Default**: Today we're excited to release React Native 0.84!
  - URL: /blog/2026/02/11/react-native-0.84.md

- **React Native Comes to Meta Quest**: React Native has always focused on helping developers reuse knowledge across platforms. What started with Android and iOS has steadily expanded to Apple TV, Windows, macOS, and even the web with react-strict-dom. In 2021, the Many Platform Vision post outlined a future where React Native could adapt to new devices and form factors without fragmenting the ecosystem.
  - URL: /blog/2026/02/24/react-native-comes-to-meta-quest.md

## How to Use This Skill

Reference these resources when working with React Native · Learn once, write anywhere.