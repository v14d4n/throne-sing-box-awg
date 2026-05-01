# throne-sing-box-awg

AWG protocol support for [throneproj/sing-box](https://github.com/throneproj/sing-box), based on [hoaxisr/amnezia-box](https://github.com/hoaxisr/amnezia-box).

To implement this in [Throne](https://github.com/throneproj/Throne) you need to build the core server with custom dependency. Clone the [throneproj/Throne](https://github.com/throneproj/Throne) repository and comment the original `sing-box` replacement in `core/server/go.mod` and add the following:
```go
//replace github.com/sagernet/sing-box => github.com/Throneproj/sing-box v1.11.16-0.20260423092808-ed3cd5614f33

replace github.com/sagernet/sing-box => github.com/v14d4n/throne-sing-box-awg v1.11.16-0-awg
```

In the `core/server/gen` directory generate proto files:
```bash
protoc --go_out=. --go_opt=paths=source_relative --go-grpc_out=. --go-grpc_opt=paths=source_relative ./*.proto
```

Then, build the server inside `core/server` and replace the original `ThroneCore` binary in your Throne installation path:
```bash
go build -tags "with_gvisor with_quic with_dhcp with_wireguard with_utls with_acme with_clash_api with_tailscale with_ccm with_ocm with_awg"
```

### Imporatnt note
This solution does not include patches for [amneziawg-go](https://github.com/amnezia-vpn/amneziawg-go) from [hoaxisr/amnezia-box](https://github.com/hoaxisr/amnezia-box). You can apply them manually by vendoring the core server dependencies and running the `patches/amneziawg-go/apply.sh` script from [hoaxisr/amnezia-box](https://github.com/hoaxisr/amnezia-box).

### Example config
To use AWG, create a profile with a custom sing-box configuration:
```json
{
  "certificate": {
    "store": "system"
  },
  "dns": {
    "rules": [
      {
        "action": "predefined",
        "answer": "localhost. IN A 127.0.0.1",
        "domain": "localhost",
        "query_type": "A",
        "rcode": "NOERROR"
      },
      {
        "action": "predefined",
        "answer": "localhost. IN AAAA ::1",
        "domain": "localhost",
        "query_type": "AAAA",
        "rcode": "NOERROR"
      },
      {
        "action": "route",
        "server": "dns-remote",
        "strategy": ""
      }
    ],
    "servers": [
      {
        "detour": "proxy",
        "domain_resolver": "dns-local",
        "server": "8.8.8.8",
        "tag": "dns-remote",
        "type": "tls"
      },
      {
        "domain_resolver": "dns-local",
        "tag": "dns-direct",
        "type": "local"
      },
      {
        "tag": "dns-local",
        "type": "local"
      }
    ]
  },
  "endpoints": [
    {
      "address": [
        "..."
      ],
      "mtu": 1280,
      "jc": ...,
      "jmin": ...,
      "jmax": ...,
      "s1": ...,
      "s2": ...,
      "s3": ...,
      "s4": ...,
      "h1": "...-...",
      "h2": "...-...",
      "h3": "...-...",
      "h4": "...-...",
      "i1": "base64 string",
      "peers": [
        {
          "address": "...",
          "allowed_ips": [
            "0.0.0.0/0",
            "::/0"
          ],
          "persistent_keepalive_interval": 25,
          "port": ...,
          "preshared_key": "...",
          "public_key": "..."
        }
      ],
      "private_key": "...",
      "tag": "proxy",
      "type": "awg"
    }
  ],
  "experimental": {
    "cache_file": {
      "enabled": true,
      "store_fakeip": true,
      "store_rdrc": true
    },
    "clash_api": {
      "default_mode": ""
    }
  },
  "inbounds": [
    {
      "listen": "127.0.0.1",
      "listen_port": 5533,
      "tag": "dns-in",
      "type": "direct"
    },
    {
      "listen": "127.0.0.1",
      "listen_port": 2080,
      "tag": "mixed-in",
      "type": "mixed"
    },
    {
      "address": [
        "172.19.0.1/24"
      ],
      "auto_route": true,
      "interface_name": "throne-tun",
      "mtu": 1500,
      "route_exclude_address": [
        "127.0.0.0/8"
      ],
      "stack": "gvisor",
      "strict_route": true,
      "tag": "tun-in",
      "type": "tun"
    }
  ],
  "log": {
    "level": "info"
  },
  "outbounds": [
    {
      "tag": "direct",
      "type": "direct"
    }
  ],
  "route": {
    "auto_detect_interface": true,
    "default_domain_resolver": {
      "server": "dns-direct",
      "strategy": ""
    },
    "final": "proxy",
    "find_process": true,
    "rule_set": [],
    "rules": [
      {
        "action": "sniff",
        "inbound": "dns-in"
      },
      {
        "action": "hijack-dns",
        "inbound": "dns-in",
        "protocol": "dns"
      },
      {
        "action": "reject",
        "inbound": "dns-in"
      },
      {
        "action": "sniff",
        "inbound": [
          "mixed-in",
          "tun-in"
        ]
      },
      {
        "action": "hijack-dns",
        "protocol": "dns"
      }
    ]
  }
}
```

# sing-box

The universal proxy platform.

[![Packaging status](https://repology.org/badge/vertical-allrepos/sing-box.svg)](https://repology.org/project/sing-box/versions)

## Documentation

https://sing-box.sagernet.org

## License

```
Copyright (C) 2022 by nekohasekai <contact-sagernet@sekai.icu>

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program. If not, see <http://www.gnu.org/licenses/>.

In addition, no derivative work may use the name or imply association
with this application without prior consent.
```
