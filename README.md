# momdec: Core Data Model Decompiler

**momdec** is a command-line tool for Mac OS X that takes a compiled Core Data model and decompiles it to produce an equivalent `xcdatamodel` or `xcdatamodeld` suitable for use in Xcode. The resulting model file can also be used with [mogenerator](https://github.com/rentzsch/mogenerator) to produce source code files for Core Data entities which have custom subclasses.

# Usage

    momdec (Foo.mom|Foo.momd|Foo.app|baseline.zip) [output directory]

The first argument is the full path used to locate a compiled managed object model file, and the second is the location where the results should be written. If the second argument is omitted, the current working directory is used. Output files are automatically named based on the inputs.

The first argument can be one of several possibilities:

* If it's a `.mom`, that is, a single managed object model, `momdec` produces a `.xcdatamodel`.
* If it's a `.momd` (which potentially contains multiple managed object models), `momdec` produces a `.xcdatamodeld` containing all models found, as well as a `.xccurrentversion` file (if appropriate) indicating the current version.
* If it's a `.app` application bundle, `momdec` locates the first `.mom` or `.momd` in the bundle and decompiles it.
* If it's an iCloud-style `baseline.zip` file, `momdec` locates the enclosed data model and produces a `.xcdatamodel` from it.

## Command line

    momdec Foo.mom /private/tmp/

Creates `Foo.xcdatamodel` in /private/tmp/

    momdec Foo.momd

Creates `Foo.xcdatamodeld` in the current working directory. This bundle will include all model versions present in the `momd` and (if appropriate) a `.xccurrentversion` file.

## Source code

This project includes a number of categories on Core Data classes which could be used in other projects. The main entry point would be in `NSManagedObjectModel+xmlElement.h`, which includes the following methods:

    - (NSXMLElement *)xmlElement;

Returns an `NSXMLElement` representing the model

    - (NSXMLDocument *)xmlDocument;

Returns a full `NSXMLDocument` representing the model. This just calls `xmlElement`, sets that element as the document root, and adds document-level metadata.

    + (NSString *)decompileModelAtPath:(NSString *)modelPath inDirectory:(NSString *)resultDirectoryPath error:(NSError **)error;

Decompiles the `mom`, `momd`, `app`, or `baseline.zip` at the specified path, saves the contents in the result directory, and returns the full path of the decompiled model.

Other categories consist of just an `xmlElement` method, which returns an `NSXMLElement` representing the receiver's portion of the decompiled model document.

# Requirements

Developed with Mac OS X 10.8.3 and Xcode 4.6.1. May work with older versions of both, but this has not been tested.

# License

MIT-style license, see LICENSE for details.

# Limitations

Models that were compiled with Xcode may be missing some data due to the following bugs. Since this data does not exist in the compiled model file, `momdec` cannot restore it when decompiling the model:

* Min/max values on Core Data decimal attributes will not be correct if the limits are not integers, because Xcode truncates the limits to integers at compile time (rdar://problem/13677527, also on [OpenRadar](http://openradar.appspot.com/radar?id=2948402)).
* Fetch requests will be missing any non-default values for the following settings, because they are lost when compiling with Xcode (rdar://problem/13863607, also on [OpenRadar](http://www.openradar.me/radar?id=3009404)):
    * Result Type
    * Fetch limit
    * Batch size
    * Include Subentities
    * Include Property Values
    * Return Objects as Faults
    * Include Pending Changes
    * Return Distinct Results

# Credits

By Tom Harrington, @atomicbird on most social networks.
