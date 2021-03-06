##Prerequisites
- iOS 8.0 or higher

##Frameworks
#####Add the frameworks below to your project:
- CoreLocation.framework
- Foundation.framework
- UIKit.framework
- QuartzCore.framework
- CoreGraphics.framework
- SystemConfiguration.framework
- libz.dylib
- CoreTelephony.framework
- AdSupport.framework

#####Property list file
Add the properties below to your project plist file:
- NSLocationAlwaysUsageDescription: Add any value, for example "Location Services"

#####Capabilities
Add the Background Modes below to your target:
- Remote notifications

##Set up the SDK

###Cocoapods  
Add the following line to your project's Podfile:
```objective-c
    pod 'Findrix'
```

###Manual￼￼
1. Download the Findrix.framework SDK for iOS. 
2. Drag the Findrix.framework file to your Xcode project and place it under the Frameworks folder.

##Initialization
```objective-c
#import <Findrix/Findrix.h>

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    [Findrix start:@"userId" clientId:@"clientId" clientSecret:@"clientSecret" photo:devicePhoto callback:^(NSString *deviceId, NSError *error) {
        
        if(error == nil) NSLog(@"Device created %@", deviceId);
        else NSLog(@"Error creating device %@", error.description);
        
      }];
    ];
    
}

- (void)applicationDidBecomeActive:(UIApplication *)application{

    [Findrix activate];
    
}
```

#####Parameters:
- userId – Provide an ID for this device(can be null or empty) 
- clientId – Request a client id with Findrix dev team
- clientSecret – Request a client secret with Findrix dev team 
- devicePhoto - Can be null of empty
- callback - Device creation callback

IMPORTANT: Don't forget to call [Findrix activate]!

##Push notifications
The first step is to create an App ID and the associated SSL certificate on the Apple Developer website. This certificate will allow the Parse server to send push notifications to the application identified by the App ID. If you already created the App ID and the associated SSL certificate, send the p12 file to Findrix dev team(Section Configuring your App ID for Development Push Notifications): 

######Generating a Certificate Request
To begin, we'll need a certificate signing request file. This will be used to authenticate the creation of the SSL certificate.

1. Launch the Keychain Access application on your Mac.
2. Select the menu item Keychain Access > Certificate Assistant > Request a
Certificate From a Certificate Authority...
3. Enter your email address and name.
4. Select "Saved to disk" to download the .certSigningRequest file to your desktop.

######Creating an App ID
Every iOS application installed on your developer device needs an App ID. As a convention, these are represented by reversed addresses (ex. com.example.MyParsePushApp). For this
￼￼push app, you can use an App ID you've already created, but make sure it does not contain a wildcard character ("*"). The following instructions cover the creation of a new App ID.

1. Navigate to the Apple Developer Member Center(https://developer.apple.com/membercenter/index.action) website, and select Certificates, Identifiers & Profiles.
2. Select Identifiers from the iOS Apps section.
3. You will see a list of your iOS App IDs. Select the + button to register a new App Id.
4. Enter a name for your new App ID, then make sure to select the checkbox next to Push Notifications under App Services.
5. Choose an App ID Prefix. The default selection should be correct in most cases.
6. Under App ID Suffix, select Explicit App ID. Enter your iOS app's Bundle ID. This
string should match the Bundle Identifier in your iOS app's Info.plist.
7. Select "Continue" and make sure that all the values were entered correctly. Push Notifications should be enabled, and the Identifier field should match your app's Bundle Identifier (plus App ID Prefix). Select "Submit" to finalize the registration of your new App ID.

######Configuring your App ID for Development Push Notifications
Now that you've created an App ID (or chosen an existing Explicit App ID), it's time to configure the App ID for Push Notifications.

1. Select your newly created App ID from the list of iOS App IDs, then select "Settings".
2. Scroll down to the Push Notifications section. Here you will be able to create both a
Development SSL Certificate, as well as a Production SSL Certificate. Start by
selecting "Create Certificate" under "Development SSL Certificate".
3. The next screen will show instructions for creating a Certificate Signing Request
(CSR). This is the same .certSigningRequest file you created earlier. Select "Continue", then select "Choose File..." and locate the .certSigningRequest you created.
4. Select "Generate". Once the certificate is ready, select "Done" and download the generated SSL certificate from the "iOS App ID Settings" screen.
5. Double click on the downloaded SSL certificate to install it in your Keychain.
6. In Keychain Access, under "My Certificates", find the certificate you just added. It
should be called "Apple Development/Production IOS Push Services.
7. Right-click on it, select "Export Apple Development IOS Push Services:...", and save
it as a .p12 file. You will be prompted to enter a password which will be used to protect the exported certificate. Enter the password "1234". Note that you might have to enter your OS X password to allow Keychain Access to export the certificate from your keychain

######Creating the Provisioning Profile
A Provisioning Profile authenticates your device to run the app you are developing. Whether you have created a new App ID or modified an existing one, you will need to regenerate your provisioning profile and install it. If you have trouble using an existing profile, try removing the App ID and setting it back. For this tutorial, we'll create a new one.

1. Navigate to the Apple Developer Member Center(https://developer.apple.com/membercenter/index.action) website, and select Certificates, Identifiers & Profiles.
2. Select Provisioning Profiles from the iOS Apps section
3. Select the + button to create a new iOS Provisioning Profile.
4. Choose "iOS App Development" as your provisioning profile type then select
"Continue.
5. Choose the App ID you created from the drop down then select "Continue".
6. Make sure to select your iOS Development/Production certificate in the next screen,
then select "Continue".
7. You will be asked to select which devices will be included in the provisioning profile.
Select "Continue" after selecting the devices you will be testing with.
8. Choose a name for this provisioning profile, such as "My Push App Profile", then
select "Generate".
9. Download the generated provisioning profile from the next screen by selecting the
"Download" button.
10. Install the profile by double-clicking on the downloaded file.

######Registering the device
Now you must register the current device for push. If you already register your device for push notifications, go to step 3:

- Change the AppDelegate.h:
```objective-c
#import <UserNotifications/UserNotifications.h>
@interface TRLSAppDelegate : UIResponder <UIApplicationDelegate, UNUserNotificationCenterDelegate>
```

- Add the macros below:
```objective-c
#define SYSTEM_VERSION_GREATER_THAN_OR_EQUAL_TO(v)  ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] != NSOrderedAscending)
#define SYSTEM_VERSION_LESS_THAN(v)                 ([[[UIDevice currentDevice] systemVersion] compare:v options:NSNumericSearch] == NSOrderedAscending)
```

-  Create the method registerDeviceForPushNotifications in the app delegate: 
```objective-c
- (void)registerDeviceForPushNotifications{
  if( SYSTEM_VERSION_LESS_THAN( @"10.0" ) )
    {
        [[UIApplication sharedApplication] registerUserNotificationSettings:[UIUserNotificationSettings settingsForTypes:(UIUserNotificationTypeSound | UIUserNotificationTypeAlert | UIUserNotificationTypeBadge) categories:nil]];
        [[UIApplication sharedApplication] registerForRemoteNotifications];

    }
    else
    {
        UNUserNotificationCenter *center = [UNUserNotificationCenter currentNotificationCenter];
        center.delegate = self;
        [center requestAuthorizationWithOptions:(UNAuthorizationOptionSound | UNAuthorizationOptionAlert | UNAuthorizationOptionBadge) completionHandler:^(BOOL granted, NSError * _Nullable error)
         {
             if( !error )
             {
                 [[UIApplication sharedApplication] registerForRemoteNotifications];
             }
         }];  
    } 
}
```
-  Call the method registerDeviceForPushNotifications before Findrix initialization in the app delegate’s application:didFinishLaunchingWithOptions: method:
```objective-c
- (BOOL)application:(UIApplication *)application
  didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    
    [self registerDeviceForPushNotifications];

}
```

Implement the callback method - application:didRegisterForRemoteNotificationsWithDeviceToken: in the app delegate. We need to inform Findrix about this new device:
```objective-c
- (void)application:(UIApplication *)app didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
  
    [Findrix setDeviceToken:deviceToken];
   
}
```
To handle a notification, implement the methods below in the app delegate:
```objective-c
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler{
    
    [Findrix applicationDidReceiveRemoteNotification:userInfo];
    
}

- (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler{
    
    [Findrix applicationDidReceiveRemoteNotificationWithNotification:notification];
    
}

- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void (^)())completionHandler{
    
    [Findrix applicationDidReceiveRemoteNotificationWithNotification:response.notification];

}
```
##Customize the UI for Your App
Once you've set up your library integration, you can customize the UI to match the color scheme and branding of your app to ensure the best user experience for your users.

Import the class 
```objective-c
#import <Findrix/FDXMessageAppearance.h>
```
You can customize the following properties
```objective-c
    // Colors
    [FDXMessageAppearance setTextColor:[UIColor blueColor]];
    [FDXMessageAppearance setBackgroundColor:[UIColor grayColor]];
    [FDXMessageAppearance setButtonBackgroundColor:[UIColor yellowColor]];
    [FDXMessageAppearance setButtonTextColor:[UIColor greenColor]];
    [FDXMessageAppearance setTitleColor:[UIColor blackColor]];
    
    //Fonts
    [FDXMessageAppearance setTitleFont:[UIFont systemFontOfSize:9.0]];
    [FDXMessageAppearance setButtonTextFont:[UIFont boldSystemFontOfSize:18.0]];
    [FDXMessageAppearance setTextFont:[UIFont italicSystemFontOfSize:12.0]];
```


