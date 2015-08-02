This questions the documentation of Chapter 20.1

The configuration is applied in following order (if an option is configured in multiple locations the last one wins):
* from gradle.properties in project build dir.
* from gradle.properties in gradle user home.
* from system properties, e.g. when -Dsome.property is set on the command line.

After checking out this code, run in this sequence, to see discrepany.

test --stop
test
test --stop
test -Dorg.gradle.daemon=true
test --stop

Result of first stop can be ignored, it's just to ensure no daemon is running

'test' will print out various system variables with interesting values

The second test invocation should say "No Gradle daemons are running."

The next test will start up a daemon, but spit out "false" for system prop.

The last test will shut down the running daemon just for completeness.

What appears to be happening is that system properties (aka -D props)
are indeed being passed to each JVM first as expected, and will take effect
first _IF_ they are only read before any other processing happens.  At
some point after starting a JVM instance, the system property is set to
the value from either the project gradle.properties or the user
gradle.properties file (depending upon whether it is in both or not).

This oddity should either be documented, or there needs to be code that skips
overriding the values for properties that have been explicitly set from
-D or --system-prop flags on the command line.

Why does this potentially matter?  

Try to explicitly set -Djava.net.ssl.keyStore on the command line and
also set it in ~/.gradle/gradle.properties.  Setting an explicit
keyStore can be necessary when dealing with two-way SSL since Java's
default SSL implementation (which Gradle uses for HTTPS connections)
is pretty limited when attempting to have multiple private keys within
the same keystore.

By the time an SSL connection is attempted by the JVM process, it only
sees the property value assigned to the one in the gradle.properties
file.

The only work-around for a situation like this is to point to a
different user home directory.  Clearly, that is not always desirable,
because the complete set of wrappers, cached external artifacts,
etc. will be duplicated, and the only thing that will be different is
a few entries in that gradle.properties file.
