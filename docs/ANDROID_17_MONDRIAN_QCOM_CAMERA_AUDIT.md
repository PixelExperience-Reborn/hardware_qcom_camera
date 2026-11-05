# Android 17 / PixelExperience-Reborn — legacy Qualcomm camera source review

**Date:** 2026-11-05  
**135-repository matrix row:** P2 `hardware/qcom/camera`  
**Decision:** preserve history; **do not** enable this msm8998 HAL as the mondrian camera provider.

## Source and platform facts

- The historic PixelExperience tree `seventeen` points to `ce8bf32a102e8b6ddc473016cfcc427df7d7a261`; its files are `msm8998/QCamera2`, `msm8998/mm-image-codec`, `msm8998/usbcamcore` and Makefiles.
- The repository's top-level `Android.mk` includes the `msm8998` subdirectory **only** when `TARGET_BOARD_PLATFORM` matches `msm8998` (with further stub/tiny/VR guards). This is not a generic modern Qualcomm camera source.
- `msm8998/QCamera2/Android.mk` uses older SoC lists and can build `camera.$(TARGET_BOARD_PLATFORM)` with dependencies including `libmmcamera_interface`, `libmmjpeg_interface`, `libqdMetaData`, legacy HIDL power, `android.hardware.power-V1-ndk` and additional vendor libraries. It cannot be assumed ABI-compatible with a Taro camera provider.
- The actual `LF52406/mondrian_android_device_xiaomi_sm8450-common` `lineage-24.0` `BoardConfigCommon.mk` sets `TARGET_BOARD_PLATFORM := taro`. `common.mk` configures `libcameraservice`'s `ext_lib` as `libcameraservice_extension.xiaomi_sm8450`.
- The reference device's `camera/Android.bp` defines that extension as a `cc_library_static` using `camera/CameraProviderExtension.cpp`, including the `frameworks/av/services/camera/libcameraservice/common` header interface. The extension includes torch-strength controls, hardware LED paths, and private framework hooks; its Android 17 C++/Soong API and device sysfs paths **must be checked independently**.
- The Android 17 AOSP reference manifest `aosp17_reference.xml` contains **no** `hardware/qcom/camera` project to replace. The active Android17 platform graph uses its official camera framework/interfaces/CameraService modules; these do not substitute for Xiaomi camera vendor blobs and provider support.

## Why an automatic 'Android 17 upgrade' would be incorrect

There is no safe one-to-one upgrade by cherry-picking old QCamera2 HAL sources or just changing its SoC list to `taro`. Modern camera behavior crosses CameraService, gralloc/buffer handling, camera metadata, vendor provider interfaces and Xiaomi's proprietary camera implementation. Copying the old HAL would risk duplicate provider modules, incompatible HIDL/AIDL declarations, incompatible metadata/buffers, broken torch and camera stability.

Historical fixes to `-Wstring-concatenation` and `display_headers` are scoped to legacy msm8998 code. Reapply only when the same defect is **actually identified** in the modern code consuming the relevant interfaces.

## Required acceptance gates for the independent PE-Reborn A17 device graph

1. Keep verified official AOSP17 `frameworks/av`, `frameworks/native`, `hardware/interfaces` and Camera2 paths; no old global camera framework override.
2. Establish the exact module and VINTF owners of modern Xiaomi camera provider/vendor services and dependencies; inspect blobs and kernel camera-driver ABI (including userspace-kernel UAPI).
3. Rework or selectively reuse the **separate** `libcameraservice_extension.xiaomi_sm8450` where compatible with Android17 CameraService source and callback hooks. Retain torch safety, current permissions and SELinux enforcing.
4. Run Android17 Soong module compilation, `checkvintf`, SELinux neverallow checks, Camera CTS/VTS and physical mondrian tests: main/wide/front cameras, flash/torch strength, video, HDR, buffer fences, permissions, camera switching and sustained-load operation.
5. Keep historical source unmerged/unused unless a **verified module consumer** specifically requires it. Do not edit working MistOS/LineageOS device trees during platform audit.

**Evidence status:** source paths, Make gates and real device hooks reviewed. No complete Android 17 build or on-device Camera test performed. Do not mark ROM camera support complete. Next matrix item: `hardware/qcom/data/ipacfg-mgr`.
