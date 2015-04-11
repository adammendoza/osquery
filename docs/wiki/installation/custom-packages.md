We support building custom deployment packages (pkg/deb/rpm) for less common use cases:

- Slipstreaming additional tools into osquery's existing packages
- Proprietary modifications to "core" features that aren't simple additional plugins
- Custom dependency modifications (patched versions of glog, thrift, etc)

The first step to creating custom packages is having [built](development/building) and tested osquery. This means reading the development guides and in most cases having a dedicated "build host".

## Linux

In your cloned osquery repository, once you have [built the code](development/building) (hopefully a tagged release):

```sh
$ make packages
```

This will use CMake and *fpm*, installed as an osquery build dependency, to generate an `osquery-VERSION.{rpm/deb}` and optionally debug and devel packages.

## OS X

In your cloned osquery repository, once you have [built the code](development/building) (hopefully a tagged release):

```sh
$ make packages
```

OS X deployment is a bit more complicated and customizable compared to Linux.
We include some help and guidance for a more esoteric script `make_osx_package.sh`:

```sh
$ ./tools/deployment/make_osx_package.sh -h
```

This tool will build an OS X package with:

- the osqueryi and osqueryd binaries
- the [LaunchDaemon](https://github.com/facebook/osquery/blob/master/tools/deployment/com.facebook.osqueryd.plist) that is responsible for osqueryd
- the osqueryd config file that was specified via the command line using "-c"

Here is the output from us running `make_osx_package.sh`:

```sh
$ ./tools/deployment/make_osx_package.sh -c ~/Desktop/osquery.conf
[+] no custom launchd path was defined. using ~/git/osquery/tools/deployment/com.facebook.osqueryd.plist
[+] copying osquery binaries
[+] copying osquery configurations
[+] finalizing preinstall and postinstall scripts
[+] creating package
[+] package created at ~/git/osquery/build/darwin/osquery-VERSION.pkg
```

The distributable package can be found at `./build/darwin/osquery-VERSION.pkg`.

You can now use your existing package distribution system ([JAMF](http://www.jamfsoftware.com/), [Chef](https://www.getchef.com/chef/), etc) to push this package to your infrastructure.

### Custom LaunchDaemon

If you want to modify the command-line arguments used to start osquery, copy and modify the [LaunchDaemon](https://github.com/facebook/osquery/blob/master/tools/com.facebook.osqueryd.plist) which is included with this repository to suit your liking.

When you run `make_osx_package.sh`, include a `-l`/`--launchd-path` flag which indicates the path of your new LaunchDaemon. If specified, this will be used instead of the default LaunchDaemon. For example:

```sh
$ ./tools/deployment/make_osx_package.sh -c /internal/osquery/osquery.conf \
  -l /internal/osquery/com.facebook.osqueryd.plist
```

### Removing the LaunchdDaemon

Perhaps you just want to deploy the osquery binaries via a pkg and you'd like to manage the scheduling of osqueryd via some other mechanism. To do this, when you run `make_osx_package.sh`, include a `-n`/`--no-launchd` flag.

This will make the package just lay the binaries down. The LaunchDaemon won't be included and no LaunchDaemon will be unloaded or loaded by the post-install script of the package. For example:

```sh
$ ./tools/deployment/make_osx_package.sh -n
```