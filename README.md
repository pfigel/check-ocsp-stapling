check_ocsp_stapling
===================

`check_ocsp_stapling` is a plugin for Nagios/Icinga that helps with monitoring
TLS servers implementing OCSP stapling. The script verifies that a TLS server
provides a OCSP response that is not expired and reports a good (non-revoked) certificate status.

The script is largely based on `check_ssl_certificate` by David Alden.

## Default Behaviour

By default, `check_ocsp_stapling` begins to warn when the stapled OCSP response
is valid for less than 36 hours from the current time and will go to a critical
state at less than 24 hours.

These values can be changed using the `-w` and `-c` command-line options. To
find an appropriate value for your CA, observe the Last Update and Next Update
values the CA uses in their OCSP responses. Publicly-trusted CAs may use a
maximum expiration time of ten days and should update their OCSP responses at
least every four days (as of version 1.3.5 of the CA/B Forum Baseline
Requirements).

## Sample Configuration
### Nagios
```
define host {
    use                generic-host
    host_name          example.com
    alias              Some Remote Host
    address            example.com
}

define command {
    name            check_ocsp_stapling
    command_name    check_ocsp_stapling
    command_line    /path/to/nagios/plugins/check_ocsp_stapling -H $HOSTADDRESS$ $ARG1$
}

define service {
    use                 generic-service
    host_name           example.com
    service_description example.com OCSP Stapling
    check_command       check_ocsp_stapling
    # to use any of the optional arguments, use this syntax:
    # check_command       check_ocsp_stapling!-w 48
}
```
### Icinga
```
object Host "example.com" {
  address = "example.com"
  # to use any of the optional arguments, use this syntax:
  # vars.port = 443
  # vars.warning = 240
}

object CheckCommand "check_ocsp_stapling" {
  command = [ PluginDir + "/check_ocsp_stapling" ]

  arguments = {
    "-H" = "$address$"
    "-a" = "$add$"
    "-c" = "$critical$"
    "-w" = "$warning$"
    "-p" = "$port$"
  }
}

object Service "example.com OCSP Stapling" {
  host_name = "example.com"
  check_command = "check_ocsp_stapling"
}
```

## Usage
```
Usage: check_ocsp_stapling -H <host> [-p <port>] [-c <num>] [-w <num>]

-a <add>   add the text to the openssl line, used for checking OCSP stapling
           with starttls ("-a '-starttls smtp'")
-c <num>   exit with CRITICAL status if number of hours left is less than <num>
-h         show this help script
-H <host>  check OCSP stapling on the indicated host
-o <path>  path to openssl binary
-p <port>  check OCSP stapling on the specified port
-w <num>   exit with WARNING status if number of hours left is less than <num>
-V         show version and license information
```

## Issues
Please report any issues you might encounter on [GitHub](https://github.com/patf/check-ocsp-stapling/issues). Pull requests are
welcome!
