#*******************************************************************************
# Copyright (C) Maxim Integrated Products, Inc., All Rights Reserved.
#
# Permission is hereby granted, free of charge, to any person obtaining a
# copy of this software and associated documentation files (the "Software"),
# to deal in the Software without restriction, including without limitation
# the rights to use, copy, modify, merge, publish, distribute, sublicense,
# and/or sell copies of the Software, and to permit persons to whom the
# Software is furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included
# in all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
# OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
# MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
# IN NO EVENT SHALL MAXIM INTEGRATED BE LIABLE FOR ANY CLAIM, DAMAGES
# OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE,
# ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR
# OTHER DEALINGS IN THE SOFTWARE.
#
# Except as contained in this notice, the name of Maxim Integrated
# Products, Inc. shall not be used except as stated in the Maxim Integrated
# Products, Inc. Branding Policy.
#
# The mere transfer of this software does not imply any licenses
# of trade secrets, proprietary technology, copyrights, patents,
# trademarks, maskwork rights, or any other form of intellectual
# property whatsoever. Maxim Integrated Products, Inc. retains all
# ownership rights.
#******************************************************************************

 ###############################################################################
# Main configuration
# Below are the most commonly needed configuration options for this Makefile.
# You shouldn't need to touch anything else unless advanced configuration is required.

# Source files to compile for this project (add path to VPATH below)
SRCS += main.c

# Where to find source files for this project
VPATH += ./src

# Where to find header files for this project
IPATH += ./src

# Set compiler flags
PROJ_CFLAGS+=-Wall # Enable warnings

# Set compiler optimization level
MXC_OPTIMIZE=-Og

 ###############################################################################

# Set the MAKE variable.  This eliminates issues with some setups
MAKE=make

# Default target processor.  This is overridden in settings.json
ifeq "$(TARGET)" ""
TARGET=MAX32665
endif

# Set output filename to target micro
ifeq "$(PROJECT)" ""
PROJECT=$(TARGET)
endif

# Create Target name variables
TARGET_UC:=$(shell echo $(TARGET) | tr a-z A-Z)
TARGET_LC:=$(shell echo $(TARGET) | tr A-Z a-z)

# Select 'GCC' or 'IAR' compiler
COMPILER=GCC

# Default board used.  This is overridden in settings.json
ifeq "$(BOARD)" ""
BOARD=FTHR
endif

# This is the path to the CMSIS root directory
ifeq "$(MAXIM_PATH)" ""
LIBS_DIR=../../../Libraries
else
LIBS_DIR=$(subst \,/,$(subst :,,$(MAXIM_PATH))/Firmware/$(TARGET_UC)/Libraries)
endif
CMSIS_ROOT=$(LIBS_DIR)/CMSIS

ifeq ($(MAKECMDGOALS),sla)
SECURE_BOOT_TOOLS_DIR=$(LIBS_DIR)/../NDALibraries/SecureBootTools
CA_SIGN_BUILD=$(SECURE_BOOT_TOOLS_DIR)/script/sign_application.sh
BUILD_SESSION=$(SECURE_BOOT_TOOLS_DIR)/script/build_application.sh
TEST_KEY=$(SECURE_BOOT_TOOLS_DIR)/key/maximtestcrk.key
endif

ifeq ($(MAKECMDGOALS),sla)
SRCS += sla_header.c
endif

VPATH += .
ifeq ($(MAKECMDGOALS),sla)
VPATH += $(SECURE_BOOT_TOOLS_DIR)/sla_header
endif

IPATH += .

# Enable assertion checking for development
PROJ_CFLAGS+=-DMXC_ASSERT_ENABLE

# Specify the target revision to override default
# "A2" in ASCII
# TARGET_REV=0x4132

# Use this variables to specify and alternate tool path
#TOOL_DIR=/opt/gcc-arm-none-eabi-4_8-2013q4/bin

# Use these variables to add project specific tool options
#PROJ_CFLAGS+=--specs=nano.specs
#PROJ_LDFLAGS+=--specs=nano.specs

# Point this variable to a startup file to override the default file
#STARTUPFILE=start.S

# Point this variable to a linker file to override the default file
#LINKERFILE=$(CMSIS_ROOT)/Device/Maxim/$(TARGET_UC)/Source/GCC/$(TARGET_LC).ld
ifeq ($(MAKECMDGOALS),sla)
LINKERFILE=$(SECURE_BOOT_TOOLS_DIR)/linker/$(TARGET_LC)_sla.ld
endif
################################################################################
# Include external library makefiles here

# Include the BSP
BOARD_DIR=$(LIBS_DIR)/Boards/$(BOARD)
include $(BOARD_DIR)/board.mk

# Include the peripheral driver
PERIPH_DRIVER_DIR=$(LIBS_DIR)/$(TARGET_UC)PeriphDriver
include $(PERIPH_DRIVER_DIR)/periphdriver.mk

################################################################################
# Include the rules for building for this target. All other makefiles should be
# included before this one.
include $(CMSIS_ROOT)/Device/Maxim/$(TARGET_UC)/Source/$(COMPILER)/$(TARGET_LC).mk

# The rule to clean out all the build products.
distclean: clean
	$(MAKE) -C ${PERIPH_DRIVER_DIR} clean
	rm -rf scp_packets


sla: all
	arm-none-eabi-objcopy $(BUILD_DIR)/$(PROJECT).elf -R .sig -O binary $(BUILD_DIR)/$(PROJECT).bin
	$(CA_SIGN_BUILD) $(BUILD_DIR)/$(PROJECT).bin $(TEST_KEY)
	arm-none-eabi-objcopy  $(BUILD_DIR)/$(PROJECT).elf --update-section .sig=$(BUILD_DIR)/$(PROJECT).bin.sig
	arm-none-eabi-objcopy $(BUILD_DIR)/$(PROJECT).elf -O binary $(BUILD_DIR)/$(PROJECT).bin.sbin
	$(BUILD_SESSION) $(BUILD_DIR)/$(PROJECT).bin.sbin scp_packets $(TEST_KEY)
