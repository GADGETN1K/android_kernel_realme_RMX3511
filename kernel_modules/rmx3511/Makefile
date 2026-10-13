#
# Out-of-tree kernel modules for realme C35 (RMX3511, ums9230_nico).
#
# The modules under kernel_modules/ use Unisoc's BSP build flow: each has a
# Makefile that runs "$(MAKE) -C $(BSP_KERNEL_PATH) M=$(BSP_MODULES_OUT)/<ko>
# src=<source dir>" and expects the board's BSP_* variables. This wrapper
# provides those variables and builds the modules stock ships, in dependency
# order (KBUILD_EXTRA_SYMBOLS chains through BSP_MODULES_OUT).
#
# LineageOS builds it as an external module:
#   TARGET_KERNEL_EXT_MODULE_ROOT := kernel/realme/RMX3511/kernel_modules
#   TARGET_KERNEL_EXT_MODULES := rmx3511
# and passes O=, ARCH=, LLVM=1, CC=, CROSS_COMPILE=, INSTALL_MOD_PATH= etc.
# on the command line, so every nested make inherits them.
#

ifeq ($(KERNEL_SRC),)
$(error KERNEL_SRC must point to the kernel source)
endif
ifeq ($(O),)
$(error O must point to the kernel build output)
endif

MODULES_ROOT := $(abspath $(CURDIR)/..)

# Board values. Camera versions come from the source paths embedded in the
# stock 13.F.54 modules (sprd_camera: isp2.6/adpt/qogirl6, sprd_sensor:
# csi2/sprd/receiver_r3p1, sprd_cpp: cpp_lite/hw/lite_r6p0); WCN is the
# external UMW2652 (marlin3_lite) on SDIO, as in the stock ikconfig.
export BSP_KERNEL_PATH := $(abspath $(KERNEL_SRC))
export BSP_MODULES_OUT := $(abspath $(O))/rmx3511_ext_modules
export BSP_KERNEL_VERSION := kernel5.4
export BSP_BUILD_VARIANT := user
export BSP_BOARD_NAME := ums9230_nico
export BSP_BOARD_CAMERA_MODULE_ISP_VERSION := isp2.6
export BSP_BOARD_CAMERA_MODULE_ISP_ADAPT_VERSION := qogirl6
export BSP_BOARD_CAMERA_MODULE_CSI_VERSION := receiver_r3p1
export BSP_BOARD_CAMERA_MODULE_CPP_VERSION := lite_r6p0
export BSP_BOARD_CAMERA_MODULE_DVFS := qogirl6
export BSP_BOARD_UNISOC_WCN_SOCKET := sdio
export MALI_PLATFORM_NAME := qogirl6
export TARGET_BOARD_PLATFORM := qogirl6
export TARGET_BUILD_VARIANT := user

# This vendor code was written for clang 12 (Android S). realme's top-level
# Makefile builds with a global -Werror, and newer clang turns old-style C
# into errors (5.4 adds -Werror=strict-prototypes,
# implicit-int, implicit-function-declaration; clang 16+ errors on
# int-conversion and incompatible-function-pointer-types by default).
# Keep them as warnings for these modules only; KCFLAGS is appended after
# the kernel's own flags. Missing symbols are still caught by modpost.
export KCFLAGS += \
	-Wno-error \
	-Wno-error=strict-prototypes \
	-Wno-error=implicit-int \
	-Wno-error=implicit-function-declaration \
	-Wno-error=int-conversion \
	-Wno-error=incompatible-function-pointer-types \
	-Wno-error=incompatible-pointer-types \
	-Wno-error=deprecated-non-prototype

# Order matters: a module must come after the ones whose Module.symvers it
# lists in KBUILD_EXTRA_SYMBOLS.
EXT_MODULES := \
	common/camera/flash/flash_drv \
	common/camera/flash/aw3641 \
	common/camera/flash/ocp8135b \
	common/camera/sensor \
	common/camera/core \
	common/camera/cpp \
	common/camera/mmdvfs \
	kernel5.4/gpu/natt \
	kernel5.4/input/fingerprint/microarray \
	kernel5.4/input/touchscreen/focaltech_ft8756_spi \
	kernel5.4/wcn/bluetooth/driver \
	kernel5.4/wcn/fm/driver

.PHONY: modules modules_install clean

modules:
	@mkdir -p $(BSP_MODULES_OUT)
	@set -e; for m in $(EXT_MODULES); do \
		echo "  EXT     $$m"; \
		$(MAKE) -C $(MODULES_ROOT)/$$m modules; \
	done

modules_install:
	@set -e; for m in $(EXT_MODULES); do \
		$(MAKE) -C $(MODULES_ROOT)/$$m modules_install; \
	done

clean:
	rm -rf $(BSP_MODULES_OUT)
