# Development notes 

## PDF support

To add the optional PDF support to Luna, you'll need to add the
`lhpdf` and `libpng`, if they are not present.  The following are
required to build those libraries.

!!! alert
    PDF creation in Luna is not currently supported

- __autotools__ : utilities needed to compile some dependent
libraries; on Mac OSX, with [Homebrew](https://brew.sh) installed,
these can be obtained by typing:
```
brew install automake autoconf libtool
```

