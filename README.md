# Linux camera raw mode bits-per-pixel fixup patches

This set of patches adds a workaround for UVC cameras which advertise
incorrect bits-per-pixel values for a raw image format, and applies
said workaround to cameras based on the GEO Semiconductor GC6500 chip,
including the MDG-217 camera assembly.

## Motivation

The above mentioned camera assembly has adequate raw mode support
before Linux kernel 4.19. However, newer kernels contain the
[patch](https://github.com/torvalds/linux/commit/95f5cbff90b9e4324839a5c28ee3153a3c9921a5)
imposing stricter checks on the camera's behaviour, which fail as the
frame size calculated by the kernel doesn't match one sent by the
camera.

Disabling or relaxing the check (e.g. by reverting the
[patch](https://github.com/torvalds/linux/commit/95f5cbff90b9e4324839a5c28ee3153a3c9921a5)
or turning equality comparison into an inequality) makes the camera
work (because the problem is with kernel's calculations of the frame
size, not the actual frames being sent), but is not the best solution
for a number of reasons.

Kernel's calculations are ultimately based on the bits-per-pixel value
for the format, which is announced by the camera incorrectly. However,
the kernel is capable of deriving the bits-per-pixel value from the
(correctly announced) raw format fourcc. Therefore, we add a quirk
enabling it to do so, and indicate that the camera model has said
quirk.

## Patch set structure and organization

Patches are organized in the `camera-bpp` subdirectory, containing
subdirectories named after stable Linux kernel versions the patch
series is expected to be applied to (possibly empty). For each given
kernel version, only one nearest-matching item needs to be applied.

Kernel versions not containing the
[patch](https://github.com/torvalds/linux/commit/95f5cbff90b9e4324839a5c28ee3153a3c9921a5)
(4.18 and below) needn't be patched for the camera to work in raw
mode.

Kernel versions below 5.2 need a version of
[the commit enabling obtaining bits-per-pixel info for format](https://github.com/torvalds/linux/commit/f44b969aa3cddabe86e9aff411889643298e8d59),
included in the related subdirectories, for the bits-per-pixel fixup
patches to build on.

## Configuration

No user-configurable options are introduce by this patch set; however,
other camera models with the same problem can be handled by adding
their description with the `UVC_QUIRK_FORCE_BPP` flag in `quirks` to
`uvc_ids[]`.
