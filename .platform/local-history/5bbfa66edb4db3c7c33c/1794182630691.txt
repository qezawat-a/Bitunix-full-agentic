#!/bin/sh

# Copyright (c) 2007-2024 The Gradle Foundation and its contributors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# ---------------------------------------------------------------------------
# This is the shell wrapper for the Gradle start script.
# ---------------------------------------------------------------------------

# Add default JVM options here. You can also use JAVA_OPTS and GRADLE_OPTS to pass JVM options to this script.
DEFAULT_JVM_OPTS='"-Xmx64m" "-Xms64m"'

# Use the maximum available, or set MAX_FD != -1 to use that value.
MAX_FD="maximum"

warn () {
    echo "$*"
}

die () {
    echo
    echo "$*"
    echo
    exit 1
}

# OS specific support (must be 'true' or 'false').
cygwin=false
msys=false
darwin=false
case "$(uname)" in
  CYGWIN* )
    cygwin=true
    ;;
  Darwin* )
    darwin=true
    ;;
  MINGW* )
    msys=true
    ;;
esac

# Attempt to set APP_HOME
# Resolve links: $0 may be a link
PRG="$0"
# Need this for relocatable directories.
while [ -h "$PRG" ] ; do
    ls=$(ls -ld "$PRG")
    link=$(expr "$ls" : '.*-> \(.*\)$')
    if expr "$link" : '/.*' > /dev/null; then
        PRG="$link"
    else
        PRG=$(dirname "$PRG")"/$link"
    fi
done
SAVED="$(pwd)"
cd "$(dirname \"$PRG\")/" >/dev/null
APP_HOME="$(pwd -P)"
cd "$SAVED" >/dev/null

APP_NAME="Gradle"
APP_BASE_NAME=$(basename "$0")

# Add default JVM options here. You can also use JAVA_OPTS and GRADLE_OPTS to pass JVM options to this script.
DEFAULT_JVM_OPTS='"-Xmx64m" "-Xms64m"'

# For Cygwin or MSYS, switch paths to Windows format before running java
if $cygwin || $msys; then
    APP_HOME=$(cygpath --path --mixed "$APP_HOME")
    CLASSPATH=$(cygpath --path --mixed "$CLASSPATH")
    JAVACMD=$(cygpath --unix "$JAVACMD")

    # We now build the pattern to construct CLASSPATH for Windows.
    ROOTDIRSRAW=$(find -L / -maxdepth 1 -mindepth 1 -maxdepth 1 -type d 2>/dev/null)
    SEP=""
    for dir in $ROOTDIRSRAW; do
        ROOTDIRS="$ROOTDIRS$SEP$dir"
        SEP="|"
    done
    OURCYGPATTERN="(^($ROOTDIRS))"
    # Add a trailing slash if the path is missing one, else it's considered a file.
    if echo "$APP_HOME" | grep -E "$OURCYGPATTERN" > /dev/null; then
        APP_HOME=$(echo "$APP_HOME" | sed -e 's#^\($OURCYGPATTERN\)#\1/#')
    fi
fi

# Split up the JVM_OPTS And GRADLE_OPTS environment variables into an array.
# The first element of array is removed if empty ($DEFAULT_JVM_OPTS).
eval "set -- $DEFAULT_JVM_OPTS $JAVA_OPTS $GRADLE_OPTS \"\$@\""

# Collect all arguments for the java command, following shell quoting and substitution rules.
set -- \
    -Dorg.gradle.appname="$APP_BASE_NAME" \
    -classpath "$APP_HOME/gradle/wrapper/gradle-wrapper.jar" \
    org.gradle.wrapper.GradleWrapperMain \
    "$@"

# Stop when "xargs" is not available.
if ! command -v xargs >/dev/null 2>&1; then
    die "xargs is not available"
fi

# Use "xargs" to parse quoted args.
#
# With xargs1996:
#   $ xargs0 <<< "$args"
#   # The "-0" flag is not supported by BSD xargs
#   # (BSD xargs supports -0 since xargs 4.6.0-1)
#   $ printf '%s\0' "$args" | xargs0 -n1 -0
#
# With xargs1996 (again, BSD xargs):
#   $ xargs0 <<< "$args"
#   # The "-0" flag is not supported by BSD xargs
#   # (BSD xargs supports -0 since xargs 4.6.0-1)
#   $ printf '%s\0' "$args" | xargs0 -n1 -0
#
# With xargs1996 (BSD xargs):
#   $ xargs0 <<< "$args"
#   # The "-0" flag is not supported by BSD xargs
#   # (BSD xargs supports -0 since xargs 4.6.0-1)
#   $ printf '%s\0' "$args" | xargs0 -n1 -0

exec java "$@"
