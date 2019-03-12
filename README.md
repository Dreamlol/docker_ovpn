OpenVPN server in a Docker container complete with an EasyRSA PKI CA.

## Quick Start

* Build docker image from Dockerfile or pull from Gitlab Docker registry (before login into registry)

      docker build -t openvpn -f path_to_dockerfile .
      or
      docker pull git-pub.intecracy.com:4567/softengi-re/iothic/backend/openvpn \
      && git tag git-pub.intecracy.com:4567/softengi-re/iothic/backend/openvpn openvpn

* Initialize the "ovpn_data" container volume that will hold the configuration files
  and certificates.  The container will prompt for a passphrase to protect the
  private key used by the newly generated certificate authority.

      docker volume create --name ovpn_data
      docker run -v ovpn_data:/etc/openvpn --log-driver=none --rm openvpn ovpn_genconfig -u "udp://server_ip_or_domain" -b -d -D -t -c -d -l "local_ip"
      docker run -v ovpn_data:/etc/openvpn --log-driver=none --rm -it openvpn ovpn_initpki

* Start OpenVPN server process

      docker run --restart=always -v ovpn_data:/etc/openvpn -d -p 1194:1194/udp --cap-add=NET_ADMIN openvpn

* If you want to attach container on host network

      docker run --restart=always -v ovpn_data:/etc/openvpn -d --network=host --cap-add=NET_ADMIN openvpn  

* Generate a client certificate without a passphrase

      docker run -v ovpn_data:/etc/openvpn --log-driver=none --rm -it openvpn easyrsa build-client-full CLIENTNAME nopass

      !You must define the number of tap interface which to be used on config file!

* Retrieve the client configuration with embedded certificates

      docker run -v ovpn_data:/etc/openvpn --log-driver=none --rm openvpn ovpn_getclient CLIENTNAME > CLIENTNAME.ovpn

* Run openvpn as daemon on client machine:

      openvpn --config CLIENTNAME.ovpn --daemon
