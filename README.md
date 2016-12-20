# onewire-nagios-plugin
Nagios Plugin for the OneWire temperature sensors

This is a working, but super simple Nagios plugin to monitor a single OneWire
temperature sensor.

In order to make sure this plugin works, you will need to make sure digitemp
is configured and can connect to a sensor.
* First install digitemp:
   #> yum install digitemp
* Second, connect your USB OneWire sensor to your nagios box.  I am using
  VMware and used USB pass-though to expose it to my nagios server
* Run digitemp to init a config file.  My USB adapter is a DS9097U so this is
  the syntax:
   #> digitemp_DS9097U -i -s /dev/ttyUSB0 -c digitemp.conf
* Your sensors should be found and listed both in the command output and in
  the newly created config file
* Check to see what sensor is the one you want to verify on - you can have
  several but the check_onewire_temp only supports one
* Once you identify the sensor, test getting the temperature for it:
   #> digitemp_DS9097U -t 0 -q -c ./digitemp.conf
* Move your config file to /etc/digitemp.conf
* Edit check_onewire_temp to have the correct sensor
* Run check_onewire_temp as root to verify it works:
   #> ./check_onewire_temp 
   OK: Current Temp: 73.40 | temp=73.40,75,80
* Copy check_onewire_temp to your plugins directory (e.g.
    /usr/lib64/nagios/plugins)
* Create a nagios service and command to run the plugin:
   define service {
     use                             generic-service
     host_name                       localhost
     service_description             Room Temp
     check_command                   check_onewire_temp
     notification_period             workhours
   }
   define command {
     command_name  check_onewire_temp
     command_line  $USER1$/check_onewire_temp
   }
* Add the user nagios is running as to the dialup group; this group has access
  to the USB devices. This is probably probably the user "nagios".
* Check nagios to verify your new config is working:
  #> nagios -v nagios.conf
* Restart nagios.  This is required because we updated the user's group
  membership and that change needs to available for the current nagios process.
  #> service nagios restart
* Check the nagios web portal for the results.

Items To Do:
* Take the sensor as a command-line argument
* Take the warning and critical temperatures as command-line arguments
