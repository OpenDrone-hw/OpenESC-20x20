# OpenESC-20x20

Open source 4-in-1 BLDC ESC with a 20 x 20 mm mounting pattern, part of the
incutec OpenDrone line. Four independent motor controllers, each with its own
MCU, gate driver and six MOSFETs, running AM32 and taking DShot over the
standard 8-pin connector.

<p>
<img src="images/front.png" width="400" alt="OpenESC-20x20 top" />
<img src="images/back.png" width="400" alt="OpenESC-20x20 bottom" />
</p>

|  |  |
|---|---|
| Size | 20 x 20 mm mounting pattern, 4x M2 with grommets |
| Input | 6S |
| Interface | 8-pin JST SH, DShot |
| Firmware | [AM32](https://github.com/am32-firmware/AM32) |

<a href="https://certification.oshwa.org/be000028.html">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="images/oshwa-certified-dark.svg" />
    <img src="images/oshwa-certified.svg" width="160" alt="OSHWA Certified Open Source Hardware, BE000028" />
  </picture>
</a>

Certified open source hardware by the [Open Source Hardware Association](https://www.oshwa.org/),
OSHWA UID **[BE000028](https://certification.oshwa.org/be000028.html)**.

## In the line

- [OpenESC-30x30](https://github.com/OpenDrone-hw/OpenESC-30x30): the same
  design at 30.5 x 30.5 mm and 3S-8S, for larger builds.
- [OpenFC-Lite-Mini](https://github.com/OpenDrone-hw/OpenFC-Lite-Mini): the
  20 x 20 mm flight controller this stacks with. It has no onboard motor
  drivers, so it needs an ESC like this one.

## Get one

[opendrone.be/products/openesc](https://opendrone.be/products/openesc)

Build video: [How Drone ESCs Work (so I built my own)](https://www.youtube.com/watch?v=TwAmmPxOpTM)
on [JustFPV](https://www.youtube.com/@justfpv1432)

## Contributing

Issues and pull requests are welcome on any repo. KiCad files cannot be merged,
so say what you intend to change before you do, on
[Discord](https://discord.gg/v3sWmTcx3R).

The design itself, the part list and the layout constraints are in
[AGENTS.md](AGENTS.md). How everything works:
[CONTRIBUTING.md](CONTRIBUTING.md).

## License

Hardware licensed under [CERN-OHL-S-2.0](https://ohwr.org/cern_ohl_s_v2.txt),
see [LICENSE](LICENSE).
