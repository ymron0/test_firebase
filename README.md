# test_firebase

## Initial setup
In the firebase console, I create a new project called `test-firebase`, which was assigned the id `test-firebase-a7964` by your system. During creation, I enabled Google Analytics.

Once the project creation is done, I create a new app for Android. I use the Android package name `ch.ymron.test_firebase`. I do not download `google-services.json`, because flutterfire will set it up for me later. I do not follow the instructions to setup the Firebase SDK in build.gradle, because the [documentation](https://firebase.google.com/docs/flutter/setup?authuser=0&hl=en&platform=android) does not tell me to do that anywhere.

I initiate my project locally with the command `flutter create test_firebase --org ch.ymron --project-name test_firebase --platforms android`. I cleanup the README and I remove the comments from `lib/main.dart` and I commit the initial project to Git.

Following [this documentation](https://firebase.google.com/docs/flutter/setup?authuser=0&hl=en&platform=android), I successively run the following two commands:
```
flutter pub add firebase_core
flutterfire configure
```
I modify `main()` so that it looks like that:
```
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );

  runApp(const MyApp());
}
```
And then I run my app for the first time to verify that everything works well.

My debug console at this point is absolutely normal:
```
D/CompatibilityChangeReporter(26234): Compat change id reported: 183155436; UID 10197; state: ENABLED
I/FirebaseApp(26234): Device unlocked: initializing all Firebase APIs for app [DEFAULT]
```

## Add Firebase Analytics
Now it's time to implement Google Analytics according to [this documentation](https://firebase.google.com/docs/analytics/get-started?platform=flutter&authuser=0).
I run `flutter pub add firebase_analytics` and, just to be sure, I also run `flutterfire configure` even though the documentation doesn't tell me to do so. I add those two lines to my `main()`:
```
FirebaseAnalytics analytics = FirebaseAnalytics.instance;
await analytics.logEvent(name: 'App_is_starting');
```

Now my debug console outputs the `Missing google_app_id` error:
```
D/CompatibilityChangeReporter(26515): Compat change id reported: 183155436; UID 10197; state: ENABLED
I/FirebaseApp(26515): Device unlocked: initializing all Firebase APIs for app [DEFAULT]
E/FA      (26515): Missing google_app_id. Firebase Analytics disabled. See https://goo.gl/NAOOOI
```

All changes to this point have been commited to the git. I have followed the documentation, and I fail to see any reference to this error in the documentation.

I'd like to point out that my two `build.gradle` files have not been modified by `flutterfire configure`. Some people report online that these files are modified automatically. I also found an [example](https://github.com/OpenSphereSoftware/FlutterMadeEasy_ZeroToMastery/blob/main/5_todo_app/android/app/build.gradle) where such automatic modification occured. 

If I look at the Firebase console, I see that 0 user has used my app.

## Add Crashlytics
I try to add [Crashlytics](https://firebase.google.com/docs/crashlytics/get-started?platform=flutter#add-sdk) to my app to confirm that the app can successfully talk with Firebase. So I run `flutter pub add firebase_crashlytics` and then `flutterfire configure`.

I add the following lines (from your doc) to my `main()`:
```
FlutterError.onError = (errorDetails) {
  FirebaseCrashlytics.instance.recordFlutterFatalError(errorDetails);
};
PlatformDispatcher.instance.onError = (error, stack) {
  FirebaseCrashlytics.instance.recordError(error, stack, fatal: true);
  return true;
};
```

After updating the `onPressed()` method of my FAB so that it throws an exception, I re-run the app.

Now my debug console looks like that:
```
D/CompatibilityChangeReporter(26699): Compat change id reported: 183155436; UID 10197; state: ENABLED
I/FirebaseApp(26699): Device unlocked: initializing all Firebase APIs for app [DEFAULT]
I/FirebaseCrashlytics(26699): Initializing Firebase Crashlytics 18.4.3 for ch.ymron.test_firebase
D/CompatibilityChangeReporter(26699): Compat change id reported: 247079863; UID 10197; state: DISABLED
D/FirebaseSessions(26699): Registering Sessions SDK subscriber with name: CRASHLYTICS, data collection enabled: true
D/FirebaseSessions(26699): Data Collection is enabled for at least one Subscriber
D/TrafficStats(26699): tagSocket(123) with statsTag=0xffffffff, statsUid=-1
D/TrafficStats(26699): tagSocket(122) with statsTag=0xffffffff, statsUid=-1
D/TrafficStats(26699): tagSocket(124) with statsTag=0x8001, statsUid=-1
E/FA      (26699): Missing google_app_id. Firebase Analytics disabled. See https://goo.gl/NAOOOI
D/FirebaseSessions(26699): Sessions SDK disabled. Events will not be sent.
I/FirebaseCrashlytics(26699): No version control information found
```

I still have the `Missing google_app_id` (obviously), but I also find some other weird things that may indicate that something does not work correctly:
```
D/FirebaseSessions(26699): Sessions SDK disabled. Events will not be sent.
I/FirebaseCrashlytics(26699): No version control information found
```
After triggering some exceptions, I see this in my log, which shows that Crashlytics tries unsuccessfully to talk with Analytics:
```
W/FirebaseCrashlytics(26699): Timeout exceeded while awaiting app exception callback from Analytics listener.
I/TRuntime.CctTransportBackend(26699): Status Code: 200
```

But in my Crashlytics console, I can see the errors. So that means that Crashlytics works!

But I point out that my `build.gradle` files have, once again, not been modified. Not sure if that's normal. I get the same with `firebase_firestore` and `firebase_auth`. They work but `flutterfire configure` never modifies `build.gradle`.

## Troubleshooting
After communication with Angela from Firebase, here is what I did:
The app-level `build.gradle` file was updated with:
```
dependencies {
    implementation(platform("com.google.firebase:firebase-bom:32.3.1"))
    implementation("com.google.firebase:firebase-analytics")
}
```
The `AndroidManifest.xml` was upgraded with `<meta-data android:name="firebase_analytics_collection_enabled" android:value="true" />`. Before that, there was no such line.

Verbose log was tested. The result is [here](/verbose_log.md).

Debug view was tested. The result is [here](/debug_view_log.md).

The firebase packages at up to date.