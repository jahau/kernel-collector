CLANG = clang
LLC = llc
KERNELSOURCE = /usr/src/linux
ARCH = x86

EXTRA_CFLAGS += -fno-stack-protector

LINUXINCLUDE += -I$(KERNELSOURCE)/arch/$(ARCH)/include/generated
LINUXINCLUDE += -I$(KERNELSOURCE)/arch/$(ARCH)/include/generated/uapi
LINUXINCLUDE += -I$(KERNELSOURCE)/include

LINUXINCLUDE += -I$(KERNELSOURCE)/include/generated/uapi
LINUXINCLUDE += -I$(KERNELSOURCE)/arch/$(ARCH)/include
LINUXINCLUDE += -I$(KERNELSOURCE)/arch/$(ARCH)/include/uapi
LINUXINCLUDE += -I$(KERNELSOURCE)/arch/$(ARCH)/include/generated
LINUXINCLUDE += -I$(KERNELSOURCE)/arch/$(ARCH)/include/generated/uapi
LINUXINCLUDE += -I$(KERNELSOURCE)/include
LINUXINCLUDE += -I$(KERNELSOURCE)/include/uapi
LINUXINCLUDE += -include $(KERNELSOURCE)/include/linux/kconfig.h
LINUXINCLUDE += -I../includes

CLANG_VERSION := $(shell clang --version | grep -o -E '[0-9]+\.[0-9]+\.[0-9]' | uniq)
LLVM_INCLUDES = -I/usr/lib/clang/$(CLANG_VERSION)/include

KERNEL_VERSION="$(shell basename $(realpath /usr/src/linux) | cut -f 2 -d '-')"

VER_MAJOR=$(shell echo $(KERNEL_VERSION) | cut -d. -f1)
VER_MINOR=$(shell echo $(KERNEL_VERSION) | cut -d. -f2)
VER_PATCH=$(shell echo $(KERNEL_VERSION) | cut -d. -f3)

CURRENT_KERNEL=$(shell echo $(VER_MAJOR)\*65536 + $(VER_MINOR)\*256 + $(VER_PATCH) |bc)

latency_process_kern.o: latency_process_kern.c
process_kern.o: process_kern.c

all: process_kern.o latency_process_kern.o

%.o: %.c
	if [ "$(CURRENT_KERNEL)" -ge 328448 ]; then sed -i -e 's/\(#define CONFIG_CC_HAS_ASM_INLINE 1\)/\/\/\1/' /usr/src/linux/include/generated/autoconf.h; fi
	$(CLANG) $(EXTRA_CFLAGS) -S -nostdinc $(LINUXINCLUDE) $(LLVM_INCLUDES) \
		-D__KERNEL__ -D__ASM_SYSREG_H -Wno-unused-value -Wno-pointer-sign \
	    	-Wno-compare-distinct-pointer-types \
	    	-Wno-gnu-variable-sized-type-not-at-end \
	    	-Wno-tautological-compare \
		-DNETDATASEL=0 \
		-O2 -emit-llvm -c $<
	$(LLC) -march=bpf -filetype=obj -o r$@ $(<:.c=.ll)
	$(CLANG) $(EXTRA_CFLAGS) -S -nostdinc $(LINUXINCLUDE) $(LLVM_INCLUDES) \
		-D__KERNEL__ -D__ASM_SYSREG_H -Wno-unused-value -Wno-pointer-sign \
	    	-Wno-compare-distinct-pointer-types \
	    	-Wno-gnu-variable-sized-type-not-at-end \
	    	-Wno-tautological-compare \
		-DNETDATASEL=1 \
		-O2 -emit-llvm -c $<
	$(LLC) -march=bpf -filetype=obj -o d$@ $(<:.c=.ll)
	$(CLANG) $(EXTRA_CFLAGS) -S -nostdinc $(LINUXINCLUDE) $(LLVM_INCLUDES) \
		-D__KERNEL__ -D__ASM_SYSREG_H -Wno-unused-value -Wno-pointer-sign \
	    	-Wno-compare-distinct-pointer-types \
	    	-Wno-gnu-variable-sized-type-not-at-end \
	    	-Wno-tautological-compare \
		-DNETDATASEL=2 \
		-O2 -emit-llvm -c $<
	$(LLC) -march=bpf -filetype=obj -o p$@ $(<:.c=.ll)

clean:
	rm *.o *.ll
