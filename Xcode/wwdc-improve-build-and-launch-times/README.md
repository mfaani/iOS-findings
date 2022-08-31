Link: https://developer.apple.com/videos/play/wwdc2022/110362/

Static library does selective loading of functions


it’s amazing how it iteratively shows how the linker works
tldr
main loads foo.
foo needs bar
bar needs baz
baz doesn’t need anything
so even though you had shaz among the static libraries, shaz doesn’t get loaded into the app. It’s iterative but also smart to not load unused stuff…

<img width="1440" alt="Screen Shot 2022-06-10 at 2 14 22 PM" src="https://user-images.githubusercontent.com/12160198/187778939-886ceb1a-902c-4b06-a63c-7c84011a3982.png">
<img width="1440" alt="Screen Shot 2022-06-10 at 2 14 32 PM" src="https://user-images.githubusercontent.com/12160198/187778943-35b9327d-6038-46d4-8101-87d4156755e4.png">
<img width="1440" alt="Screen Shot 2022-06-10 at 2 14 25 PM" src="https://user-images.githubusercontent.com/12160198/187778946-d6a9d719-b392-4f07-94aa-aa36431c3f9b.png">


This year linker is 2X faster, because it use cores better…

### When to use Static library:
static library is for code not actively changing…
it slows down the linker because entire static library has to be rebuilt + table of contents. It’s just extra IO
use SL only for stable code i.e. code that doesn’t change much…
But a downside of that is that it slows down the linker. That is because to make builds reproducible and follow traditional static library semantics, the linker has to process static libraries in a fixed, serial order. That means some of the parallelization wins of ld64 cannot be used with static libraries. But if you don’t really need this historical behavior, you can use a linker option to speed up your build. That linker option is called “all load”. It tells the linker to blindly load all .o files from all static libraries. This is helpful if your app is going to wind up selectively loading most of the content from all the static libraries anyways. Using -all_load will allow the linker to parse all the static libraries and their content in parallel.
tldr if you know you’re going to load all, then just tell it to linker so it won’t do things iteratively… (edited) 

### Problems with using `-all_load`:
But if your app does clever tricks where it has multiple static libraries implementing the same symbols, and depends on the command line order of the static libraries to drive which implementation is used, then this option is not for you. Because the linker will load all the implementations and not necessarily get the symbol semantics that were found in regular static linking mode. The other downside of -all_load is that it may make your program bigger because “unused” code is now being added in.
tldr it’s not too smart to resolve multiple libraries with identical symbols +
you’d ultimately add extra files. You can -dead_strip (along with -all_load)to remove unused ones. But profile this to make sure things work as you expect… (edited) 


`-no_exported_symbols flag`:
`-no-deduplicate flag`:
…


### Static library gotchas



### Dynamic libraries:
1990s
Static library had scaling problems. Static link time would take long…
Dynamic libraries == dylibs == dll == dso



Instead of copying code, the linker just records some kind of a promise --> your code is now: your code + some promises (not the libraries anymore). The virtual memory system can re-use the same memory that used shared dylibs…



### Costs of Dynamic libraries:
slower launch time. you deferred linking time from build time to launch time…
more dirty memory. A dynamic library based program will have more dirty pages. In the static library case, the linker would co-locate all globals from all static libraries into the same DATA pages in the main executable. But with dylibs, each library has its DATA page
Extra cost because of needing a dynamic linker, because you made promises, and need to fulfill them…
(edited)


?????
### How DL works on run time:
An executable binary is divided up into segments, usually at least TEXT, DATA, and LINKEDIT. Segments are always a multiple of the page size for the OS. Each segment has a different permission (edited)
<img width="414" alt="Screen Shot 2022-06-10 at 3 08 42 PM" src="https://user-images.githubusercontent.com/12160198/187779043-20771e16-62a7-4315-8355-ff310e4ffbc0.png">


### DL best practices:
Use less fewer dylibs
don’t do IO or networking in a static initializer.
Too many SL and your build debug cycles times are slow
Too many DL and your launch time is slow
due to improvements, you can use more static libraries… 


`dyld_usage` shows in the screenshot below the time it takes for your static initializers…
Though mainly it’s for apply fixups (page and linking)

<img width="705" alt="Screen Shot 2022-06-10 at 3 13 37 PM" src="https://user-images.githubusercontent.com/12160198/187779129-d7c0e987-9272-4a90-81be-0bf0b12a0fdb.png">


`dyld_info`
to inspect binaries (in disk and on dyld cache)


## Meta discussion on this afterwards: 

Ignoring performance and build vs launch time, and only in terms of packaging, how are the Static linking (SL) and Dynamic Linking (DL) different?
I recall being told that with SL, when you copy one will override i.e. if similar symbols exist.
However with DL similar symbols can exist because they’re all bundled
Can some shed some light onto that? 



K: With static linking, statically-linked binaries will have their objects copied into the target binary directly. There are various benefits to that (one being that the linker can see which objects are and aren’t used by the target binary, and can skip copying unused bits — saving space). But yes, duplicate symbol names will clash.
With dynamic linking, the dynamically-linked binaries will be shipped alongside the target binary, or will be pre-installed on the system. That’s why you don’t need to ship the Swift dylibs will apps any more — they’re preinstalled into the operating system. Since resolving is done at runtime, the compiler can’t strip unused parts out since it has no idea. As long as you have a language (i.e., not Objective-C) that separates symbols by namespace/whatever, duplicate symbol names won’t clash. ObjC doesn’t have this, so duplicates will clash at runtime rather than compiletime.
This is a very basic overview — Googling “static vs dynamic linking” will get you a bunch of articles on it
That said, Swift does module namespacing when statically linked as well, so symbol name clashing is more of a (language + link type)-specific behaviour than purely based on how linking is done


M:so if
`Foo.app` statically links in the following order
Bar.lib
Baz.lib
and both libraries have `class Panda` only one Baz.lib implementation (because it was loaded last) will get used? Won’t this cause the app to not compile?
Or by symbol you mean class `Panda.specificFuncWithSpecificSignature` <-- This makes more sense i.e. there won’t be compile time crashes. But this will create very undefined behavior. Right?


B: The behavior will differ if that's a Swift vs ObjC vs @objc Swift class
It also matters how the code came to be in the static libraries. Are they copies of the same static library? Are they the result of compiling two different Swift modules (see  K’s comments about module name spacing)?


K: I can’t remember if attempting to static link a duplicate symbol into a binary is a link failure or not in ObjC/C


K: In ObjC with dynamic linking you’ll get a “Class <x> is present in both x.framework and y.framework — the one that’ll be used is undefined” warning at runtime

M:so overriding won’t happen in pure Swift SL?
K: B said: "It also matters how the code came to be in the static libraries. Are they copies of the same static library? Are they the result of compiling two different Swift modules (see 
K’s comments about module name spacing)?

  M: Thanks. I’m trying to validate what I’ve inferred. Often I infer incorrectly :shrug: (edited) 


K: We’re missing a bunch of information to be able to answer your question. So I’ll fill in some blanks:
Assuming Swift
Assuming Bar.lib comes from a project that generates a module called Bar
Assuming Baz.lib comes from a project that generates a module called Baz
Assuming you’re not using any @objc declarations
Assuming they both have a class called Panda
… then no, there’s no clash. Due to module namespacing, the classes are Bar.Panda and Baz.Panda (well, not quite due to Swift mangling, but meh) and therefore there’s no clash to cause a problem. (edited) 
11:35
Dynamic or static linking isn’t really relevant, because there’s no name clash
  
---
  
B: 
So Obj-C in general is an issue as the Objective-C runtime uses what is effectively a flat namespace, which is to say that it only considers the type name and not the module name or dylib name or binary name it is loaded from. 
So if you have an Obj-C class named Foo defined in two separate dynamic libraries (or your app and a dylib) you’ll get errors at runtime from the Obj-C runtime about the duplicate class definitions. 
This also applies to Swift `@objc` classes, since they’re also Obj-C classes, but by default `@objc` classes have their Objective-C name be qualified with the module name (`ModuleA.Foo`), rather than just being a bare identifier. 
If you do `@objc(Foo) class Foo { … }` in Swift you’re back to having the same issues with a plain Obj-C class
  
W: (This is why name prefixes are a thing in ObjC)
