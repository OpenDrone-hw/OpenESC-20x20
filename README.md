# OpenESC-20x20

Open source 4-in-1 BLDC ESC with a 20 x 20 mm mounting pattern, part of the
incutec OpenDrone line. Four independent motor controllers, each with its own
MCU, gate driver and six MOSFETs, running AM32 and taking DShot over the
standard 8-pin connector.

<p>
<img src="images/front.png" width="400" alt="OpenESC-20x20 top" />
<img src="images/back.png" width="400" alt="OpenESC-20x20 bottom" />
</p>

[![Status](https://img.shields.io/endpoint?url=https://opendrone.be/api/status/OpenESC-20x20.json)](https://github.com/OpenDrone-hw/.github/blob/main/CONTRIBUTING.md#the-life-of-a-project)
[![Shop](https://img.shields.io/badge/shop-opendrone.be-ffb700)](https://opendrone.be/products/openesc)
[![Discord](https://img.shields.io/badge/Discord-%23esc-5865F2?logo=discord&logoColor=white)](https://discord.com/channels/1494019459822653512/1494782966302507118)
[![Video](https://img.shields.io/badge/YouTube-How%20Drone%20ESCs%20Work-FF0000?logo=youtube&logoColor=white)](https://www.youtube.com/watch?v=TwAmmPxOpTM)
[![OSHWA](https://img.shields.io/badge/OSHWA-BE000028-0099b0)](https://certification.oshwa.org/be000028.html)

Maintained by [@Just4Stan](https://github.com/Just4Stan).

## Specifications

| | |
|---|---|
| Continuous | 40 A / channel |
| Firmware | AM32 |
| ESC protocol | DShot, bidirectional |
| Telemetry | Extended DShot |
| Input | 2-6S LiPo (6.0-25.2 V) |
| BEC | None |
| MCU | One per motor |
| MOSFETs | 6 per motor |
| Current sense | On-board, 165 A |
| FC connector | JST-SH 8-pin |
| Mounting | 20 x 20 mm, 3.0 mm holes |
| Dimensions | 31.2 x 33.0 mm |
| PCB | 6-layer, 1.6 mm, 2 oz copper |

Technical write-up, part list and layout constraints: [AGENTS.md](AGENTS.md).

## In the line

What pairs with what, and what is available:
[opendrone.be](https://opendrone.be).

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Hardware licensed under [CERN-OHL-S-2.0](https://ohwr.org/cern_ohl_s_v2.txt),
see [LICENSE](LICENSE).
