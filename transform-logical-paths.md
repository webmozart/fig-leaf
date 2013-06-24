PSR-T: Transformation Of Logical Paths To File System Paths
===========================================================

This document describes an algorithm to transforms a logical path to a file
system path. Among other things, this general-purpose tranformation algorithm
allows mapping of class names and other resource names to file system paths.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).


1. Definitions
--------------

**Logical Separator**: A single character to delimit _logical segments_; for
example, a slash, backslash, colon, etc.

**Logical Segment**: A string delimited by _logical separators_.

**Fully Qualified Logical Path**: A _logical separator_ followed by zero or
more _logical segments_ delimited by _logical separators_. Given a _logical
separator_ of "/", then `/`, `/Foo`, and `/Foo/Bar` are _fully
qualified logical paths_.

**Logical Path Prefix**: A _logical path prefix_ is any contiguous series of
_logical separators_ and _logical segments_ at the beginning of a
_fully qualified logical path_, terminating in a _logical separator_. For
example, given a _logical separator_ of "/", then `/`, `/Foo/`, and `/Foo/Bar/`
are _logical path prefixes_ for a _fully qualified logical path_ of
`/Foo/Bar/Baz`.

**Logical Path Suffix**: Given a _fully qualified logical path_ and a
_logical path prefix_, the _logical path suffix_ is the remainder of the
_fully qualified logical path_ after the _logical path prefix_. For example,
given a _logical separator_ of "/", a _fully qualified logical path_ of
`/Foo/Bar/Baz/Qux`, and a _logical path prefix_ of `/Foo/Bar/`, then `Baz/Qux`
is the _logical path suffix_.

**File System Base Directory**: A directory in the file system associated with
a _logical path prefix_.


2. Specification
----------------

Given a fully qualified logical path, a logical path prefix, and a logical
separator, implementations:

- MUST replace the logical path prefix with a file system base directory
  associated with that logical path prefix,

- MUST replace logical path separators in the logical path suffix with
  directory separators, and

- MAY append a file name extension.

The result is a file system path that MAY exist.


3. Example Implementation
-------------------------

The example implementation MUST NOT be regarded as part of the specification;
it is an example only. Implementations MAY contain additional features and MAY
differ in how they are implemented.

```php
<?php
/**
 * Example implementation.
 * 
 * Note that this is only an example, and is not a specification in itself.
 * 
 * @param string $logical_path The logical path to transform.
 * @param string $logical_prefix The logical prefix associated with $base_dir.
 * @param string $logical_sep The logical separator in the logical path.
 * @param string $base_dir The base directory for the transformation.
 * @param string $file_ext An optional file extension.
 * @return string The logical path transformed into a file system path.
 */
function transform(
    $logical_path,
    $logical_prefix,
    $logical_sep,
    $base_dir,
    $file_ext = null
) {
    // make sure the logical prefix ends in a separator
    $logical_prefix = rtrim($logical_prefix, $logical_sep)
                    . $logical_sep;
    
    // make sure the base directory ends in a separator
    $base_dir = rtrim($base_dir, DIRECTORY_SEPARATOR)
              . DIRECTORY_SEPARATOR;
    
    // find the logical suffix 
    $logical_suffix = substr($logical_path, strlen($logical_prefix));
    
    // transform into a file system path
    return $base_dir
         . str_replace($logical_sep, DIRECTORY_SEPARATOR, $logical_suffix)
         . $file_ext;
}

/**
 * Example transformations.
 */
// "\Foo\Bar\Baz\Qux" => "/path/to/foo-bar/src/Baz/Qux.php"
transform(
    '\Foo\Bar\Baz\Qux',
    '\Foo\Bar',
    '\\',
    '/path/to/foo-bar/src',
    '.php'
);

// ":Foo:Bar:Baz:Qux" => "/path/to/foo-bar/config/Baz/Qux.yml"
transform(
    ':Foo:Bar:Baz:Qux',
    ':Foo:Bar',
    ':',
    '/path/to/foo-bar/config',
    '.yml'
);

// "/Foo/Bar/Baz/Qux/" => "/path/to/foo-bar/resources/Baz/Qux/"
transform(
    '/Foo/Bar/Baz/Qux',
    '/Foo/Bar',
    '/',
    '/path/to/foo-bar/resources'
);
```
