# OpenVPN3 Ubuntu Linux.
This document will show you how to create a session, verify the connection 
and then close the session on Ubuntu Linux.
___

### Start Session
`<file>.ovpn` is the configuration file.  
You either need to use the full path or the current working directory.
```shell
openvpn3 session-start --config siteautomation-config.ovpn
```
If the command above works, you should see the following output example:  
> Connected to 47.44.60.229 (47.44.60.229)`
___

### Check Connection Status
Used the following command to verify all maintained and established connections:  
```shell
openvpn3 sessions-list
```
___

### Disconnect Connection
Used the following command to disconnect the connection:    
```shell
openvpn3 session-manage --config siteautomation-config.ovpn --disconnect
```

