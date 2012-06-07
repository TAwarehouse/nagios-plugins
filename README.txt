Copyright 2012 TripAdvisor, LLC

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

        http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

======================================================================

This is a set of plugins for the Nagios monitoring system
(http://www.nagios.org).  They monitor Hadoop and Hive information that we
have found useful in our data warehouse implementation.  Some are more
generic system tests for which we did not find existing Nagios plugins,
but we needed them to monitor our clusters.

You install these just like any other third-party plugin to Nagios: just
drop them into the "libexec" directory within the Nagios installation tree,
make the executable and add the appropriate commands to the "etc/nrpe.cfg"
file.  For example:

command[check_cronjobs]=/usr/local/nagios/libexec/check_cronjobs -w 24 -c 48 
command[check_local_mail]=/usr/local/nagios/libexec/check_local_mail
command[check_hdp_connectivity]=/usr/local/nagios/libexec/check_hdp_connectivity --hadoop_root /hadoop

Then check and reload your Nagios configuration:

     service nagios checkconfig
     service nagios reload

The comments within each plugin describe what it does and any arguments in
accepts.

	- The TripAdvisor Data Warehouse Team.
