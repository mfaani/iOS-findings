Each Xcode build, creates a `.app` **folder**. 
The application _binary_ is inside that folder something like path/to/build/myCoolApp.app/myCoolApp
It's common for binaries to not have extensions. `ls`, `strings` are all binaries. You just call them you don't do something like `ls.exe`
For more on that see [What Files Go Into an Application Bundle?](https://developer.apple.com/library/archive/documentation/CoreFoundation/Conceptual/CFBundles/BundleTypes/BundleTypes.html#//apple_ref/doc/uid/10000123i-CH101-SW13)

### An `.ipa` != `.app` file. 

It's a zip file that contains your app and other bits and blobs used for delpoying the app to different places. Like this: 
<img width="804" alt="Screen Shot 2021-10-19 at 12 40 59 PM" src="https://user-images.githubusercontent.com/12160198/137954902-5e809d99-4460-47ac-97db-e4cdf8824a05.png">

You don't create a `.ipa` file unless you're archiving for app store and stuff. Normal runs from Xcode only create the `.app` folder. 
