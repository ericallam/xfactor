xfactor
=======

Refactoring Objective-C in Xcode (and AppCode) are great, but are not automatable, are not repeatable-with-small-changes. For large scale refactorings, scripting can be your best friend. But unfortunately the Xcode (or AppCode) refactoring tools are limited to their respective UI's. 

That's where xfactor comes in.

xfactor allows opens up the world of automation to your Objective-C refactorings, with:

- Rename file
- Duplicate file with a new name
- Rename symbol (class name, protocol name, string constant, etc.)
- Rename method

### Usage

Every xfactor command works with a `xcodeproj` file to make changes to the appropriate files in your project. For example, if you wanted to rename a class from `EAModel` to `ELAModel`, you would run the `rename-class` command like so:

```
$ xfactor rename-class EAModel ELAModel -p Project.xcodeproj
```

You can also limit the scope of changes to a subgroup of the project (in this case, the `Classes` group), like so:

```
$ xfactor rename-class EAModel ELAModel -p Project.xcodeproj -s Classes
```

Xcode projects can have many targets, so you can also limit changes to files that belong to that target:

```
$ xfactor rename-class EAModel ELAModel -p Project.xcodeproj -t EAApp
```

Of course, using these options might mean that there will be old symbols in other parts of your project that don't get renamed, so use them judiciously. 

The `rename-class` command will also rename the corresponding file if it matches the class name. For example, in the command above `EAModel.m` would be renamed to `ELAModel.m` and `EAModel.h` would be renamed to `ELAModel.h`. You can opt-out of the file renaming by passing the `--no-file` option, like so:

```
$ xfactor rename-class --no-file EAModel ELAModel -p Project.xcodeproj
```

Later, if you decide you do want to change the file, you can use the `rename-file` command to do just that and nothing more:

```
$ xfactor rename-file EAModel ELAModel -p Project.xcodeproj
```

The command above renames both implementation `.m` and header `.h` files, but if you want to limit the refactor to a specific file, just include the extension:

```
$ xfactor rename-file EAModel.m ELAModel.m -p Project.xcodeproj
```

### Rename method

xfactor also includes method name changes. For example, if you wanted to change the name of `sharedInstance` to `shared` on the `EAModel` class:

```
$ xfactor rename-method EAModel+sharedInstance shared -p Project.xcodeproj
```

A more complicated method name, with multiple params, can also work:

```
$ xfactor rename-method EAModel-initWithPath:queue: initWithFilePath:backgroundQueue: -p Project.xcodeproj
```

You can even swap arguments at all invocations and declarations. For example, if this will swap the second and third arguments:

```
$ xfactor rename-method EAModel-initWithName:queue:<2>relationships:<3> EAModel-initWithName:relationships:<3>backgroundQueue:<2> -p Project.xcodeproj
```

Since xfactor is a wrapper around the `tops` command, you can easily drop down into raw `tops` rules to perform even more advanced method refactorings, like the following which will rename the method `minFrameWidth:forStyle:buttonMask:` to `minFrameWidthWithTitle:styleMask:` in all occurrences. Within invocations of this method, it changes the style argument to be the logical OR of the previous style argument and the previous button mask argument. Within method declarations, it changes the type for the style argument to be unsigned int. Within the implementation of this method, it changes all uses of the button mask argument to the style argument.

File `advancedReplace.tops`:

```
replacemethod "minFrameWidth:forStyle:<style> buttonMask:<bmask>" with "minFrameWidthWithTitle:styleMask:<style>" {
  replace "<style_arg>" with "<style_arg>|<bmask_arg>"
  replace "<style_type>" with "(unsigned int)"
} within ("<implementation>") { replace "<bmask_param>" "<style_param>" }
```

```
$ xfactor raw advancedReplace.tops -p Project.xcodeproj
```

The main benefit of using `xfactor raw` over plain `tops` is that `tops` doesn't know anything about project structure and files. `xfactor` does and will automatically apply this refactoring project-wide.

### Installation

`xfactor` is a ruby gem and should work with Mac OS X out of the box by installing it like so:

```
$ gem install xfactor
```

After installing it you can read much more detailed usage information using `man xfactor` or `xfactor help`. You might also want to read more about the `tops` command line tool that powers much of xfactor.


### Limitations

Currently, xfactor uses the `tops` command line tool which doesn't have any semantic knowledge of your code, which means it's refactorings aren't as "clean" as Xcode's. This means that while `xfactor rename-class` works fairly well, `xfactor rename-method` will sometimes rename methods that you don't intend. For example, if you rename `sharedInstance` to `instance`, all method definitions and invocations, no matter what class or kind of object they belong to, will be renamed.

### Roadmap

In the future, it would be great to migrate this tool to use clang and libTooling, so full semantic knowledge of the program can be used to do smarter and more interesting refactorings.

### Author

Eric Allam. Contact on [twitter](http://twitter.com/eallam)

### License

xfactor is available under the MIT license. See the LICENSE file for more info
