# VPNed container

This is an example of how to run containers through the VPN connection of other container with periodic IP rotation.

[docker.io/qmcgaw/gluetun](https://github.com/qdm12/gluetun) image is used as a VPN client. It supports multiple VPN providers and has API which can be used to reconnecting to the VPN obtaining a new IP each time. As an IP rotation tool I used basic shell script with `while` loop sleeping and calling API with curl (you can bake this into other other image if needed).

Set rotation period for the VPN reconnecting with `VPN_ROTATE_MINUTES` variable.

By default it is set to 12 minutes in the `.env` file.

## Preparation

1. Create `.env` file from the provided example: `cp .env.example .env`.
2. Edit `.env` file replacing `put_your_value_here` string with appropriate values.
3. Put [certificates](cyberghost/README.md) into the VPN provider directory.

## Run the project

Start the project: `docker-compose up -d`

Stop the project:  `docker-compose down`

Stream the logs:   `docker-compose logs --tail 20 -f`

Restart VPN (for obtaining a new IP):

```
docker-compose exec vpnrotate curl -s http://127.0.0.1:8000/v1/publicip/ip
docker-compose exec vpnrotate curl -s -X PUT -d '{"status":"stopped"}' http://127.0.0.1:8000/v1/openvpn/status
docker-compose exec vpnrotate curl -s -X PUT -d '{"status":"running"}' http://127.0.0.1:8000/v1/openvpn/status
docker-compose exec vpnrotate curl -s http://127.0.0.1:8000/v1/publicip/ip
```

## Run multiple instances (for having few outgoing IPs)

Few instances of the project can be run by specifying a unique project name via `-p` option:

```
docker-compose -p ddos1 up -d
docker-compose -p ddos2 up -d
docker-compose -p ddos3 up -d
docker-compose -p ddos4 up -d
```

One-liner for the same with the 3min interval between each instance start:

```
for i in `seq 4`; do docker-compose -p ddos$i up -d; sleep 3m; done
```

One-liner for stopping all of the above:

```
for i in `seq 4`; do docker-compose -p ddos$i down; done
```
