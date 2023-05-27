1. Install OpenVPN with https://github.com/mitrapartha/openvpn-install/blob/master/openvpn-install.sh

At this point you should be able to download the .ovpn file to a client and connect (using solely the certificate for authentication).

That’s the easy bit, now let’s tackle the tricky bit!

2. Login to https://portal.azure.com/ and select Azure Active Directory, App Registrations then New Registration.

From the Authentication tab enable the “Treat application as a public client” option.

From the API Permissions tab click Add Permission and select Azure Active Directory Graph from the bottom (Supported legacy APIs) section then Directory.Read.All.

3. Download my modified version of openvpn-azure-ad-auth.py from https://github.com/mitrapartha/openvpn-azure-ad-auth/blob/master/openvpn-azure-ad-auth.py to /etc/openvpn/config

4. Create /etc/openvpn/config/openvpn-azure-ad-auth.yaml following the instructions in README.md)- I strongly recommend NOT enabling token_cache as it allowed me to connect to the VPN without a password in certain scenarios.

5. Add the debian repositories to package manager;
```
set system package repository stretch components 'main contrib non-free'
set system package repository stretch distribution stretch
set system package repository stretch url http://http.us.debian.org/debian
```
The install the prerequisites by running;
```
sudo apt-get update
sudo apt-get install python-pyparsing
sudo apt-get install python-six
sudo apt-get install python-appdirs
sudo apt-get install python-yaml
sudo apt-get install python-requests
sudo apt-get install python-adal
sudo apt-get install python-pbkdf2
```
6. Activate the script by running;
```
./openvpn-azure-ad-auth.py --consent
```
And follow the on-screen instructions.

7. Finally tweak the EdgeRouter config to use the python script;
```
set interfaces openvpn vtun0 openvpn-option "--duplicate-cn"
set interfaces openvpn vtun0 openvpn-option "--auth-user-pass-verify /etc/openvpn/config/openvpn-azure-ad-auth.py via-env"
set interfaces openvpn vtun0 openvpn-option "--script-security 3"
```
And you should be good to go!

If you experience any issues later, set the log_level to debug and check /etc/openvpn/config/openvpn-azure-ad-auth.log (you can also try issuing show log | grep openvpn)
