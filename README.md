# xbar-plugins
Plugin repository for xbar (BitBar)


## icinga2-status

![CleanShot_xbar1.png](img%2FCleanShot_xbar1.png)
The image shows the status output from two icinga servers. 
![CleanShot_xbar2.png](img%2FCleanShot_xbar2.png)

### Configuration

#### Setup icinga

You need an icinga api user with the following permissions
```json
object ApiUser "bitbar" {
  password = "xxxxx"
  permissions = [ "status/query", "objects/query/*" ]
}
```
See https://icinga.com/docs/icinga-2/latest/doc/12-icinga2-api/#authentication

#### Settings

Create a config file `$HOME/.config/icinga2statuscfg.json` with one or more icinga servers or with one server and different filters.
For location of config file see below.

Remove the "<- ..." in the example

```json
{
  "SERVER": [
    {
        "SERVERDISPLAYNAME": "Sensor - Server XXX", <- name in pulldown menu
        "SERVER1":"intern.server1.example.com",
        "SERVER2":"server1.example.com", <- alternate server name or IP if necessary, e.g. for VPN
        "PORT1":443,
        "PORT2":5665,            <- port for alternate server
        "APIURL": "https://{server}:{port}", <- API-URL (use python format with SERVER PORT from above)
        "BASEURL": "https://{server}:{port}/icingaweb2/monitoring", <- baseURL for links
        "USER":"bitbar",
        "PASSWORD":"xxxxx",
        "VERIFY":true,           <- Check TLS-Certificate (recommended)
        "SHOW_SERVICES_UP":true, <- show service details for services in state UP
        "SHOW_HOSTS_UP":true,    <- show host details for services in state UP
        "OUTPUTFILE": false,     <- debug, insert "path/filename" instead false
        "ADDFILTER": ""          <- filter for icinga API request, see below
    },
    {
       ... repeat for additional icinga servers
    }
  ],
" GLOBAL": {
    "SHOW_OK_DETAILS": true    <- global switch, enable or disable details
  }
}
```

#### Config file location

You can set a different location by changing the `config_file` line:
```python
# Modify the line below if necessary, e.g.
config_file = '.config/icinga2statuscfg.json'
```
This line describes the path to the configuration file. There are two storage location modes: relative to homedir or relative to plugin.
Default is relative to homedir, the `config_file` is appended to homedir path. 
See `main` function:

```python
def main():
   # ...
   # relative to homedir
   with open(Path().home() / config_file) as json_data_file:
        cfgdata = json.load(json_data_file)
   # or relative to plugin
   with open(Path().absolute() / config_file) as json_data_file:
        cfgdata = json.load(json_data_file)
```
#### Filter
To filter the output of the plugin to include only the necessary information, use the REST API  **advanced filter** for services and host. See
https://icinga.com/docs/icinga-2/latest/doc/12-icinga2-api/#advanced-filters
Please note that the filter will be appended to an existing one. So use “&&” to specialize the output.  

Filter examples to display host.groups from Icinga:
- `"ADDFILTER": "&&\"notify-group1\" in host.groups"`
- `"ADDFILTER": "&&\"ZFS Hosts\" in host.groups"`

#### other

If `VERIFY` for TLS certificate is `False` you may have to disable SSL/TLS warnings too. Add this line
```python
import requests, json, urllib.parse
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
```

