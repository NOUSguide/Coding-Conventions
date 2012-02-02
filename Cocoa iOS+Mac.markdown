# Objective-C Coding Convention and Best Practices

Most of these guidelines are to match Apple's documentation and community-accepted best practices. This document is mainly targeted towards iOS development, but applies to Mac as well.

## Usage

These coding guidelines shall be applied for all new iOS and Mac projects, converting old projects is possible but not recommended in general.

## Warnings

Warnings in your own code should be fixed in general. Sometimes it is not possible to remove a warning but it is clear that to the developer that there is no problem with the underlying code. In this cases it is possible to silence a warning with pragma comments. This should be used **very rarely** and should always be documented.

```objective-c
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Warc-performSelector-leaks"

// Code that generates a warning but really shouldn't
// ...

#pragma clang diagnostic pop
```

Another option, which again should be rarely used is to silence warnings in a file completely. This should only be done on files of third-party-libraries which won't get changed. For this one has to specify the option `-w` as Compiler Flag for a file (Target - Build Phases - Compile Sources).

## TODOs

Todos should be marked as follows as should contain a brief description of what there is to do:

```objective-c
 // TODO: Fix the Bug XYZ that happens when the user clicks on the second button while the App is loading
```

Here's a handy script which can be added as a Build Phase (Target - Build Phases - Add Build Phase - Run Script) to your project that automatically generates a warning out of each TODO-Comment:

```bash
 KEYWORDS="TODO:|FIXME:|\?\?\?:|\!\!\!:"
find "${SRCROOT}" \( -name "*.h" -or -name "*.m" \) -print0 | xargs -0 egrep --with-filename --line-number --only-matching "($KEYWORDS).*\$" | perl -p -e "s/($KEYWORDS)/ warning: \$1/"
```

## Static Analyzer

The Clang static analyzer should be used early and often, as it can detect various code smells and helps finding those.

## Comments

One-line comments `//` are preferred to multiline `/**/` comments. Comments should be used to document **why** something is happening and not what is happening and should be used to explain situations that are not trivial.

For documentation AppleDoc is used: http://www.gentlebytes.com/home/appledocapp/

## Operators

```objective-c
NSString *foo = @"bar";
NSInteger answer = 42;
answer += 9;
answer++;
answer = 40 + 2;
```

The `++`, `--`, etc should **always** be after the variable instead of before to be consistent with other operators. Operators separated should be surrounded by spaces unless there is no right operand.


## Types

`NSInteger` and `NSUInteger` should be used instead of `int`, `long`, etc per Apple's best practices and 64-bit safety. `CGFloat` is preferred over `float` for the same reasons. This future proofs code for 64-bit platforms.

All Apple types should be used over primitive ones. For example, if you are working with time intervals, use `NSTimeInterval` instead of `double` even though it is synonymous. This is considered best practice and makes for clearer code.

## Spacing
Code shall be indented by 4 spaces. If you want to wrap a long line of code then make sure it is colon aligned per Xcode’s formatting. Keywords such as if, while, do, switch, etc. should have a space after them. Wasted vertical space should be avoided, there should not be more than one empty line.

```objective-c
// long method name calling convention
color = [NSColor colorWithCalibratedHue:0.10
                             saturation:0.82
                             brightness:0.89
                                  alpha:1.00];
```

## Methods

```objective-c
- (void)someMethod {
    // Do stuff
}


- (NSString *)stringByReplacingOccurrencesOfString:(NSString *)target withString:(NSString *)replacement {
    return nil;
}
```

There should **always** be a space between the `-` or `+` and the return type (`(void)` in this example). There should **never** be a space between the return type and the method name.

There should **never** be a space before or after colons. If the parameter type is a pointer, there should **always** be a space between the class and the `*`.

The return type should **never** be ommitted (defaults to id for methods and to int for functions).

There should **always** be a space between the end of the method and the opening bracket. The opening bracket should **always** be on the same line.


## Pragma Mark and Implementation Organization

An excerpt of a UIView:

```objective-c

////////////////////////////////////////////////////////////////////////
#pragma mark - Lifecycle
////////////////////////////////////////////////////////////////////////

- (void)dealloc {
    // Unregister Notifications, clean up etc.
}


////////////////////////////////////////////////////////////////////////
#pragma mark - UIView
////////////////////////////////////////////////////////////////////////

- (id)layoutSubviews {
    [super layoutSubviews];
    
    // Stuff
}


- (void)drawRect:(CGRect)rect {
    // Drawing code
}
```

Methods should be grouped by inheritance, the first group however should always be called `Lifecycle` and should contain all initializers (regardless of inheritance) and dealloc if implemented. In the above example, if some `UIResponder` methods were used, they should go between the `NSObject` and `UIView` methods since that's where they fall in the inheritance chain. Protocol-Methods should go right after the methods defined in the class and the last group should be called `Private` and should contain stuff only used internally. 

There should be a pragma mark before each group, formatted like above (A good tip is to create a Xcode snippet with the above layout and use the key `pragma` for example).


## Control Structures

There should **always** be a space after the control structure (i.e. `if`, `else`, etc).


### If/Else

```objective-c
if (button.enabled) {
    // Stuff
} else if (otherButton.enabled) {
    // Other stuff
} else {
    // More stuf
}
```

`else` statements should begin on the same line as their preceding `if` statement.
    
```objective-c
// Comment explaining the conditional
if (something) {
    // Do stuff
}

// Comment explaining the alternative
else {
    // Do other stuff
}
```

If comments are desired around the `if` and `else` statement, they should be formatted like the example above with one empty line between the end of a block and the following comment.


### Switch

```objective-c
switch (something.state) {
    case 0: {
        // Something
        break;
    }
    
    case 1: {
        // Something
        break;
    }
    
    case 2:
    case 3: {
        // Something
        break;
    }
    
    default: {
        // Something
        break;
    }
}
```

Brackets are desired around each case. If multiple cases are used, they should be on separate lines. `default` should **always** be the last case and should **always** be included.


### For

```objective-c
for (NSInteger i = 0; i < 10; i++) {
    // Do something
}


for (NSString *key in dictionary) {
    // Do something
}
```

When iterating using integers, it is preferred to start at `0` and use `<` rather than starting at `1` and using `<=`. Fast enumeration is generally preferred because it is easier on the eyes as well as faster. Also consider using block enumeration where applicable.


### While

```objective-c
while (something < somethingElse) {
    // Do something
}
```

## Import

**Always** use `@class` whenever possible in header files instead of `#import` since it has a slight compile time performance boost.

From the [Objective-C Programming Guide](http://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ObjectiveC/ObjC.pdf) (Page 38):

> The @class directive minimizes the amount of code seen by the compiler and linker, and is therefore the simplest way to give a forward declaration of a class name. Being simple, it avoids potential problems that may come with importing files that import still other files. For example, if one class declares a statically typed instance variable of another class, and their two interface files import each other, neither class may compile correctly.

### Header Prefix

Adding frameworks that are used in the majority of a project to a header prefix is preferred. If these frameworks are in the header prefix, they should **never** be imported in source files in the project.

For example, if a header prefix looks like the following:

```objective-c
#ifdef __OBJC__
    #import <Foundation/Foundation.h>
    #import <UIKit/UIKit.h>
#endif
```

`#import <Foundation/Foundation.h>` should never occur in the project outside of the header prefix.

## Properties

```objective-c
@property (nonatomic, strong) UIColor *topColor;
@property (nonatomic, assign) CGSize shadowOffset;
@property (nonatomic, unsafe_unretained, readonly) UIActivityIndicatorView *activityIndicator;
@property (nonatomic, assign, getter=isLoading) BOOL loading;
```

If the property is `nonatomic`, it should be first. The next option should **always** be `strong`, `weak`, or `assign` since if it is omitted, there is a warning. `readonly` should be the next option if it is specified. `readwrite` should never be specified in header file. `readwrite` should only be used in class extensions to overwrite a specified `readonly` in the headwer file. `getter` or `setter` should be last, `setter` should rarely be used, `getter` should mainly be used for BOOL properties.

See an example of `readwrite` in the *Private Methods* section.

iVars should be omitted since properties are auto-synthesized, each @synthesize should be on a single line and should synthesize a member-variable with a trailing _ . This is to not create a name clash with either Apple's private iVars (leading _) or parameter names (eliminates the need for cryptic parameter names like aTableView instead of just tableView and there is no need to rename the parameternames generated by code completion).

```objective-c
@synthesize topColor = topColor_;
```

## Encapsulation
Encapsulation should be used to restrict access to certain component of an object (private methods, variables, etc.) and to group related functionality. Dependencies should be kept as minimal as possible, especially global dependencies should be avoided.

The header file should only contain methods and properties, that belong to you interface. Private methods, properties etc. should be put in a private class extension at the top of your implementation file. Header files generally don't contain an iVar-section.

## Methods
Any method that is empty or just calls super should be removed from the source code as they are redundant.

## dealloc
If -dealloc is implemented (ARC) it should be placed directly below the -init methods of any class. If there are no -init methods then it should be the first method after class level methods. On non-ARC code dealloc **must** always be implemented and release all the iVars.

## init
The designated initializer of a class shall be marked with a comment in the header file, the designated initializer of the base-class should **always** be overridden. e.g. always override initWithFrame in your UIView-subclass. Dot-Notation shall not be used in init and dealloc, only direct iVar-access.

## Private Methods and Properties

MyShoeTier.h

```objective-c
@interface MyShoeTier : NSObject 

// In general no iVar-section

@property (noatomic, strong, readonly) MyShoe *shoe;

...

@end
```

MyShoeTier.m

```objective-c
#import "MyShoeTier.h"

@interface MyShoeTier () {
  // Here can go private iVars if there is the need to
}

@property (nonatomic, strong, readwrite) MyShoe *shoe;
@property (nonaomic, strong) NSMutableArray *laces;

- (void)crossLace:(MyLace *)firstLace withLace:(MyLace *)secondLace;

@end

@implementation MyShoeTier

// Here shall go the @synthesize-block
...

@end
```

Private methods should **always** be created in a class extension for simplicity since a named category can't be used if it adds or modifies any properties.

Note: The above example provides an example for an acceptable use of a `readwrite` property.

## Naming

In general, everything should be prefixed with a 2 letter prefix. Longer prefixes are acceptable, but not recommended.

It is a good idea to prefix classes with an application specific application if it is application specific code. If there is code you plan on using in other applications or open sourcing, you should prefix it with the company-wide prefix `NG`.

Here are a few examples:

```objective-c
NGLoadingView     // Simple view that can be used in other applications
AppDelegate       // AppDelegate should ALWAYS be named AppDelegate
TVLoadingView     // `NGLoadingView` customized for the application TVthek
```

Classes, Categories, Functions and Protocols should **always** start with a capitalized letter, variables, properties and method names with a lowercase letter.

## Naming Again

Variables, Methods or Classes should never be abbreviated, verbose naming, as heavily used by Apple, is preferred. Ambiguity shall be avoided, as well as connectives like `and` and `with` in method names.

```objective-c
NGBrowserViewController                             // instead of NGBrowserVC
currentPageIndex                                    // instead of pageIdx or just idx

- (void)updateSegmentTextViewContentSize
- (void)updateBarButtonItemsForInterfaceOrientation:(UIInterfaceOrientation)interfaceOrientation
```

Methods that return (autoreleased) objects should follow the following naming scheme, get should not be used for marking getters like in other languages (e.g. Java):

```objective-c
[object thing+condition];
[object thing+input:input];
[object thing+identifer:input];

// Examples:
realPath    = [path stringByExpandingTildeInPath];
fullString  = [string stringByAppendingString:@"Extra Text"];
object      = [array objectAtIndex:3];
```

If you want a variation on a value the following format is possible too:

```objective-c
[object adjective+thing];
[object adjective+thing+condition];
[object adjective+thing+input:input];

// Examples:

capitalized = [name capitalizedString];
rate        = [number floatValue];
newString   = [string decomposedStringWithCanonicalMapping];
subarray    = [array subarrayWithRange:segment];
```

## Categories

Categories should mainly be used for code where the implementation is not available (framework code) and should be avoided if possible, because they can create name clashes and weird bugs. Categories should always be named like follows:

```objective-c
@interface UITableView (NGLoading)
@interface UITableView (TVLoading)
```

The files shall be named accordingly:
UITableView+NGLoading.h/.m, UITableView+TVLoading.h/.m


## Global C Functions

Functions should start with a capitalized Prefix and should follow the same naming scheme as classes (`get` shall be used for functions that return values opposed to method-getters).

```objective-c
NGGetGlobalFormatOfString(@"test");
```

## Extern, Const, and Static

```objective-c
extern NSString *const kNGMyConstant;
extern NSString *myExternString;
static NSString *const kNGMyStaticConstant;
static NSString *staticString;
```

## Notifications

Notification names should **always** follow the following naming scheme:

Class of Affected Object + Did/Will + Action + "Notification"

Example:

```objective-c
UIApplicationWillResignActiveNotification
```

## Booleans

Boolean variables shouldn't be compared against YES and NO, since they already represent YES or NO. YES and NO is tu use instead of true/false, TRUE/FALSE, 0/1, etc.
Each other datatype used in a condition must be explicitly converted to a BOOL by using an equation, e.g. 

* if (pointer != nil) instead of if (pointer)
* if (intValue == 0) instead of if (!intValue)
* if (strcmp("test",myCString) == 0) instead of if (!strcmp("test",myCString))
* ...

## Enums

Enum values should always be prefixed with the name of the enum, following Apple's guidelines. For Bitmask-Type enums the shift-left operator (<<) should be used.

```objective-c
enum {
    Foo,
    Bar
};

typedef enum {
    NGLoadingViewStyleLight,
    NGLoadingViewStyleDark
} NGLoadingViewStyle;

typedef enum {
    NGHUDViewStyleLight = 7,
    NGHUDViewStyleDark = 12
} NGHUDViewStyle;

typedef enum {
    NGHUDViewTypeSingle = 1 << 0,    // For bitmasks
    NGHUDViewTypeDouble = 1 << 1,
    NGHUDViewTypeTriple = 1 << 2
} NGHUDViewType;
```

## Dynamic Typing

Is it preferred to tell the compiler the type of a variable, if possible. That means the use of bare `id` should be avoided, instead a specified protocol or class name should be used.

## Singletons

Singletons should be **avoided** whenever possible since they create a global dependency and are considered bad practise. However there are of course appropriate uses of Singletons, but there should not be a general `ApplicationManager` or something similar that handles a lot of functionality (see Encapsulation).

## Blocks/GCD

Blocks shall be heavily used and are preferred to delegate-methods in a lot of cases (but not all). GCD shall be used as a lightweight mechanism for parallelism, whenever cancellation of operations is needed NSOperation/NSOperationQueue should be used instead.


## Info.plist

The info.plist should always be named Info.plist and not Project_Info.plist or anything else.

## App Delegate

The application delegate, if its a separate class, should be named AppDelegate and not Project_AppDelegate or anything else.

## Other Sources to look at

* Apple HIG: http://developer.apple.com/library/ios/#documentation/UserExperience/Conceptual/MobileHIG/Introduction/Introduction.html
* The Objective-C programming language: http://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/ObjectiveC/Introduction/introObjectiveC.html
* http://stackoverflow.com/questions/155964/what-are-best-practices-that-you-use-when-writing-objective-c-and-cocoa
* http://cocoadevcentral.com/articles/000082.php
* http://cocoadevcentral.com/articles/000083.php

## Example Code

```objective-c

// MSCanteenMenuParser.h

#import "MSJSONParser.h"

@class MSCanteen;    // class-directive instead of import

@interface MSCanteenMenuParser : MSJSONParser // no iVar-section

// the designated initializer
- (id)initWithJSON:(id)JSON canteen:(MSCanteen *)canteen;

@end
```

```objective-c

// MSCanteenMenuParser.m

#import "MSCanteenMenuParser.h"
#import "MSCanteen.h" // import here to make known in implementation-file
#import "FKIncludes.h"

// Private Class Extension for private methods, properties etc.
@interface MSCanteenMenuParser ()

@property (nonatomic, strong) MSCanteen *canteen;

@end

@implementation MSCanteenMenuParser

@synthesize canteen = canteen_;    // synthesize with trailing "_"

////////////////////////////////////////////////////////////////////////
#pragma mark - Lifecycle
////////////////////////////////////////////////////////////////////////

- (id)initWithJSON:(id)JSON canteen:(MSCanteen *)canteen {
    if ((self = [super initWithJSON:JSON])) {
        canteen_ = canteen;
    }
    
    return self;
}

- (id)initWithJSON:(id)JSON {
    // Usage of Assert
    NSAssert(NO, @"Cannot parse menu without known canteen");
    return [self initWithJSON:JSON canteen:nil];
}

////////////////////////////////////////////////////////////////////////
#pragma mark - MSJSONParser
////////////////////////////////////////////////////////////////////////

- (void)parseWithCompletion:(ms_network_block_t)completion {
    // all menus of the given canteen
    NSArray *menus = [self.JSON valueForKey:FKConstant.JSON.CanteenMenu.RootNode];
    
    if (menus.count == 0) {
        [self callCompletion:completion statusCode:MSStatusCodeJSONError];
    }
    
    // format of long method call
    [FKPersistenceAction persistData:menus
                          entityName:[MSCanteenMenu entityName]
                     dictionaryIDKey:FKConstant.JSON.CanteenMenu.Name
                       databaseIDKey:[MSCanteenMenu uniqueKey]
                         updateBlock:^(MSCanteenMenu *canteenMenu, NSDictionary *data, NSManagedObjectContext *localContext) {
                             canteenMenu.canteen = [self.canteen inContext:localContext];
                             
                             // parse the canteen menu itself
                             [canteenMenu setFromDictionary:data];
                             // parse the menu items
                             [canteenMenu setItemsFromArray:[data valueForKey:FKConstant.JSON.CanteenMenu.Items]];
                             
                             [localContext save];
                         } completion:^{
                             [self callCompletion:completion statusCode:MSStatusCodeSuccess];
                         }];
}

@end


```

#### Attribution

This document is heaviliy based on Sam Soffes' coding conventions, thanks for making publicly available.