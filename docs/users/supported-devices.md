---
title: Supported devices
---

Felix86 requires `rv64gv_zba_zbb_zbc_zbs` to work. RVA23 supports all of these extensions, so boards with RVA23 such as future SpacemiT K3 boards will be supported.

SpacemiT K1 boards are also supported.

Some examples:

- Banana Pi F3 (has mPCIe slot)
- Milk-V Jupiter (has PCIe slot)
- DC-ROMA II
- Muse Pi Pro
- Orange Pi RV2

A board with a PCIe slot is preferred for connecting a discrete GPU.

## Devices that don't support felix86

Devices with no vector extension (RVV 1.0), or with the old "XTheadVector"/RVV 0.7.1 won't be supported.

Devices with no bit-manipulation extensions are no longer supported. This is to reduce maintenance burden and because any upcoming core that is RVA23 will have bit-manipulation extensions and non-RVA23 SoCs like SpacemiT K1 have bit-manipulation extensions.

Some instructions may assume you're running on VLEN < 2048 hardware for better translations. Any possible future devices with higher VLEN are not supported.