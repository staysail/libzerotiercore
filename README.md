= libzerotiercore

This repository is really just a wrapper around the ZeroTierOne
project (from ZeroTier.com).  While that project does offer a minimal
CMakeFile for building a "core" version of the library (suitable for
integration in stackless projects like the NNG transport -- see
github.com/nanomsg/nng), the integration effort is rather limited at
best, and not sufficient for complete integration as a 3rd party project.

This repository attempts to correct that deficiency, while still using
the core ZeroTierOne repository.

To be clear, this repository is for building a stackless version of the
core libzerotiercore library. If you need a version with TCP/IP support,
please see the libzt project from ZeroTier.

== LICENSE

This repository is provided for now under the terms of the GPLv3, which
match the license used for the actual ZeroTierOne.  If you've purchased
a commercial license for ZeroTierOne and want to use this repository to
facilitate your own integration, please contact us. (We'd be happy to offer
different terms, but chose the GPLv3 to match the upstream project's license
so as to avoid any possible licensing confusion.)

