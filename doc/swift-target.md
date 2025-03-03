# ANTLR4 Language Target, Runtime for Swift

## Performance Note

To use ANTLR4 Swift target in production environment, make sure to turn on compiler optimizations by following [these instructions](https://github.com/apple/swift-package-manager/blob/main/Documentation/Usage.md#setting-the-build-configuration) if you use SwiftPM to build your project. 

If you are using Xcode to build your project, it's unlikely you will not use `release` build for production build.

Conclusion is, you need to turn on `release` mode (which will have all the optimization pre configured for you) so the ANTLR4 Swift target can have reasonable parsing speed.

## Install ANTLR4

Make sure you have the ANTLR
installed. [The getting started guide](getting-started.md) should get
you started.

## Create a Swift lexer or parser 
This is pretty much the same as creating a Java lexer or parser, 
except you need to specify the language target, for example:

``` 
$ antlr4 -Dlanguage=Swift MyGrammar.g4 
``` 

If you integrate this as a build step inside Xcode, then you should use the
"gnu" message format to have any error messages parsed by Xcode.  You may
also want to use the `-o` option to put the autogenerated files in a
separate subdirectory.

```
antlr4 -Dlanguage=Swift -message-format gnu -o Autogen MyGrammar.g4
```

For a full list of antlr4 tool options, please visit the
[tool documentation page](tool-options.md).

## Build your Swift project with ANTLR runtime

### Note

We use __boot.py__ script located at the root of the Swift runtime folder
`antlr4/runtime/Swift` to provide additional support for both Xcode-based
projects and SPM-based projects. Below sections are organized for both of
the flavors. If you want to quickly get started, try:

```
python boot.py --help
```

for information about this script.

### Xcode Projects

Note that even if you are otherwise using ANTLR from a binary distribution,
you should compile the ANTLR Swift runtime from source, because the Swift
language does not yet have a stable ABI.

ANTLR uses Swift Package Manager to generate Xcode project files. 

#### Download source code for ANTLR

```
git clone https://github.com/antlr/antlr4
```

#### Generate Xcode project for ANTLR runtime

The `boot.py` script includes a wrapper around `swift package
generate-xcodeproj`. Use this to generate `Antlr4.xcodeproj` for the ANTLR
Swift runtime. (using _swift package generate-xcodeproj_ is not recommended)
since the project is dependent on some parser files generated by _boot.py_.

```
cd antlr4/runtime/Swift
python boot.py --gen-xcodeproj
```

#### Import ANTLR Swift runtime into your project

Open your own project in Xcode.

Open Finder in the `runtime/Swift` directory:

```
# From antlr4/runtime/Swift
open .
```

Drag `Antlr4.xcodeproj` into your project.

After this is done, your Xcode project navigator will be something like the
screenshot below. In this example, your own project is "Smalltalk", and you
will be able to see `Antlr4.xcodeproj` shown as a contained project.

<img src=images/xcodenav.png width="300">

#### Edit the build settings if necessary

Swift Package Manager currently does not support iOS, watchOS, or tvOS.  If
you wish to build for those platforms, you will need to alter the project
build settings manually.

#### Add generated parser and lexer to project

Make sure the parsers/lexers
generated in __step 2__ are added to the project. To do this, you can
drag the generated files from Finder to the Xcode IDE. Remember to
check __Copy items if needed__ to make sure the files are actually
moved into the project folder instead of symbolic links (see the
screenshot below). After moving you will be able to see your files in
the project navigator.  Make sure that the Target Membership settings
are correct for your project.

<img src=images/dragfile.png width="500">

#### Add the ANTLR Swift runtime as a dependency

Select your own project in Xcode and go to the Build Phases settings panel.
Add the ANTLR runtime under __Target Dependencies__ and __Link Binary With
Libraries__.

<img src=images/xcodedep.png width="800">

#### Build your project

The runtime and generated grammar should now build correctly.

### Swift Package Manager Projects

Add Antlr4 as a dependency to your `Package.swift` file. For more information, please see the [Swift Package Manager documentation](https://github.com/apple/swift-package-manager/tree/master/Documentation).


```swift
.package(name: "Antlr4", url: "https://github.com/antlr/antlr4", from: "4.10"
```
## Swift access levels

You may use the `accessLevel` option to control the access levels on generated
code.  This option can either be specified with `-DaccessLevel=value` on
the `antlr4` command line, or inside your `.g4` file like this:

```
options {
    accessLevel = 'value';
}
```

By default (with the `accessLevel` option unspecified) the generated code
uses the following access levels:

* `open` for anything that you can feasibly extend with subclassing:
the generated parser, lexer, and context classes, the the listener and
visitor base classes, and all their accessor and setter functions.
* `public` for anything that should not be subclassed, but otherwise is
useful to client code: protocols, initializers, and static definitions such
as the lexer tokens, symbol names, and so on.
* `internal` or `private` for anything that should not be accessed directly.

If you specify `accessLevel = 'public'` then all items that are `open` by
default will use `public` instead.  Otherwise, the behavior is the same as
the default.

If you specify `accessLevel = ''` or `accessLevel='internal'` then all items
that are `open` or `public` by default will use Swift's default (internal)
access level instead.

Those are the only supported values for `accessLevel` when using the Swift
code-generator.

We recommend using `accessLevel = ''`.  Even if you are creating a parser
as part of a library, you would usually want to wrap it in an API of your
own and keep the ANTLR-generated parser internal to your module.  You
only need to use the less restrictive access levels if you need to expose
the parser directly as part of your own module's API.
