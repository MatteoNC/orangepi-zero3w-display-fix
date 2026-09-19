# Kernel patches

Patch set tested on real Orange Pi Zero 3W hardware based on the Allwinner A733 / sun60iw2 platform.

These files reproduce the changes present in the validated working kernel used during NeuraPocket development.

## Patch order

### 01-combophy-issue133.patch

Combo PHY initialization change required for the tested DisplayPort path:

```c
phy_writew(combo0->phy_reg + (0xE003 << 1), 0x0003);
This modification enables the required PLL1 clock configuration in the Combo PHY path used for DisplayPort + USB operation.

This specific fix was already publicly reported in Orange Pi Linux issue #133.

Credit for the original discovery belongs to the author of issue #133.

This repository does NOT claim authorship of this modification.

The patch is included here because it is part of the complete kernel configuration that was successfully tested on real Orange Pi Zero 3W hardware.

Without this patch, the tested DisplayPort configuration did not represent the final validated kernel baseline.

The purpose of including it in this repository is therefore reproducibility of the complete tested configuration, while preserving clear attribution to the original author.

02-edp-de-attach.patch

Adds:

edp_de_attach(drm_edp->desc->hw_id, de_id);

after:

sunxi_tcon_mode_init()

This modification is present in the final validated working kernel.

Its individual necessity has NOT yet been independently demonstrated.

For this reason, it should currently be considered part of the tested working configuration rather than a proven standalone fix.

03-de1-rcq-dirty-ahb.patch

Display Engine workaround used on DE1 during RCQ updates:

if (hwde->id == 1) {
    de_update_ahb(hwde);
}

During debugging, the DisplayPort link and hardware output were already operational while normal DRM plane contents remained black.

In the final validated kernel, dirty shadow register blocks are copied through AHB on DE1 before the normal RCQ update is issued.

An earlier diagnostic version forced all register blocks to be marked dirty before the AHB update.

That version was discarded.

The final working implementation only copies register blocks that are already marked dirty.

This is the less invasive implementation that was present in the validated kernel.

04-drm-fbdev.patch

Enables the DRM DMA fbdev helper:

drm_fbdev_dma_setup(drm, 32);

The validated system obtained a working:

/dev/fb0

and Linux tty output on the external display.

This allowed the NeuraPocket / Amber dashboard to be displayed directly on a connected HDMI monitor through the validated DisplayPort-to-HDMI output path.

The patch also restores:

.output_poll_changed = drm_fb_helper_output_poll_changed,

and adds error handling for:

drm_dev_register()

The complete file reflects the configuration present in the validated working kernel.

90-build-only-fixes.patch

Contains build-path corrections that were required while compiling the Orange Pi kernel tree used for testing.

These changes concern:

touchscreen Kconfig path;
Cedar VE Makefile include path;
Cedar VE codec Makefile path.

These modifications are NOT considered part of the DisplayPort / DRM display fix itself.

They are kept in a separate patch to avoid confusing build-environment corrections with the actual display-related changes.

Tested result

The complete validated kernel configuration achieved the following path:

Orange Pi Zero 3W
        |
        v
USB-C DisplayPort output
        |
        v
HDMI display
        |
        v
DRM scanout
        |
        v
/dev/fb0
        |
        v
Linux tty console
        |
        v
NeuraPocket / Amber dashboard

The display output was successfully tested on real Orange Pi Zero 3W hardware.

The validated setup is not tied to one specific monitor model and was designed for standard HDMI displays connected through the working DisplayPort-to-HDMI path.

Important scope note

This patch set documents the exact combination of modifications present in a kernel that was successfully tested on real Orange Pi Zero 3W hardware.

At this stage it is intentionally described as a:

tested working workaround

and NOT as a:

formally isolated minimal upstream fix

No claim is made that every patch in this directory is individually required.

Some modifications may prove unnecessary once each component is independently isolated and tested.

The purpose of this repository is to make the known working configuration available to the Orange Pi community while clearly separating:

previously published fixes;
display-related modifications;
framebuffer initialization;
build-only corrections.
Attribution

The Combo PHY change contained in:

01-combophy-issue133.patch

was previously reported in Orange Pi Linux issue #133.

Credit for that discovery belongs to the original issue author.

The additional debugging, Display Engine investigation, integration work and hardware validation documented here were performed during development of NeuraPocket on the Orange Pi Zero 3W.

Baseline

Kernel repository:

orangepi-xunlong/linux-orangepi

Kernel branch:

orange-pi-6.6-sun60iw2

Base commit used during development:

2ac08e8c7cdc28abbdc5c9a9dd812f887ae9c79f

Hardware:

Orange Pi Zero 3W
Allwinner A733 / sun60iw2

The original validated NeuraPocket kernel remains preserved separately as a frozen working baseline.

Disclaimer

Use these patches at your own risk.

The changes documented here were tested on the specific Orange Pi Zero 3W hardware and software environment described above.

Compatibility with other Orange Pi boards, kernel versions, USB-C adapters, DisplayPort configurations or display hardware has not been systematically validated.

Feedback, independent testing and technical review from the Orange Pi community are welcome.
