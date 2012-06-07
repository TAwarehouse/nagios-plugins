#! /bin/sh
#  Copyright 2012 TripAdvisor, LLC
#
#  Licensed under the Apache License, Version 2.0 (the "License");
#  you may not use this file except in compliance with the License.
#  You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
#  Unless required by applicable law or agreed to in writing, software
#  distributed under the License is distributed on an "AS IS" BASIS,
#  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#  See the License for the specific language governing permissions and
#  limitations under the License.
#
# Inverse Log file pattern detector - complains if a log file
# fails to update according to text
# Written by Mark Schuldenfrei (mschuldenfrei@tripadvisor.com)
# Last Modified: 07-30-2010
#
# inspired by check_log from Nagios, but basically completely rewritten.
#
# Usage: ./check_log_updated -l <log_file> -s <savedata> -t <text>
#
# Description:
# This plugin will scan a log file (specified by the <log_file> option)
# for a specific text (specified by the <text> option).  Successive
# calls to the plugin script will report a FAILURE if there are no new
# matches.  This makes the plugin useful for checking for updates in
# a log file.  Interim data will be stored in the savename file.
#
# Output:
#
# On the first run of the plugin, it will return an OK state with a message
# of "Log check data initialized".  On successive runs, it will return an OK
# state if *some* new pattern matches have been found.  If the plugin
# does not detect any new pattern matches, it will return a CRITICAL state
# and print out a message: "NO CHANGE last_match", where last_match
# is the last line that did match
#
# Notes:
#
# If you use this plugin make sure to keep the following in mind:
#
#    1.  The "max_attempts" value for the service should be 1, as this
#        will prevent Nagios from retrying the service check (the
#        next time the check is run it will not produce the same results).
#
#    2.  The "notify_recovery" value for the service should be 0, so that
#        Nagios does not notify you of "recoveries" for the check.  Since
#        pattern matches in the log file will only be reported once and not
#        the next time, there will always be "recoveries" for the service, even
#        though recoveries really don't apply to this type of check.
#
#    3.  You *must* supply a different <savedata> for each service that
#        you define to use this plugin script - even if the different services
#        check the same <log_file> for pattern matches.  This is necessary
#        because of the way the script operates.
#
# Examples:
#
# Check for regular STATUS messages in somelogfile
#
#   check_log_updated -l somelogfile -s savestatus -t "STATUS"
#

PROGNAME=`/bin/basename $0`
REVISION="1.0"

print_usage() {
    echo "Usage: $PROGNAME -l logfile -s savedata -t text"
    echo "Usage: $PROGNAME --help"
    echo "Usage: $PROGNAME --version"
}

print_help() {
    echo $PROGNAME $REVISION
    echo ""
    print_usage
    echo ""
    echo "Log file pattern detector plugin for Nagios"
    echo "Reports on a log file when the log file fails to change"
    echo ""
}

CRITICAL=2
WARNING=1
OK=0

test='LOG_UPDATED';

critical_exit() {
  echo "$test CRITICAL: $*"
  exit $CRITICAL
}
ok_exit() {
  echo "$test OK: $*"
  exit $OK
}



# Make sure the correct number of command line
# arguments have been supplied

if [ $# -lt 6 ]; then
    print_usage
    critical_exit "Incorrect number of parameters"
fi

# Grab the command line arguments

while test -n "$1"; do
    case "$1" in
        --help)
            print_help
            ok_exit
            ;;
        -h)
            print_help
            ok_exit
            ;;
        --version)
            echo $PROGNAME $REVISION
            ok_exit
            ;;
        -V)
            echo $PROGNAME $REVISION
            ok_exit
            ;;
        --logfilename)
            logfile=$2
            shift
            ;;
        -l)
            logfile=$2
            shift
            ;;
        --savedatafile)
            savedata=$2
            shift
            ;;
        -s)
            savedata=$2
            shift
            ;;
        --text)
            searchtext=$2
            shift
            ;;
        -t)
            searchtext=$2
            shift
            ;;
        *)
            echo "Unknown argument: $1"
            print_usage
            critical_exit "Unknown argument: $1"
            ;;
    esac
    shift
done

# Make sure values we require are set
if [ "$logfile" = "" ]; then
  echo ERROR: log file name not set
  print_usage
  critical_exit "Log file name not set"
fi
if [ "$savedata" = "" ]; then
  echo ERROR: save data file name not set
  print_usage
  critical_exit "Save data file name not set"
fi
if [ "$searchtext" = "" ]; then
  echo ERROR: text string not set
  print_usage
  critical_exit "Text string not set"
fi

# If the source log file doesn't exist, exit

if [ ! -e $logfile ]; then
    echo "Log check error: Log file $logfile does not exist!"
    critical_exit "Log file $logfile does not exist"
elif [ ! -r $logfile ] ; then
    echo "Log check error: Log file $logfile is not readable!"
    critical_exit "Log file $logfile is not readable"
fi

# If the saved data file doesn't exist, this must be the first time
# we're running this test, so store a list of the lines we match

if [ ! -e $savedata ]; then
    fgrep $searchtext $logfile > $savedata
    echo "Log check data initialized..."
    ok_exit
fi

# The saved data file exists, so compare it to the original log now

# The temporary file that the script should use while
# processing the log file.
tempfile=`mktemp /tmp/$PROGNAME.XXXXXXXXXX`

# Do the search for the text
fgrep $searchtext $logfile > $tempfile
lastentry=`tail -1 $tempfile`	 # Save the last line, if we need it.

# Did we differ?
cmp -s $savedata $tempfile
result=$?
cp $tempfile $savedata
rm -f $tempfile

if [ "$result" = "0" ]; then # no changes, so exit with error
    critical_exit "($lastentry)"
else # found something
    ok_exit "Log updated"
fi

critical_exit "Should not reach this line of code"
