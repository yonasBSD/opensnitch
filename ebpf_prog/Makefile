# OpenSnitch - 2023
#
# On Debian based distros we need the following 2 directories.
# Otherwise, just use the kernel headers from the kernel sources.
#
KERNEL_VER ?= $(shell ls -d /lib/modules/*/source | sort | tail -1 | cut -d/ -f4)
KERNEL_DIR ?= /lib/modules/$(KERNEL_VER)/source
KERNEL_HEADERS ?= /usr/src/linux-headers-$(KERNEL_VER)/
CC = clang
LLC ?= llc
ARCH ?= $(shell uname -m)

# as in /usr/src/linux-headers-*/arch/
# TODO: extract correctly the archs, and add more if needed.
ifeq ($(ARCH),x86_64)
	ARCH := x86
else ifeq ($(ARCH),i686)
	ARCH := x86
else ifeq ($(ARCH),armv7l)
	ARCH := arm
else ifeq ($(ARCH),armv8l)
	ARCH := arm
else ifeq ($(ARCH),aarch64)
	ARCH := arm64
else ifeq ($(ARCH),loongarch64)
	ARCH := loongarch
else ifeq ($(ARCH),riscv64)
	ARCH := riscv
else ifeq ($(ARCH),s390x)
	ARCH := s390
endif

ifeq ($(ARCH),arm)
	# on previous archs, it fails with "SMP not supported on pre-ARMv6"
	EXTRA_FLAGS = "-D__LINUX_ARM_ARCH__=7"
endif

SRC := $(wildcard *.c)
BIN := $(SRC:.c=.o)
CFLAGS = -I. \
	-I$(KERNEL_HEADERS)/arch/$(ARCH)/include/generated/ \
	-I$(KERNEL_HEADERS)/include \
	-include $(KERNEL_DIR)/include/linux/kconfig.h \
	-I$(KERNEL_DIR)/include \
	-I$(KERNEL_DIR)/include/uapi \
	-I$(KERNEL_DIR)/include/generated/uapi \
	-I$(KERNEL_DIR)/arch/$(ARCH)/include \
	-I$(KERNEL_DIR)/arch/$(ARCH)/include/generated \
	-I$(KERNEL_DIR)/arch/$(ARCH)/include/uapi \
	-I$(KERNEL_DIR)/arch/$(ARCH)/include/generated/uapi \
	-I$(KERNEL_DIR)/tools/testing/selftests/bpf/ \
	-D__KERNEL__ -D__BPF_TRACING__ -Wno-unused-value -Wno-pointer-sign \
	-D__TARGET_ARCH_$(ARCH) -Wno-compare-distinct-pointer-types \
	$(EXTRA_FLAGS) \
	-Wunused \
	-Wno-unused-value \
	-Wno-gnu-variable-sized-type-not-at-end \
	-Wno-address-of-packed-member \
	-Wno-tautological-compare \
	-Wno-unknown-warning-option  \
	-fno-stack-protector \
	-g -O2 -emit-llvm

all: $(BIN)

%.bc: %.c
	$(CC) $(CFLAGS) -c $<

%.o: %.bc
	$(LLC) -march=bpf -mcpu=generic -filetype=obj -o $@ $<

clean:
	rm -f $(BIN)

.SUFFIXES:
