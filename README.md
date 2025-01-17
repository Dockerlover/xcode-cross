Cross compilation with Xcode tools
==================================

This is a reproducible Dockerfile for cross compiling for Apple platforms
from Linux, usable as base image for CI style setups.

This builds on [cctools-port](https://github.com/tpoechtrager/cctools-port)
for providing the basic tools for cross compilation, together with
[xcbuild](https://github.com/facebook/xcbuild) for building xcode project
files, wrapping it up into a easy to use and reproducible Dockerfile.

This setup currently uses a vanilla LLVM/Clang release instead
of the Apple provided clang sources.

This requires an original Xcode.app bundle (which can't be redistributed
freely). It has been tested with and should work with Xcode versions 7,
8 and 9 up to 9.2, but the 9.3 and 9.4 version doesn't work (requiring a
newer version of libtapi, that hasn't been opensourced by Apple yet).
Older versions of Xcode can be downloaded from [Apple](https://developer.apple.com/download/more/).

The Xcode bundle can be stripped down to more manageable sizes for use
with this cross compilation setup, since very little of the bundle
actually is used:

    cp -a /Applications/Xcode.app .
    ./strip-xcode.sh Xcode.app
    tar -Jcvf xcode.tar.xz Xcode.app
    rm -rf Xcode.app

Host the resulting xcode.tar.xz file somewhere, then build the docker
image like this:

    docker build --build-arg XCODE_URL=https://qiniu.17ued.club/xcbuild/xcode.tar.xz .

The docker image has got all the necessary tools added to the path, but
at the end of the path, so that e.g. `gcc` still resolves to the normal
host compiler (for cases when projects need to compile things for both
the host and the target). The cross tools can be used with normal cross
compilation prefixes like `x86_64-apple-darwin-gcc` (which invokes clang,
not gcc), or less architecturally biased like `apple-darwin-gcc` (for cases
when the architecture will be specified with an `-arch` parameter.
