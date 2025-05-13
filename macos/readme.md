# Install OpenVPN and Easy-RSA
  ```
  $ brew install openvpn easy-rsa ngrok
  ```

# OpenVPN Note
  To start openvpn now and restart at startup:
  ```
  $ sudo brew services start openvpn
  ```
  Where openvpn files:
  ```
  /usr/local/etc/openvpn
  ```

# Easy-RSA Note
  ```
  $ which easyrsa
  $ /usr/local/bin/easyrsa
  ```

# Enable NAT
  ```
  $ sudo vim enable-nat.sh
  ```
  ```bash
  #!/bin/bash
  # Save this as enable-nat.sh
  
  # Get your main network interface (usually en0 for WiFi or en1 for Ethernet)
  INTERFACE=$(route get default | grep interface | awk '{print $2}')
  
  # Enable NAT
  sudo sysctl -w net.inet.ip.forwarding=1
  sudo pfctl -F all
  cat << EOF | sudo tee /etc/pf.conf
  nat on $INTERFACE from 10.8.0.0/24 to any -> ($INTERFACE)
  pass all
  EOF
  
  sudo pfctl -f /etc/pf.conf
  sudo pfctl -e
  ```
  ```
  $ sudo chmod +x enable-nat.sh
  $ sudo ./enable-nat.sh
  ```
## Important Note: If you rebooted and remember to run ./enable-nat.sh again.

# Init Easy RSA for certs files

## init-pki
  ```
  $ /usr/local/bin/easyrsa init-pki
  ```
  ```
  WARNING!!!

  You are about to remove the EASYRSA_PKI at:
  * /usr/local/etc/pki
  
  and initialize a fresh PKI here.
  
  Type the word 'yes' to continue, or any other input to abort.
    Confirm removal: yes
  
           ******************************************
           * SECOND WARNING - STOP - SECOND WARNING *
           ******************************************
  
    To keep your current 'pki/vars' settings use 'init-pki soft'.
    To keep your current Request files use 'init-pki soft'
    The Requests can then be signed by a new CA (Partial CA renewal)
    To keep your current Easy-RSA TLS Key use 'init-pki soft'
    This private key file is in use by your current VPN.
  
         ** USE OF   'init-pki soft'   IS RECOMMENDED **
  
  
  Type the word 'yes' to continue, or any other input to abort.
    
    WARNING: COMPLETELY DESTROY current PKI (NOT recommended) ?
  
      [yes/NO]: yes
  
  
  Notice
  ------
  'init-pki' complete; you may now create a CA or requests.
  
  Your newly created PKI dir is:
  * /usr/local/etc/pki
  
  Using Easy-RSA configuration:
  * undefined
  ```
  We save the files to "/usr/local/etc/pki" where we will check our certs.

## build-ca
  ```
  $ /usr/local/bin/easyrsa build-ca nopass
  ```
  ```
  You are about to be asked to enter information that will be incorporated
  into your certificate request.
  What you are about to enter is what is called a Distinguished Name or a DN.
  There are quite a few fields but you can leave some blank
  For some fields there will be a default value,
  If you enter '.', the field will be left blank.
  -----
  Common Name (eg: your user, host, or server name) [Easy-RSA CA]:oli
  
  Notice
  ------
  CA creation complete. Your new CA certificate is at:
  * /usr/local/etc/pki/ca.crt
  
  Create an OpenVPN TLS-AUTH|TLS-CRYPT-V1 key now: See 'help gen-tls'
  
  Build-ca completed successfully.
  ```
  We get the ca.crt for openvpn's server.conf.

## gen-dh
  ```
  $ /usr/local/bin/easyrsa gen-dh
  ```
  ```
  Generating DH parameters, 2048 bit long safe prime
  DH parameters appear to be ok.

  Notice
  ------
  
  DH parameters of size 2048 created at:
  * /usr/local/etc/pki/dh.pem
  ```
  We get the dh.pem for openvpn's server.conf

## build-server-full
  ```
  $ /usr/local/bin/easyrsa build-server-full server nopass
  ```
  ```
  Notice
  ------
  Private-Key and Public-Certificate-Request files created.
  Your files are:
  * req: /usr/local/etc/pki/reqs/server.req
  * key: /usr/local/etc/pki/private/server.key
  
  You are about to sign the following certificate:
  
    Requested CN:     'server'
    Requested type:   'server'
    Valid for:        '825' days
  
  
  subject=
      commonName                = server
  
  Type the word 'yes' to continue, or any other input to abort.
    Confirm requested details: yes
  
  Using configuration from /usr/local/etc/pki/121ac4f9/temp.6.1
  Check that the request matches the signature
  Signature ok
  The Subject's Distinguished Name is as follows
  commonName            :ASN.1 12:'server'
  Certificate is to be certified until Apr 16 14:10:49 2027 GMT (825 days)
  
  Write out database with 1 new entries
  Database updated
  
  Notice
  ------
  Inline file created:
  * /usr/local/etc/pki/inline/private/server.inline
  
  
  Notice
  ------
  Certificate created at:
  * /usr/local/etc/pki/issued/server.crt
  ```
  We get the "server" keys for openvpn's server.conf

# build-client-full
  ```
  $ /usr/local/bin/easyrsa build-client-full client1 nopass
  ```
  You can change "client1" to your device name or your others for isolating different client
  ```
  Notice
  ------
  Private-Key and Public-Certificate-Request files created.
  Your files are:
  * req: /usr/local/etc/pki/reqs/client1.req
  * key: /usr/local/etc/pki/private/client1.key
  
  You are about to sign the following certificate:
  
    Requested CN:     'client1'
    Requested type:   'client'
    Valid for:        '825' days
  
  
  subject=
      commonName                = client1
  
  Type the word 'yes' to continue, or any other input to abort.
    Confirm requested details: yes
  
  Using configuration from /usr/local/etc/pki/6b19c899/temp.6.1
  Check that the request matches the signature
  Signature ok
  The Subject's Distinguished Name is as follows
  commonName            :ASN.1 12:'client1'
  Certificate is to be certified until Apr 16 14:10:58 2027 GMT (825 days)
  
  Write out database with 1 new entries
  Database updated
  
  Notice
  ------
  Inline file created:
  * /usr/local/etc/pki/inline/private/client1.inline
  
  
  Notice
  ------
  Certificate created at:
  * /usr/local/etc/pki/issued/client1.crt
  ```
  We get the "client1" keys for openvpn's server.conf and client.conf. The client.conf will be used for your openvpn client profile(*.ovpn).

# Genkey ta.key for client site
  ```
  $ mkdir -pv ~/openvpn_files && cd ~/openvpn_files && /usr/local/opt/openvpn/sbin/openvpn --genkey secret ta.key
  ```
  You can copy ca and client keys to same folder
  ```
  $ cp /usr/local/etc/pki/ca.crt ~/openvpn_files
  $ cp /usr/local/etc/pki/private/client1.key ~/openvpn_files
  $ cp /usr/local/etc/pki/issued/client1.crt
  ```

# Update OpenVPN configuration "server.conf"
  ```
  $ vim /usr/local/etc/openvpn/server.conf
  ```
  Change "YOUR_MAC_NAME" to yours and update server.conf.
  ```
  port 1194
  proto tcp
  dev tun
  ca /usr/local/etc/pki/ca.crt
  cert /usr/local/etc/pki/issued/server.crt
  key /usr/local/etc/pki/private/server.key
  dh /usr/local/etc/pki/dh.pem
  server 10.8.0.0 255.255.255.0
  push "redirect-gateway def1 bypass-dhcp"
  push "dhcp-option DNS 8.8.8.8"
  push "dhcp-option DNS 8.8.4.4"
  keepalive 10 120
  tls-auth /Users/YOUR_MAC_NAME/openvpn_files/ta.key 0
  cipher AES-256-CBC
  auth SHA256
  user nobody
  group nogroup
  persist-key
  persist-tun
  status openvpn-status.log
  verb 3
  max-clients 30
  tcp-nodelay
  comp-lzo
  push "comp-lzo"
  ```


# Gen OpenVPN profile "client1.ovpn"
  Create the file, copy and revise "remote" detail accordingly after setup server side
  ```
  $ vim ~/openvpn_files/client1.ovpn
  ```
  ```
  client
  dev tun
  proto tcp
  remote YOUR_SERVER_IP_or_Ngrok_TCP_DOMAIN YOUR_SERVER_PUBLIC_PORT_or_Ngrok_TCP_PORT
  resolv-retry infinite
  nobind
  user nobody
  group nogroup
  persist-key
  persist-tun
  remote-cert-tls server
  cipher AES-256-CBC
  auth SHA256
  key-direction 1
  verb 3
  
  ca ca.crt
  cert client1.crt
  key client1.key
  tls-auth ta.key 1
  ```
  Zip the folder ~/openvpn_files and shared to your others device, and unzip it, drop the client1.ovpn to OpenVPN client to connect.
  Ensure all keys are same folder.

  
# Start the OpenVPN 
  ```
  $ sudo /usr/local/opt/openvpn/sbin/openvpn --duplicate-cn --config /usr/local/etc/openvpn/server.conf
  ```
  ```
  2025-01-11 23:38:30 WARNING: --topology net30 support for server configs with IPv4 pools will be removed in a future release. Please migrate to --topology subnet as soon as possible.
  2025-01-11 23:38:30 DEPRECATED OPTION: --cipher set to 'AES-256-CBC' but missing in --data-ciphers (AES-256-GCM:AES-128-GCM:CHACHA20-POLY1305). OpenVPN ignores --cipher for cipher negotiations. 
  2025-01-11 23:38:30 OpenVPN 2.6.12 x86_64-apple-darwin21.6.0 [SSL (OpenSSL)] [LZO] [LZ4] [PKCS11] [MH/RECVDA] [AEAD]
  2025-01-11 23:38:30 library versions: OpenSSL 3.4.0 22 Oct 2024, LZO 2.10
  2025-01-11 23:38:30 Diffie-Hellman initialized with 2048 bit key
  2025-01-11 23:38:30 Opened utun device utun3
  2025-01-11 23:38:30 /sbin/ifconfig utun3 delete
  ifconfig: ioctl (SIOCDIFADDR): Can't assign requested address
  2025-01-11 23:38:30 NOTE: Tried to delete pre-existing tun/tap instance -- No Problem if failure
  2025-01-11 23:38:30 /sbin/ifconfig utun3 10.8.0.1 10.8.0.2 mtu 1500 netmask 255.255.255.255 up
  2025-01-11 23:38:30 /sbin/route add -net 10.8.0.0 10.8.0.2 255.255.255.0
  add net 10.8.0.0: gateway 10.8.0.2
  2025-01-11 23:38:30 Could not determine IPv4/IPv6 protocol. Using AF_INET6
  2025-01-11 23:38:30 Socket Buffers: R=[131072->131072] S=[131072->131072]
  2025-01-11 23:38:30 setsockopt(IPV6_V6ONLY=0)
  2025-01-11 23:38:30 Listening for incoming TCP connection on [AF_INET6][undef]:1194
  2025-01-11 23:38:30 TCPv6_SERVER link local (bound): [AF_INET6][undef]:1194
  2025-01-11 23:38:30 TCPv6_SERVER link remote: [AF_UNSPEC]
  2025-01-11 23:38:30 GID set to nogroup
  2025-01-11 23:38:30 UID set to nobody
  2025-01-11 23:38:30 MULTI: multi_init called, r=256 v=256
  2025-01-11 23:38:30 IFCONFIG POOL IPv4: base=10.8.0.4 size=62
  2025-01-11 23:38:30 MULTI: TCP INIT maxclients=1019 maxevents=1024
  2025-01-11 23:38:30 Initialization Sequence Completed
  ```

# Optional - Ngrok expose
  ```
  $ ngrok tcp 1194
  ```
  ```
  Session Status                online                                                                    
  Version                       3.19.0                                                                    
  Region                        Asia Pacific (ap)                                                         
  Latency                       45ms                                                                      
  Web Interface                 http://127.0.0.1:4040                                                     
  Forwarding                    tcp://xxx:xxx -> localhost:1194                           
                                                                                                          
  Connections                   ttl     opn     rt1     rt5     p50     p90                               
                                2       1       0.00    0.00    552.76  1105.51
  ```
  
# Optional - SSH Reverse Tunnel
  You can rent a external server from AWS, digital ocean etc, and we go to set by SSH proxy way to do.
  Go to the server and gen ssh key and press Enter to accept defaults unless you need a custom key location or passphrase.
  ```
  $ ssh-keygen
  ```
  Go back local macbook which installed the OpenVPN, and copy the public key of the server for passwordless login.  
  ```
  ssh-copy-id non-root-user@your_server_ip
  ```
  Try the ssh command on the local macbook with the custom external port you like on the server. Ctrl+C to stop and exit.
  ```
  ssh -N -R 0.0.0.0:external_port:localhost:1194 non-root-user@your_server_ip -p 22
  ```
  It doesn't output something, just running on the frontground.
  We go to the external server to check the connection and whether the custom port is listening.
  ```
  netstat -tuln | grep external_port
  tcp        0      0 0.0.0.0:external_port            0.0.0.0:*               LISTEN     
  tcp6       0      0 :::external_port                 :::*                    LISTEN 
  ```
  If it is not showing like above, we go to change the ssh config setting and set `GatewayPorts` to `yes`.
  ```
  sudo nano /etc/ssh/sshd_config
  ```
  Save and restart by the command.
  ```
  sudo systemctl restart sshd
  ```
  Check if a firewall is active (e.g., UFW on Ubuntu):
  ```
  sudo ufw status
  ```
  If it’s enabled and doesn’t list `external_port`, allow it:
  ```
  sudo ufw allow external_port/tcp
  ```
  If you’re using iptables instead:
  ```
  sudo iptables -L -n
  ```
  Add a rule if needed:
  ```
  sudo iptables -A INPUT -p tcp --dport external_port -j ACCEPT
  ```
  Verify the port is open from outside using a tool like nc (netcat) from another machine:
  ```
  nc -zv vps_ip 7022
  ```
  If it says “connection succeeded,” the port is reachable.






  
  
