# OpenCV 2.4.9 (packaged by [Pattern](http://pattern.nu))

[![Build Status](https://travis-ci.org/PatternConsulting/opencv.svg?branch=master)](https://travis-ci.org/PatternConsulting/opencv)

[OpenCV](http://opencv.org) Java bindings packaged with native libraries, seamlessly delivered as a turn-key Maven dependency.

## Deprecation Notice

Due to time constraints, this package is not actively maintained. As of December 29, 2015, it's strongly recommended users migrate to [OpenPnP's excellent fork of this project](https://github.com/openpnp/opencv), generously and regularly maintained by [Jason von Nieda](https://github.com/vonnieda).

## Usage

### Project

Pattern's OpenCV package is added to your project as any other dependency.

#### [Maven](http://maven.apache.org/)

```xml
<project>
  
  <!-- ... -->
  
  <dependencies>
    
    <!-- ... -->
    
    <dependency>
      <groupId>nu.pattern</groupId>
      <artifactId>opencv</artifactId>
      <version>4.20</version>
    </dependency>
    
    <!-- ... -->
    
  </dependencies>
  
  <!-- ... -->
  
</project>
```

#### [SBT](http://scala-sbt.org)

```scala
// ...

libraryDependencies += "nu.pattern" % "opencv" % "2.4.9-7"

// ...
```

### API

Typically, using the upstream [OpenCV Java bindings involves loading the native library](http://docs.opencv.org/doc/tutorials/introduction/desktop_java/java_dev_intro.html#java-sample-with-ant) as follows:

```java
static {
  System.loadLibrary(org.opencv.core.Core.NATIVE_LIBRARY_NAME);
}
```

Fortunately, this is unchanged except for one caveat. To use the native libraries included with this package, first call [`nu.pattern.OpenCV.loadShared()`](https://github.com/PatternConsulting/opencv/blob/master/src/main/java/nu/pattern/OpenCV.java).

This call will—exactly once per class loader—first attempt to load from the system-wide installation (exactly as if `System.loadLibrary(org.opencv.core.Core.NATIVE_LIBRARY_NAME);` were called without any preceding steps). If that fails, the loader will select a binary from the package appropriate for the runtime environment's operating system and architecture. It will write that native library to a temporary directory (also defined by the environment), add that directory to `java.library.path`. _This involves writing to disk_, so consider the implications. Temporary files will be garbage-collected on clean shutdown.

This approach keeps most clients decoupled from Pattern's package and loader. As long as this is done sufficiently early in execution, any library using the OpenCV Java bindings can use the usual load call as documented by the OpenCV project.

There are, however, cases where Java class loaders are frequently changing (_e.g._, application servers, SBT projects, Scala worksheets), and [spurious attempts to load the native library will result in JNI errors](https://github.com/PatternConsulting/opencv/issues/7). As a partial work-around, this package offers an alternative API, [`nu.pattern.OpenCV.loadLocal()`](https://github.com/PatternConsulting/opencv/blob/master/src/main/java/nu/pattern/OpenCV.java), which—also exactly once per class loader—extracts the binary appropriate for the runtime platform, and passes it to `System#load(String)`. Ultimately, this may eventually load the library redundantly in the same JVM, which could be unsafe in production. Use with caution and understand the implications.

It's recommended developers using any JNI library read further:

- [JNI 1.2 Specifications: Library and Version Management](http://docs.oracle.com/javase/7/docs/technotes/guides/jni/jni-12.html#libmanage)
- [Holger Hoffstätte's Comments on Native Libraries, Class Loaders, and Garbage Collection](https://groups.google.com/forum/#!msg/ospl-developer/J4i6cF6yPk0/-3Jm3Qs_HDwJ)

## Debugging

[Java logging](http://docs.oracle.com/javase/8/docs/api/java/util/logging/package-summary.html) is used to produce log messages from `nu.pattern.OpenCV`.

## Rationale

Developers wishing to use the Java API for OpenCV would typically go through the process of building the project, and building it for each platform they wished to support (_e.g._, 32-bit Linux, OS X). This project provides those binaries for inclusion as a typical dependency in Maven, Ivy, and SBT projects.

Apart from testing, this package deliberately specifies no external dependencies. It does, however, make use of modern Java APIs (such as [Java NIO](http://docs.oracle.com/javase/tutorial/essential/io/fileio.html)).

## Contributing

Producing native binaries is the most cumbersome process in maintaining this package. If you can contribute binaries _for the current version_, please make a pull request including the build artifacts and any platform definitions in `nu.pattern.OpenCV`.

## Support

The following platforms are supported by this package:

OS | Architecture
--- | ---
OS X | x86_32
OS X | x86_64
Linux | x86_64
Linux | x86_32

If you can help create binaries for additional platforms, please see notes under [_Contributing_](#contributing).

## Credits

This package is maintained by [Michael Ahlers](http://github.com/michaelahlers).
  
## Acknowledgements

- [Greg Borenstein](https://github.com/atduskgreg), who's advice and [OpenCV for Processing](https://github.com/atduskgreg/opencv-processing) project informed this package's development. 
- [Alex Osborne](https://github.com/ato), for helpful [utility class producing temporary directories with Java NIO that are properly garbage-collected on shutdown](https://gist.github.com/ato/6774390).
