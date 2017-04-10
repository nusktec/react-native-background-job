# react-native-background-job [![npm version](https://badge.fury.io/js/react-native-background-job.svg)](https://badge.fury.io/js/react-native-background-job)

Schedule background jobs that run your JavaScript when your app is in the background. 

The jobs will run even if the app has been closed and, by default, also persists over restarts.

This library relies on React Native's [`HeadlessJS`](https://facebook.github.io/react-native/docs/headless-js-android.html) which is currently only supported on Android.

On the native side it uses [`JobScheduler`](https://developer.android.com/reference/android/app/job/JobScheduler.html) which means that the jobs can't be scheduled exactly and for Android 23+ they fire at most once per 15 minutes +-5 minutes. `JobSceduler` was used since it seemed to be the most battery efficient way of scheduling background tasks. I'm open to pull requests that implement more exact scheduling.

## Requirements

-   RN 0.36+

## Supported platforms

-   Android (API 21+)

Want iOS? Go in and vote for Headless JS to be implemented for iOS: [Product pains](https://productpains.com/post/react-native/headless-js-for-ios)

## Getting started

`$ yarn add react-native-background-job`

or

`$ npm install react-native-background-job --save`

### Mostly automatic installation

`$ react-native link react-native-background-job`

### Manual installation

<!--
#### iOS

1. In XCode, in the project navigator, right click `Libraries` ➜ `Add Files to [your project's name]`
2. Go to `node_modules` ➜ `react-native-background-job` and add `RNBackgroundJob.xcodeproj`
3. In XCode, in the project navigator, select your project. Add `libRNBackgroundJob.a` to your project's `Build Phases` ➜ `Link Binary With Libraries`
4. Run your project (`Cmd+R`)<

-->

#### Android

1.  Open up `android/app/src/main/java/[...]/MainApplication.java`
    -   Add `import com.pilloxa.backgroundjob.BackgroundJobPackage;` to the imports at the top of the file
    -   Add `new BackgroundJobPackage()` to the list returned by the `getPackages()` method
2.  Append the following lines to `android/settings.gradle`:


            include ':react-native-background-job'
            project(':react-native-background-job').projectDir = new File(rootProject.projectDir, 	'../node_modules/react-native-background-job/android')

3.  Insert the following lines inside the dependencies block in `android/app/build.gradle` and bump the minSdkVersion to 21:


              compile project(':react-native-background-job')

## Usage

The jobs have to be registered each time React Native starts, this is done using the `register` function. Since HeadlessJS does not mount any components the `register` function must be run outside of any class definitions (see [example/index.android.js](https://github.com/vikeri/react-native-background-job/blob/8b8fdb2cb4bc0907eb16a54204c85c3b7a60dfa4/example/index.android.js#L20-L23))

Registering the job does not mean that the job is scheduled, it just informs React Native that this `job` function should be tied to this `jobKey`. The job is then scheduled using the `schedule` function. **The job will not fire while the app is in the foreground**. This is since the job is run on the only JavaScript thread and if running the job when app is in the foreground it would freeze the app.

For a full example check out [example/index.android.js](example/index.android.js)

## API

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

### register

Registers jobs and the functions they should run. 

This has to run on each initialization of React Native. Only doing this will not start running the job. It has to be scheduled by `schedule` to start running.

**Parameters**

-   `obj` **[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)** 
    -   `obj.jobKey` **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** A unique key for the job
    -   `obj.job` **[function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function)** The JS-function that will be run

**Examples**

```javascript
import BackgroundJob from 'react-native-background-job';

const backgroundJob = {
 jobKey: "myJob",
 job: () => console.log("Running in background")
};

BackgroundJob.register(backgroundJob);
```

### schedule

Schedules a new job. 

This only has to be run once while `register` has to be run on each initialization of React Native.

**Parameters**

-   `obj` **[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)** 
    -   `obj.jobKey` **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** A unique key for the job
    -   `obj.timeout` **[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number)** How long the JS job may run before being terminated by Android (in ms).
    -   `obj.period` **[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number)?** The frequency to run the job with (in ms). This number is not exact, Android may modify it to save batteries. Note: For Android > N, the minimum is 900 0000 (15 min). (optional, default `900000`)
    -   `obj.persist` **[boolean](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** If the job should persist over a device restart. (optional, default `true`)
    -   `obj.warn` **[boolean](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** If a warning should be raised if overwriting a job that was already scheduled. (optional, default `true`)
    -   `obj.networkType` **[number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number)?** Only run for specific network requirements, (not respected by pre Android N devices) [docs](https://developer.android.com/reference/android/app/job/JobInfo.html#NETWORK_TYPE_ANY) (optional, default `BackgroundJob.NETWORK_TYPE_NONE`)
    -   `obj.requiresCharging` **[boolean](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** Only run job when device is charging, (not respected by pre Android N devices) [docs](https://developer.android.com/reference/android/app/job/JobInfo.Builder.html#setRequiresCharging(boolean)) (optional, default `false`)
    -   `obj.requiresDeviceIdle` **[boolean](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** Only run job when the device is idle, (not respected by pre Android N devices) [docs](https://developer.android.com/reference/android/app/job/JobInfo.Builder.html#setRequiresDeviceIdle(boolean)) (optional, default `false`)

**Examples**

```javascript
import BackgroundJob from 'react-native-background-job';

const backgroundJob = {
 jobKey: "myJob",
 job: () => console.log("Running in background")
};

BackgroundJob.register(backgroundJob);

var backgroundSchedule = {
 jobKey: "myJob",
 timeout: 5000
}

BackgroundJob.schedule(backgroundSchedule);
```

### getAll

Fetches all the currently scheduled jobs

**Parameters**

-   `obj` **[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)** 
    -   `obj.callback` **function ([Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array))** A list of all the scheduled jobs will be passed to the callback

**Examples**

```javascript
import BackgroundJob from 'react-native-background-job';

BackgroundJob.getAll({callback: (jobs) => console.log("Jobs:",jobs)});
```

### cancel

Cancel a specific job

**Parameters**

-   `obj` **[Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object)** 
    -   `obj.jobKey` **[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)** The unique key for the job
    -   `obj.warn` **[boolean](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean)?** If one tries to cancel a job that has not been scheduled it will warn (optional, default `true`)

**Examples**

```javascript
import BackgroundJob from 'react-native-background-job';

BackgroundJob.cancel({jobKey: 'myJob'});
```

### cancelAll

Cancels all the scheduled jobs

**Examples**

```javascript
import BackgroundJob from 'react-native-background-job';

BackgroundJob.cancelAll();
```

### setGlobalWarnings

Sets the global warning level

**Parameters**

-   `warn` **[boolean](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 

**Examples**

```javascript
import BackgroundJob from 'react-native-background-job';

BackgroundJob.setGlobalWarnings(false);
```

## Debugging

If you are using Android API +25 you can manually trigger the jobs by using the following command in a terminal:

```sh
$ adb shell cmd jobscheduler run -f your.package jobIntId
```

`jobIntId`: is the hashed `jobKey`. Get that value by going to [Java REPL](http://www.javarepl.com/term.html) and input:

```java
"yourJobKey".hashCode();
// 1298333557
```

## Troubleshooting

### `No task registered for key myJob`

Make sure you call the `register` function at the global scope (i.e. not in any component life cycle methods (render, iDidMount etc)). Since the components are not rendered in Headless mode if you run the register function there it will not run in the background and hence the library will not find which function to run.

See [example project](https://github.com/vikeri/react-native-background-job/blob/c0e4bc8e9dd692695169d6c9855d39e2ff917a61/example/index.android.js#L20-L25)


### `AppState.currentState` is `"active"` when I'm running my Headless task in the background

This is a [React Native issue](https://github.com/facebook/react-native/issues/11561), you can get around it by calling `NativeModules.AppState.getCurrentAppState` directly instead.

### My job always runs in the background even if i specified `requiresCharging`, `requiresDeviceIdle` or a specific `networkType`

This is an [Android issue](https://code.google.com/p/android/issues/detail?id=81265), it seems that you can not have these restrictions at the same time as you have a periodic interval for pre Android N devices.

## Sponsored by

[![pilloxa](http://pilloxa.com/images/pilloxa-round-logo.svg)](http://pilloxa.com)
