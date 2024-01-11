# Automation
some automation scripts

migrate_devices :
Migrates devices from one organization to another. The script works in 3 stages:
1. Run the script in export mode to compile device information into a local file
2. Run the script in unclaim mode to remove devices from their source organization
3. Run the script in import mode to claim devices into the destination organization
    as listed in the file exported during the first step
    
As a safety measure, you need to specify a netowrk tag to be used as a filter for 
exporting and unclaiming.

Note that there may be a gap of several minutes between when devices are unclaimed from one
organization and when they are claimable into another. Please plan maintenance breaks
accordingly.

The export function creates a file containing the following information about devices in scope:
    Serial number
    Name
    Latitude
    Longitude
    Street address
    Management interface
    MX ports
    MX SSIDs
    Site-to-site VPN
    Switch ports
   
Network configuration is NOT migrated. Before running this script, make sure your destination
organization has networks with the exact same names as the ones the devices will be removed
from in the source organization. To move network configuration at scale, use a script such
as this one first:
https://github.com/meraki/automation-scripts/tree/master/migrate_networks

Syntax, Windows:
    python migrate_devices.py [-k <api_key>] -o <org_name> -m <mode> [-t <tag>] [-f <file>]
        [-d <device_filter>] [-v <vpn_mode>]
    
Syntax, Linux and Mac:
    python3 migrate_devices.py [-k <api_key>] -o <org_name> -m <mode> [-t <tag>] [-f <file>]
        [-d <device_filter>] [-v <vpn_mode>]
    
Mandatory parameters:
    -o <org_name>       The name of the organization you want to interact with
    -m <mode>           Mode of operation for the script. Valid forms:
                            -m export           Exports device data to file
                            -m unclaim          Removes devices from org
                            -m import           Imports device data from file
                            -m refresh          Does not claim devices, only refreshes config
    
Optional parameters:
    -k <api_key>        Your Meraki Dashboard API key. If omitted, one will be loaded from
                        environment variable MERAKI_DASHBOARD_API_KEY
    -t <tag>            Network tag to be matched for exporting and unclaiming. This parameter
                        is MANDATORY when exporting or unclaiming, but will be IGNORED during
                        import
    -f <file>           Name of the file to be used for export/import/refresh. This parameter is 
                        MANDATORY when importing or refreshing and OPTIONAL when exporting,
                        defaulting to devices_<timestamp>.json if omitted. It will be IGNORED
                        when running the script in unclaim mode
    -d <device_filter>  Only process devices with model names starting with given string, case
                        insensitive
    -v <vpn_mode>       Whether to also process VPN settings. Valid forms:
                            -v none             Do not process VPN configuration (default)
                            -v site             Only process site-to-site VPN configuration
                        * The below forms are experimental and may not work with all configurations
                            -v client           Only process Anyconnect/L2TP client VPN config
                            -v both             Process both client and site-to-site config
              
Example:
    1. Export all devices in networks tagged "franchise" into a file using the default
        filename format, from organization "Big Industries Inc":
    python migrate_devices.py -k 1234 -o "Big Industries Inc" -m export -t franchise
    
    2. Unclaim all devices exported during the previous step:
    python migrate_devices.py -k 1234 -o "Big Industries Inc" -m unclaim -t franchise
    
    3. Claim all exported devices into organization "Big Franchise", assuming timestamp was
        2022-22-21_11.42.00:
    python migrate_devices.py -k 1234 -o "Big Franchise" -m import -f devices_2022-22-21_11.42.00.json
    
****
