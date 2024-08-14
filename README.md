# OpenVPN client skeleton

I use this folder to start/stop an [OpenVPN](https://en.wikipedia.org/wiki/OpenVPN) client connection with the following constraints:

- I use the DNS server present in the private network only for a certain type of subdomain (here example.com).
- the VPN is used only for servers inside the private network. For example, I don't use this VPN to access the Internet.
- the entire configuration is file-based and scripted (GitOps practice).
- OpenVPN client is managed by SystemD.

## Configuration

- Put your certificates in `ca.pem` and `tls-crypt.pem` file
- Fill `auth.txt` file

### Second dns server

Here's how to configure a dns server used only for a particular domain (here `*.example`).

```
$ sudo mkdir -p /etc/systemd/resolved.conf.d
$ cat <<EOF | sudo tee /etc/systemd/resolved.conf.d/example.com.conf > /dev/null
[Resolve]
DNS=10.57.40.1
Domains=~example
EOF
```

```
$ sudo systemctl restart systemd-resolved
```

### Using systemd to start, stop OpenVPN client

```
$ cat <<EOF | sudo tee /etc/systemd/system/openvpn-tcp.service > /dev/null
[Unit]
Description=OpenVPN connection to OpenVPN TCP
After=network.target

[Service]
ExecStart=/usr/sbin/openvpn --config $(pwd)/tcp.ovpn
ExecReload=/bin/kill -HUP \$MAINPID
Restart=on-failure
WorkingDirectory=$(pwd)

[Install]
WantedBy=multi-user.target
EOF
```

```
$ cat <<EOF | sudo tee /etc/systemd/system/openvpn-udp.service > /dev/null
[Unit]
Description=OpenVPN connection to OpenVPN TCP
After=network.target

[Service]
ExecStart=/usr/sbin/openvpn --config $(pwd)/udp.ovpn
ExecReload=/bin/kill -HUP \$MAINPID
Restart=on-failure
WorkingDirectory=$(pwd)

[Install]
WantedBy=multi-user.target
EOF
```

```
$ sudo systemctl daemon-reload
```

## Start / stop instructions

```
$ ./start.sh
```

```
$ ./stop.sh
```
