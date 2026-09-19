# Kernel patches

Patch set tested on a real Orange Pi Zero 3W (Allwinner A733 / sun60iw2).

These files reproduce the changes present in the validated working kernel used during NeuraPocket development.

## Patch order

### 01-combophy-issue133.patch

Combo PHY initialization change:

```c
phy_writew(combo0->phy_reg + (0xE003 << 1), 0x0003);
