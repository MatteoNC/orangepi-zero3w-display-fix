# Orange Pi Zero 3W DisplayPort / DRM Framebuffer Fix

Tested workaround for DisplayPort output and Linux DRM framebuffer on the **Orange Pi Zero 3W**, based on the **Allwinner A733 / sun60iw2** platform.

## Status

**Tested and working on real Orange Pi Zero 3W hardware.**

The validated setup reaches the following end-to-end result:

`Orange Pi Zero 3W -> USB-C DisplayPort output -> HDMI display -> DRM framebuffer -> /dev/fb0 -> Linux tty console`

The final system was also able to display the NeuraPocket / Amber dashboard directly on an external HDMI monitor.

## Tested environment

* Board: Orange Pi Zero 3W
* SoC/platform: Allwinner A733 / sun60iw2
* Kernel base: Orange Pi Linux 6.6 branch
* Kernel branch: `orange-pi-6.6-sun60iw2`
* Base commit tested: `2ac08e8c7cdc28abbdc5c9a9dd812f887ae9c79f`
* Display path: USB-C DisplayPort to HDMI display
* DRM framebuffer: working
* `/dev/fb0`: working
* Linux tty console: working

## Problem

The DisplayPort link could be established and the display hardware was detected, but normal DRM scanout remained black.

During diagnosis, hardware-level DisplayPort output could already be demonstrated, while normal framebuffer content was still not visible.

The final working kernel contains changes in several distinct areas:

1. DisplayPort Combo PHY initialization.
2. eDP/DisplayPort to Display Engine attachment.
3. Display Engine DE1 / RCQ update handling.
4. DRM fbdev initialization.

## Important attribution

The Combo PHY change:

```c
phy_writew(combo0->phy_reg + (0xE003 << 1), 0x0003);
```

was already reported publicly in Orange Pi Linux issue **#133**.

That discovery is **not claimed by this project** and should be credited to the original issue author.

This repository documents the complete configuration that was validated on real hardware and the additional DRM / Display Engine changes used to obtain working framebuffer output.

## Display Engine workaround

The validated kernel contains a SUN60IW2-specific workaround in the DE1 RCQ path.

Before the normal RCQ update, dirty shadow register blocks are copied through AHB:

```c
if (hwde->id == 1) {
    de_update_ahb(hwde);
}
```

This was the final, less invasive version used in the validated kernel.

An earlier diagnostic version forced all register blocks dirty before the AHB update, but that version is **not** the final implementation.

## DRM framebuffer

The validated kernel enables the DRM DMA fbdev helper:

```c
drm_fbdev_dma_setup(drm, 32);
```

With the working Display Engine path this produced a functional:

```text
/dev/fb0
```

and allowed the Linux tty console to appear on the external display.

## eDP / DE attachment

The validated kernel also contains:

```c
edp_de_attach(drm_edp->desc->hw_id, de_id);
```

after `sunxi_tcon_mode_init()`.

At this stage this repository does **not** claim that every individual modification has been independently proven necessary.

The documented result is the combination actually tested and validated on real hardware.

## Scope

This repository is intended to document a **tested working workaround** and diagnostic findings.

It is not currently presented as a definitive upstream fix or a formally isolated minimal patch.

The original working kernel remains preserved separately as a validated baseline.

## Build-only fixes

During compilation of the Orange Pi kernel tree, additional unrelated build-path corrections were required.

These concern:

* touchscreen Kconfig path
* Cedar VE Makefile include paths

They are not considered part of the DisplayPort/DRM root problem and will be documented separately from the display changes.

## Credits

Orange Pi / Allwinner Linux sources:

`orangepi-xunlong/linux-orangepi`

Combo PHY DP+USB fix:

Orange Pi Linux GitHub issue **#133**, credited to the original issue author.

Additional debugging, integration and hardware validation were performed during development of **NeuraPocket** on Orange Pi Zero 3W.

## Disclaimer

Use these modifications at your own risk.

The changes described here were validated on the specific hardware and software environment documented above. Compatibility with other boards, kernels, adapters or display configurations has not yet been systematically tested.
