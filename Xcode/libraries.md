Discussion with **Apple Engineer** during labs 2022: 

Difference of Framework, Dynamic Library and Static library: 
Framework bundles of chunck of code + series of assets (icons, xibs). Framework assume you're using Dynamic Libraries
Static Libraries: don't use Framework so you have to manually copy resources instead of embedding
Framework: are packaged. Static libraries you have manually copy xibs, when you copy one will override i.e. if similar symboles exist, they'll override one another. 
with Framework, the whole framework will copied into your app bundle + subdirectores. Example you'll have something like:
Framework1/LB + resources/library.dylib
Framework2/LB + resources/library.dylib

-----

To see what dynamic libraries you've added: 
```
otool -L path/to/binary
```
Example result: 

```ruby

/usr/lib/libicucore.A.dylib (compatibility version 1.0.0, current version 68.2.0)
/usr/lib/libsqlite3.dylib (compatibility version 9.0.0, current version 329.0.0)
/usr/lib/libz.1.dylib (compatibility version 1.0.0, current version 1.2.11)
@rpath/AIQ.framework/AIQ (compatibility version 1.0.0, current version 1.0.0) // FOR EACH POD WE USE...
@rpath/IoT.framework/IoT (compatibility version 1.0.0, current version 1.0.0)
/System/Library/Frameworks/CFNetwork.framework/CFNetwork (compatibility version 1.0.0, current version 1312.0.0)
/usr/lib/libobjc.A.dylib (compatibility version 1.0.0, current version 228.0.0)
/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1311.0.0)
/usr/lib/swift/libswiftAVFoundation.dylib (compatibility version 1.0.0, current version 2036.25.1, weak)
/usr/lib/swift/libswiftAccelerate.dylib (compatibility version 1.0.0, current version 27.0.0, weak)
/usr/lib/swift/libswiftCallKit.dylib (compatibility version 1.0.0, current version 3.0.0, weak)

```


- If the framework is bundled inside your signed application binary. you can dynamically load it
- If the framework is downloaded into ANY folder that invalidates the signature of your app, then you can not dynamically load it. (This enforcement began in iOS 11)

## Dynamic Library vs Static Library
https://www.vadimbulavin.com/static-dynamic-frameworks-and-libraries/
