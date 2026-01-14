# 定义u-boot版本
VERSION = 2016
PATCHLEVEL = 03
SUBLEVEL =
EXTRAVERSION =
NAME =

# -r 禁用 GNU Make 自带的内置规则
# -R 禁用 GNU Make 自带的内置变量(禁用的是为编译/构建场景下的内置变量，并非所有变量，所以CURDIR是生效的)
# --include=<dir> 告诉make，当执行include xxx.mk这类指令时，优先到我指定的CURDIR去查找
# CURDIR Make的内置变量，代表当前 Makefile 所在的绝对路径
MAKEFLAGS += -rR --include-dir=$(CURDIR)

unexport LC_ALL
LC_COLLATE=C
LC_NUMERIC=C
export LC_COLLATE LC_NUMERIC

unexport GREP_OPTIONS

# $(origin V) Make的内置函数，返回V的来源(command line/file/undefined)
ifeq ("$(origin V)", "command line")
  KBUILD_VERBOSE = $(V)
endif
# KBUILD_VERBOSE默认是0
ifndef KBUILD_VERBOSE
  KBUILD_VERBOSE = 0
endif

# quiet_ 只输出极简的进度提示
# @ make的特殊符号，会抑制命令的回显
# 默认是静默，并且抑制命令的回显
ifeq ($(KBUILD_VERBOSE),1)
  quiet =
  Q =
else
  quiet=quiet_
  Q = @
endif

# $(filter %s, TEXT)
# 核心目的是为了兼容旧版本的make，最终都是为什么命令行键入make -s时quiet=silent_
ifneq ($(filter 4.%,$(MAKE_VERSION)),)
ifneq ($(filter %s ,$(firstword x$(MAKEFLAGS))),)
  quiet=silent_
endif
else
ifneq ($(filter s% -s%,$(MAKEFLAGS)),)
  quiet=silent_
endif
endif

export quiet Q KBUILD_VERBOSE

ifeq ($(KBUILD_SRC),)

ifeq ("$(origin O)", "command line")
  KBUILD_OUTPUT := $(O)
endif

PHONY := _all
_all:

$(CURDIR)/Makefile Makefile: ;

ifneq ($(KBUILD_OUTPUT),)
saved-output := $(KBUILD_OUTPUT)
KBUILD_OUTPUT := $(shell mkdir -p $(KBUILD_OUTPUT) && cd $(KBUILD_OUTPUT) \
								&& /bin/pwd)
$(if $(KBUILD_OUTPUT),, \
     $(error failed to create output directory "$(saved-output)"))

PHONY += $(MAKECMDGOALS) sub-make

$(filter-out _all sub-make $(CURDIR)/Makefile, $(MAKECMDGOALS)) _all: sub-make
	@:

sub-make: FORCE
	$(Q)$(MAKE) -C $(KBUILD_OUTPUT) KBUILD_SRC=$(CURDIR) \
	-f $(CURDIR)/Makefile $(filter-out _all sub-make,$(MAKECMDGOALS))

skip-makefile := 1
endif
endif

ifeq ($(skip-makefile),)

MAKEFLAGS += --no-print-directory

ifeq ("$(origin C)", "command line")
  KBUILD_CHECKSRC = $(C)
endif
ifndef KBUILD_CHECKSRC
  KBUILD_CHECKSRC = 0
endif

ifdef SUBDIRS
  KBUILD_EXTMOD ?= $(SUBDIRS)
endif

ifeq ("$(origin M)", "command line")
  KBUILD_EXTMOD := $(M)
endif

PHONY += all
ifeq ($(KBUILD_EXTMOD),)
_all: all
else
_all: modules
endif

ifeq ($(KBUILD_SRC),)
        srctree := .
else
        ifeq ($(KBUILD_SRC)/,$(dir $(CURDIR)))
                srctree := ..
        else
                srctree := $(KBUILD_SRC)
        endif
endif
objtree		:= .
src		:= $(srctree)
obj		:= $(objtree)

VPATH		:= $(srctree)$(if $(KBUILD_EXTMOD),:$(KBUILD_EXTMOD))

export srctree objtree VPATH

unexport CDPATH

HOSTARCH := $(shell uname -m | \
	sed -e s/i.86/x86/ \
	    -e s/sun4u/sparc64/ \
	    -e s/arm.*/arm/ \
	    -e s/sa110/arm/ \
	    -e s/ppc64/powerpc/ \
	    -e s/ppc/powerpc/ \
	    -e s/macppc/powerpc/\
	    -e s/sh.*/sh/)

HOSTOS := $(shell uname -s | tr '[:upper:]' '[:lower:]' | \
	    sed -e 's/\(cygwin\).*/cygwin/')

export	HOSTARCH HOSTOS

ifeq ($(HOSTARCH),$(ARCH))
CROSS_COMPILE ?=
endif

KCONFIG_CONFIG	?= .config
export KCONFIG_CONFIG

CONFIG_SHELL := $(shell if [ -x "$$BASH" ]; then echo $$BASH; \
	  else if [ -x /bin/bash ]; then echo /bin/bash; \
	  else echo sh; fi ; fi)

HOSTCC       = cc
HOSTCXX      = c++
HOSTCFLAGS   = -Wall -Wstrict-prototypes -O2 -fomit-frame-pointer
HOSTCXXFLAGS = -O2

ifeq ($(HOSTOS),cygwin)
HOSTCFLAGS	+= -ansi
endif

ifeq ($(HOSTOS),darwin)
DARWIN_MAJOR_VERSION	= $(shell sw_vers -productVersion | cut -f 1 -d '.')
DARWIN_MINOR_VERSION	= $(shell sw_vers -productVersion | cut -f 2 -d '.')

os_x_before	= $(shell if [ $(DARWIN_MAJOR_VERSION) -le $(1) -a \
	$(DARWIN_MINOR_VERSION) -le $(2) ] ; then echo "$(3)"; else echo "$(4)"; fi ;)

HOSTCC       = $(call os_x_before, 10, 5, "cc", "gcc")
HOSTCFLAGS  += $(call os_x_before, 10, 4, "-traditional-cpp")
HOSTLDFLAGS += $(call os_x_before, 10, 5, "-multiply_defined suppress")

HOSTLDFLAGS += $(call os_x_before, 10, 7, "", "-Xlinker -no_pie")
endif

KBUILD_MODULES :=
KBUILD_BUILTIN := 1

ifeq ($(MAKECMDGOALS),modules)
  KBUILD_BUILTIN := $(if $(CONFIG_MODVERSIONS),1)
endif

export KBUILD_MODULES KBUILD_BUILTIN
export KBUILD_CHECKSRC KBUILD_SRC KBUILD_EXTMOD

scripts/Kbuild.include: ;
include scripts/Kbuild.include

AS		= $(CROSS_COMPILE)as
ifneq ($(shell $(CROSS_COMPILE)ld.bfd -v 2> /dev/null),)
LD		= $(CROSS_COMPILE)ld.bfd
else
LD		= $(CROSS_COMPILE)ld
endif
CC		= $(CROSS_COMPILE)gcc
CPP		= $(CC) -E
AR		= $(CROSS_COMPILE)ar
NM		= $(CROSS_COMPILE)nm
LDR		= $(CROSS_COMPILE)ldr
STRIP		= $(CROSS_COMPILE)strip
OBJCOPY		= $(CROSS_COMPILE)objcopy
OBJDUMP		= $(CROSS_COMPILE)objdump
AWK		= awk
PERL		= perl
PYTHON		= python
DTC		= dtc
CHECK		= sparse

CHECKFLAGS     := -D__linux__ -Dlinux -D__STDC__ -Dunix -D__unix__ \
		  -Wbitwise -Wno-return-void -D__CHECK_ENDIAN__ $(CF)

KBUILD_CPPFLAGS := -D__KERNEL__ -D__UBOOT__

KBUILD_CFLAGS   := -Wall -Wstrict-prototypes \
		   -Wno-format-security \
		   -fno-builtin -ffreestanding
KBUILD_AFLAGS   := -D__ASSEMBLY__

UBOOTRELEASE = $(shell cat include/config/uboot.release 2> /dev/null)
UBOOTVERSION = $(VERSION)$(if $(PATCHLEVEL),.$(PATCHLEVEL)$(if $(SUBLEVEL),.$(SUBLEVEL)))$(EXTRAVERSION)

export VERSION PATCHLEVEL SUBLEVEL UBOOTRELEASE UBOOTVERSION
export ARCH CPU BOARD VENDOR SOC CPUDIR BOARDDIR
export CONFIG_SHELL HOSTCC HOSTCFLAGS HOSTLDFLAGS CROSS_COMPILE AS LD CC
export CPP AR NM LDR STRIP OBJCOPY OBJDUMP
export MAKE AWK PERL PYTHON
export HOSTCXX HOSTCXXFLAGS DTC CHECK CHECKFLAGS

export KBUILD_CPPFLAGS NOSTDINC_FLAGS UBOOTINCLUDE OBJCOPYFLAGS LDFLAGS
export KBUILD_CFLAGS KBUILD_AFLAGS

export MODVERDIR := $(if $(KBUILD_EXTMOD),$(firstword $(KBUILD_EXTMOD))/).tmp_versions

export RCS_FIND_IGNORE := \( -name SCCS -o -name BitKeeper -o -name .svn -o    \
			  -name CVS -o -name .pc -o -name .hg -o -name .git \) \
			  -prune -o
export RCS_TAR_IGNORE := --exclude SCCS --exclude BitKeeper --exclude .svn \
			 --exclude CVS --exclude .pc --exclude .hg --exclude .git

PHONY += scripts_basic
scripts_basic:
	$(Q)$(MAKE) $(build)=scripts/basic
	$(Q)rm -f .tmp_quiet_recordmcount

scripts/basic/%: scripts_basic ;

PHONY += outputmakefile
outputmakefile:
ifneq ($(KBUILD_SRC),)
	$(Q)ln -fsn $(srctree) source
	$(Q)$(CONFIG_SHELL) $(srctree)/scripts/mkmakefile \
	    $(srctree) $(objtree) $(VERSION) $(PATCHLEVEL)
endif

version_h := include/generated/version_autogenerated.h
timestamp_h := include/generated/timestamp_autogenerated.h

no-dot-config-targets := clean clobber mrproper distclean \
			 help %docs check% coccicheck \
			 ubootversion backup

config-targets := 0
mixed-targets  := 0
dot-config     := 1

ifneq ($(filter $(no-dot-config-targets), $(MAKECMDGOALS)),)
	ifeq ($(filter-out $(no-dot-config-targets), $(MAKECMDGOALS)),)
		dot-config := 0
	endif
endif

ifeq ($(KBUILD_EXTMOD),)
        ifneq ($(filter config %config,$(MAKECMDGOALS)),)
                config-targets := 1
                ifneq ($(words $(MAKECMDGOALS)),1)
                        mixed-targets := 1
                endif
        endif
endif

ifeq ($(mixed-targets),1)

PHONY += $(MAKECMDGOALS) __build_one_by_one

$(filter-out __build_one_by_one, $(MAKECMDGOALS)): __build_one_by_one
	@:

__build_one_by_one:
	$(Q)set -e; \
	for i in $(MAKECMDGOALS); do \
		$(MAKE) -f $(srctree)/Makefile $$i; \
	done

else
ifeq ($(config-targets),1)

KBUILD_DEFCONFIG := sandbox_defconfig
export KBUILD_DEFCONFIG KBUILD_KCONFIG

config: scripts_basic outputmakefile FORCE
	$(Q)$(MAKE) $(build)=scripts/kconfig $@

%config: scripts_basic outputmakefile FORCE
	$(Q)$(MAKE) $(build)=scripts/kconfig $@

else

ifeq ($(dot-config),1)
-include include/config/auto.conf

-include include/config/auto.conf.cmd

$(KCONFIG_CONFIG) include/config/auto.conf.cmd: ;

include/config/%.conf: $(KCONFIG_CONFIG) include/config/auto.conf.cmd
	$(Q)$(MAKE) -f $(srctree)/Makefile silentoldconfig
	$(Q)$(MAKE) -f $(srctree)/scripts/Makefile.autoconf || \
		{ rm -f include/config/auto.conf; false; }
	$(Q)touch include/config/auto.conf

-include include/autoconf.mk
-include include/autoconf.mk.dep

ifneq ($(wildcard $(KCONFIG_CONFIG)),)
ifneq ($(wildcard include/config/auto.conf),)
autoconf_is_old := $(shell find . -path ./$(KCONFIG_CONFIG) -newer \
						include/config/auto.conf)
ifeq ($(autoconf_is_old),)
include config.mk
include arch/$(ARCH)/Makefile
endif
endif
endif

ifndef LDSCRIPT
	ifdef CONFIG_SYS_LDSCRIPT
		LDSCRIPT := $(srctree)/$(CONFIG_SYS_LDSCRIPT:"%"=%)
	endif
endif

ifndef LDSCRIPT
	ifeq ($(wildcard $(LDSCRIPT)),)
		LDSCRIPT := $(srctree)/board/$(BOARDDIR)/u-boot.lds
	endif
	ifeq ($(wildcard $(LDSCRIPT)),)
		LDSCRIPT := $(srctree)/$(CPUDIR)/u-boot.lds
	endif
	ifeq ($(wildcard $(LDSCRIPT)),)
		LDSCRIPT := $(srctree)/arch/$(ARCH)/cpu/u-boot.lds
	endif
endif

else
include/config/auto.conf: ;
endif

ifdef CONFIG_CC_OPTIMIZE_FOR_SIZE
KBUILD_CFLAGS	+= -Os
else
KBUILD_CFLAGS	+= -O2
endif

KBUILD_CFLAGS += $(call cc-option,-fno-stack-protector)
KBUILD_CFLAGS += $(call cc-option,-fno-delete-null-pointer-checks)

KBUILD_CFLAGS	+= -g
KBUILD_AFLAGS	+= -g

ifeq ($(shell $(CONFIG_SHELL) $(srctree)/scripts/gcc-stack-usage.sh $(CC)),y)
	KBUILD_CFLAGS += -fstack-usage
endif

KBUILD_CFLAGS += $(call cc-option,-Wno-format-nonliteral)

ifeq ($(ARCH),m68k)
ifeq ($(findstring 3.4,$(shell $(CC) --version)),3.4)
KBUILD_AFLAGS += -Wa,-gstabs,-S
endif
endif

KBUILD_CFLAGS   += $(call cc-option,-Werror=date-time)

include scripts/Makefile.extrawarn

KBUILD_CPPFLAGS += $(KCPPFLAGS)
KBUILD_AFLAGS += $(KAFLAGS)
KBUILD_CFLAGS += $(KCFLAGS)

UBOOTINCLUDE    := \
		-Iinclude \
		$(if $(KBUILD_SRC), -I$(srctree)/include) \
		$(if $(CONFIG_SYS_THUMB_BUILD), $(if $(CONFIG_HAS_THUMB2),, \
			-I$(srctree)/arch/$(ARCH)/thumb1/include),) \
		-I$(srctree)/arch/$(ARCH)/include \
		-include $(srctree)/include/linux/kconfig.h

NOSTDINC_FLAGS += -nostdinc -isystem $(shell $(CC) -print-file-name=include)
CHECKFLAGS     += $(NOSTDINC_FLAGS)

cpp_flags := $(KBUILD_CPPFLAGS) $(PLATFORM_CPPFLAGS) $(UBOOTINCLUDE) \
							$(NOSTDINC_FLAGS)
c_flags := $(KBUILD_CFLAGS) $(cpp_flags)

HAVE_VENDOR_COMMON_LIB = $(if $(wildcard $(srctree)/board/$(VENDOR)/common/Makefile),y,n)

libs-y += lib/
libs-$(HAVE_VENDOR_COMMON_LIB) += board/$(VENDOR)/common/
libs-$(CONFIG_OF_EMBED) += dts/
libs-y += fs/
libs-y += net/
libs-y += disk/
libs-y += drivers/
libs-y += drivers/dma/
libs-y += drivers/gpio/
libs-y += drivers/i2c/
libs-y += drivers/mmc/
libs-y += drivers/mtd/
libs-$(CONFIG_CMD_NAND) += drivers/mtd/nand/
libs-y += drivers/mtd/onenand/
libs-$(CONFIG_CMD_UBI) += drivers/mtd/ubi/
libs-y += drivers/mtd/spi/
libs-y += drivers/net/
libs-y += drivers/net/phy/
libs-y += drivers/pci/
libs-y += drivers/power/ \
	drivers/power/fuel_gauge/ \
	drivers/power/mfd/ \
	drivers/power/pmic/ \
	drivers/power/battery/ \
	drivers/power/regulator/
libs-y += drivers/spi/
libs-$(CONFIG_FMAN_ENET) += drivers/net/fm/
libs-$(CONFIG_SYS_FSL_DDR) += drivers/ddr/fsl/
libs-$(CONFIG_ALTERA_SDRAM) += drivers/ddr/altera/
libs-y += drivers/serial/
libs-y += drivers/usb/dwc3/
libs-y += drivers/usb/emul/
libs-y += drivers/usb/eth/
libs-y += drivers/usb/gadget/
libs-y += drivers/usb/gadget/udc/
libs-y += drivers/usb/host/
libs-y += drivers/usb/musb/
libs-y += drivers/usb/musb-new/
libs-y += drivers/usb/phy/
libs-y += drivers/usb/ulpi/
libs-y += cmd/
libs-y += common/
libs-$(CONFIG_API) += api/
libs-$(CONFIG_HAS_POST) += post/
libs-y += test/
libs-y += test/dm/
libs-$(CONFIG_UT_ENV) += test/env/

libs-y += $(if $(BOARDDIR),board/$(BOARDDIR)/)

libs-y := $(sort $(libs-y))

u-boot-dirs	:= $(patsubst %/,%,$(filter %/, $(libs-y))) tools examples

u-boot-alldirs	:= $(sort $(u-boot-dirs) $(patsubst %/,%,$(filter %/, $(libs-))))

libs-y		:= $(patsubst %/, %/built-in.o, $(libs-y))

u-boot-init := $(head-y)
u-boot-main := $(libs-y)

ifeq ($(CONFIG_USE_PRIVATE_LIBGCC),y)
PLATFORM_LIBGCC = arch/$(ARCH)/lib/lib.a
else
PLATFORM_LIBGCC := -L $(shell dirname `$(CC) $(c_flags) -print-libgcc-file-name`) -lgcc
endif
PLATFORM_LIBS += $(PLATFORM_LIBGCC)
export PLATFORM_LIBS
export PLATFORM_LIBGCC

LDPPFLAGS += \
	-include $(srctree)/include/u-boot/u-boot.lds.h \
	-DCPUDIR=$(CPUDIR) \
	$(shell $(LD) --version | \
	  sed -ne 's/GNU ld version \([0-9][0-9]*\)\.\([0-9][0-9]*\).*/-DLD_MAJOR=\1 -DLD_MINOR=\2/p')

ifneq ($(CONFIG_BOARD_SIZE_LIMIT),)
BOARD_SIZE_CHECK = \
	@actual=`wc -c $@ | awk '{print $$1}'`; \
	limit=`printf "%d" $(CONFIG_BOARD_SIZE_LIMIT)`; \
	if test $$actual -gt $$limit; then \
		echo "$@ exceeds file size limit:" >&2 ; \
		echo "  limit:  $$limit bytes" >&2 ; \
		echo "  actual: $$actual bytes" >&2 ; \
		echo "  excess: $$((actual - limit)) bytes" >&2; \
		exit 1; \
	fi
else
BOARD_SIZE_CHECK =
endif

ifneq ($(CONFIG_STATIC_RELA),)
DO_STATIC_RELA = \
	start=$$($(NM) $(1) | grep __rel_dyn_start | cut -f 1 -d ' '); \
	end=$$($(NM) $(1) | grep __rel_dyn_end | cut -f 1 -d ' '); \
	tools/relocate-rela $(2) $(3) $$start $$end
else
DO_STATIC_RELA =
endif

ALL-y += u-boot.srec u-boot.bin u-boot.sym System.map u-boot.cfg binary_size_check

ALL-$(CONFIG_ONENAND_U_BOOT) += u-boot-onenand.bin
ifeq ($(CONFIG_SPL_FSL_PBL),y)
ALL-$(CONFIG_RAMBOOT_PBL) += u-boot-with-spl-pbl.bin
else
ifneq ($(CONFIG_SECURE_BOOT), y)
ALL-$(CONFIG_RAMBOOT_PBL) += u-boot.pbl
endif
endif
ALL-$(CONFIG_SPL) += spl/u-boot-spl.bin
ALL-$(CONFIG_SPL_FRAMEWORK) += u-boot.img
ALL-$(CONFIG_TPL) += tpl/u-boot-tpl.bin
ALL-$(CONFIG_OF_SEPARATE) += u-boot.dtb
ifeq ($(CONFIG_SPL_FRAMEWORK),y)
ALL-$(CONFIG_OF_SEPARATE) += u-boot-dtb.img
endif
ALL-$(CONFIG_OF_HOSTFILE) += u-boot.dtb
ifneq ($(CONFIG_SPL_TARGET),)
ALL-$(CONFIG_SPL) += $(CONFIG_SPL_TARGET:"%"=%)
endif
ALL-$(CONFIG_REMAKE_ELF) += u-boot.elf
ALL-$(CONFIG_EFI_APP) += u-boot-app.efi
ALL-$(CONFIG_EFI_STUB) += u-boot-payload.efi

ifneq ($(BUILD_ROM),)
ALL-$(CONFIG_X86_RESET_VECTOR) += u-boot.rom
endif

ifeq ($(CONFIG_TEGRA)$(CONFIG_SPL),yy)
ALL-y += u-boot-tegra.bin u-boot-nodtb-tegra.bin
ALL-$(CONFIG_OF_SEPARATE) += u-boot-dtb-tegra.bin
endif

ifneq ($(CONFIG_BUILD_TARGET),)
ALL-y += $(CONFIG_BUILD_TARGET:"%"=%)
endif

LDFLAGS_u-boot += $(LDFLAGS_FINAL)
ifneq ($(CONFIG_SYS_TEXT_BASE),)
LDFLAGS_u-boot += -Ttext $(CONFIG_SYS_TEXT_BASE)
endif

quiet_cmd_objcopy = OBJCOPY $@
cmd_objcopy = $(OBJCOPY) --gap-fill=0xff $(OBJCOPYFLAGS) \
	$(OBJCOPYFLAGS_$(@F)) $< $@

quiet_cmd_zobjcopy = OBJCOPY $@
cmd_zobjcopy = $(OBJCOPY) $(OBJCOPYFLAGS) $(OBJCOPYFLAGS_$(@F)) $< $@

quiet_cmd_efipayload = OBJCOPY $@
cmd_efipayload = $(OBJCOPY) -I binary -O $(EFIPAYLOAD_BFDTARGET) -B $(EFIPAYLOAD_BFDARCH) $< $@

quiet_cmd_mkimage = MKIMAGE $@
cmd_mkimage = $(objtree)/tools/mkimage $(MKIMAGEFLAGS_$(@F)) -d $< $@ \
	$(if $(KBUILD_VERBOSE:1=), >/dev/null)

quiet_cmd_cat = CAT     $@
cmd_cat = cat $(filter-out $(PHONY), $^) > $@

append = cat $(filter-out $< $(PHONY), $^) >> $@

quiet_cmd_pad_cat = CAT     $@
cmd_pad_cat = $(cmd_objcopy) && $(append) || rm -f $@

all:		$(ALL-y)
ifneq ($(CONFIG_SYS_GENERIC_BOARD),y)
	@echo "===================== WARNING ======================"
	@echo "Please convert this board to generic board."
	@echo "Otherwise it will be removed by the end of 2014."
	@echo "See doc/README.generic-board for further information"
	@echo "===================================================="
endif
ifeq ($(CONFIG_DM_I2C_COMPAT),y)
	@echo "===================== WARNING ======================"
	@echo "This board uses CONFIG_DM_I2C_COMPAT. Please remove"
	@echo "(possibly in a subsequent patch in your series)"
	@echo "before sending patches to the mailing list."
	@echo "===================================================="
endif

PHONY += dtbs
dtbs dts/dt.dtb: checkdtc u-boot
	$(Q)$(MAKE) $(build)=dts dtbs

quiet_cmd_copy = COPY    $@
      cmd_copy = cp $< $@

ifeq ($(CONFIG_OF_SEPARATE),y)
u-boot-dtb.bin: u-boot-nodtb.bin dts/dt.dtb FORCE
	$(call if_changed,cat)

u-boot.bin: u-boot-dtb.bin FORCE
	$(call if_changed,copy)
else
u-boot.bin: u-boot-nodtb.bin FORCE
	$(call if_changed,copy)
endif

%.imx: %.bin
	$(Q)$(MAKE) $(build)=arch/arm/imx-common $@

u-boot.dtb: dts/dt.dtb
	$(call cmd,copy)

OBJCOPYFLAGS_u-boot.hex := -O ihex

OBJCOPYFLAGS_u-boot.srec := -O srec

u-boot.hex u-boot.srec: u-boot FORCE
	$(call if_changed,objcopy)

OBJCOPYFLAGS_u-boot-nodtb.bin := -O binary \
		$(if $(CONFIG_X86_RESET_VECTOR),-R .start16 -R .resetvec)

binary_size_check: u-boot-nodtb.bin FORCE
	@file_size=$(shell wc -c u-boot-nodtb.bin | awk '{print $$1}') ; \
	map_size=$(shell cat u-boot.map | \
		awk '/_image_copy_start/ {start = $$1} /_image_binary_end/ {end = $$1} END {if (start != "" && end != "") print "ibase=16; " toupper(end) " - " toupper(start)}' \
		| sed 's/0X//g' \
		| bc); \
	if [ "" != "$$map_size" ]; then \
		if test $$map_size -ne $$file_size; then \
			echo "u-boot.map shows a binary size of $$map_size" >&2 ; \
			echo "  but u-boot-nodtb.bin shows $$file_size" >&2 ; \
			exit 1; \
		fi \
	fi

u-boot-nodtb.bin: u-boot FORCE
	$(call if_changed,objcopy)
	$(call DO_STATIC_RELA,$<,$@,$(CONFIG_SYS_TEXT_BASE))
	$(BOARD_SIZE_CHECK)

u-boot.ldr:	u-boot
		$(CREATE_LDR_ENV)
		$(LDR) -T $(CONFIG_CPU) -c $@ $< $(LDR_FLAGS)
		$(BOARD_SIZE_CHECK)

OBJCOPYFLAGS_u-boot.ldr.hex := -I binary -O ihex

OBJCOPYFLAGS_u-boot.ldr.srec := -I binary -O srec

u-boot.ldr.hex u-boot.ldr.srec: u-boot.ldr FORCE
	$(call if_changed,objcopy)

ifndef CONFIG_SYS_UBOOT_START
CONFIG_SYS_UBOOT_START := 0
endif

quiet_cmd_cpp_cfg = CFG     $@
cmd_cpp_cfg = $(CPP) -Wp,-MD,$(depfile) $(cpp_flags) $(LDPPFLAGS) -ansi \
	-DDO_DEPS_ONLY -D__ASSEMBLY__ -x assembler-with-cpp -P -dM -E -o $@ $<

MKIMAGEFLAGS_u-boot.img = -A $(ARCH) -T firmware -C none -O u-boot \
	-a $(CONFIG_SYS_TEXT_BASE) -e $(CONFIG_SYS_UBOOT_START) \
	-n "U-Boot $(UBOOTRELEASE) for $(BOARD) board"

MKIMAGEFLAGS_u-boot-dtb.img = $(MKIMAGEFLAGS_u-boot.img)

MKIMAGEFLAGS_u-boot.kwb = -n $(srctree)/$(CONFIG_SYS_KWD_CONFIG:"%"=%) \
	-T kwbimage -a $(CONFIG_SYS_TEXT_BASE) -e $(CONFIG_SYS_TEXT_BASE)

MKIMAGEFLAGS_u-boot-spl.kwb = -n $(srctree)/$(CONFIG_SYS_KWD_CONFIG:"%"=%) \
	-T kwbimage -a $(CONFIG_SYS_TEXT_BASE) -e $(CONFIG_SYS_TEXT_BASE)

MKIMAGEFLAGS_u-boot.pbl = -n $(srctree)/$(CONFIG_SYS_FSL_PBL_RCW:"%"=%) \
		-R $(srctree)/$(CONFIG_SYS_FSL_PBL_PBI:"%"=%) -T pblimage

u-boot-dtb.img u-boot.img u-boot.kwb u-boot.pbl: u-boot.bin FORCE
	$(call if_changed,mkimage)

u-boot-spl.kwb: u-boot.img spl/u-boot-spl.bin FORCE
	$(call if_changed,mkimage)

u-boot.sha1:	u-boot.bin
		tools/ubsha1 u-boot.bin

u-boot.dis:	u-boot
		$(OBJDUMP) -d $< > $@

u-boot.cfg:	include/config.h FORCE
	$(call if_changed,cpp_cfg)

ifdef CONFIG_TPL
SPL_PAYLOAD := tpl/u-boot-with-tpl.bin
else
SPL_PAYLOAD := u-boot.bin
endif

OBJCOPYFLAGS_u-boot-with-spl.bin = -I binary -O binary \
				   --pad-to=$(CONFIG_SPL_PAD_TO)
u-boot-with-spl.bin: spl/u-boot-spl.bin $(SPL_PAYLOAD) FORCE
	$(call if_changed,pad_cat)

MKIMAGEFLAGS_lpc32xx-spl.img = -T lpc32xximage -a $(CONFIG_SPL_TEXT_BASE)

lpc32xx-spl.img: spl/u-boot-spl.bin FORCE
	$(call if_changed,mkimage)

OBJCOPYFLAGS_lpc32xx-boot-0.bin = -I binary -O binary --pad-to=$(CONFIG_SPL_PAD_TO)

lpc32xx-boot-0.bin: lpc32xx-spl.img FORCE
	$(call if_changed,objcopy)

OBJCOPYFLAGS_lpc32xx-boot-1.bin = -I binary -O binary --pad-to=$(CONFIG_SPL_PAD_TO)

lpc32xx-boot-1.bin: lpc32xx-spl.img FORCE
	$(call if_changed,objcopy)

lpc32xx-full.bin: lpc32xx-boot-0.bin lpc32xx-boot-1.bin u-boot.img FORCE
	$(call if_changed,cat)

CLEAN_FILES += lpc32xx-*

OBJCOPYFLAGS_u-boot-with-tpl.bin = -I binary -O binary \
				   --pad-to=$(CONFIG_TPL_PAD_TO)
tpl/u-boot-with-tpl.bin: tpl/u-boot-tpl.bin u-boot.bin FORCE
	$(call if_changed,pad_cat)

SPL: spl/u-boot-spl.bin FORCE
	$(Q)$(MAKE) $(build)=arch/arm/imx-common $@

u-boot-with-spl.imx u-boot-with-nand-spl.imx: SPL u-boot.bin FORCE
	$(Q)$(MAKE) $(build)=arch/arm/imx-common $@

MKIMAGEFLAGS_u-boot.ubl = -n $(UBL_CONFIG) -T ublimage -e $(CONFIG_SYS_TEXT_BASE)

u-boot.ubl: u-boot-with-spl.bin FORCE
	$(call if_changed,mkimage)

MKIMAGEFLAGS_u-boot-spl.ais = -s -n $(if $(CONFIG_AIS_CONFIG_FILE), \
	$(srctree)/$(CONFIG_AIS_CONFIG_FILE:"%"=%),"/dev/null") \
	-T aisimage -e $(CONFIG_SPL_TEXT_BASE)
spl/u-boot-spl.ais: spl/u-boot-spl.bin FORCE
	$(call if_changed,mkimage)

OBJCOPYFLAGS_u-boot.ais = -I binary -O binary --pad-to=$(CONFIG_SPL_PAD_TO)
u-boot.ais: spl/u-boot-spl.ais u-boot.img FORCE
	$(call if_changed,pad_cat)

u-boot-signed.sb: u-boot.bin spl/u-boot-spl.bin
	$(Q)$(MAKE) $(build)=arch/arm/cpu/arm926ejs/mxs u-boot-signed.sb
u-boot.sb: u-boot.bin spl/u-boot-spl.bin
	$(Q)$(MAKE) $(build)=arch/arm/cpu/arm926ejs/mxs u-boot.sb

MKIMAGEFLAGS_u-boot-spl.img = -A $(ARCH) -T firmware -C none \
	-a $(CONFIG_SPL_TEXT_BASE) -e $(CONFIG_SPL_TEXT_BASE) -n XLOADER
spl/u-boot-spl.img: spl/u-boot-spl.bin FORCE
	$(call if_changed,mkimage)

OBJCOPYFLAGS_u-boot.spr = -I binary -O binary --pad-to=$(CONFIG_SPL_PAD_TO) \
			  --gap-fill=0xff
u-boot.spr: spl/u-boot-spl.img u-boot.img FORCE
	$(call if_changed,pad_cat)

ifneq ($(CONFIG_ARCH_SOCFPGA),)
quiet_cmd_socboot = SOCBOOT $@
cmd_socboot = cat	spl/u-boot-spl.sfp spl/u-boot-spl.sfp	\
			spl/u-boot-spl.sfp spl/u-boot-spl.sfp	\
			u-boot.img > $@ || rm -f $@
u-boot-with-spl.sfp: spl/u-boot-spl.sfp u-boot.img FORCE
	$(call if_changed,socboot)
endif

ifneq ($(CONFIG_X86_RESET_VECTOR),)
rom: u-boot.rom FORCE

IFDTOOL=$(objtree)/tools/ifdtool
IFDTOOL_FLAGS  = -f 0:$(objtree)/u-boot.dtb
IFDTOOL_FLAGS += -m 0x$(shell $(NM) u-boot |grep _dt_ucode_base_size |cut -d' ' -f1)
IFDTOOL_FLAGS += -U $(CONFIG_SYS_TEXT_BASE):$(objtree)/u-boot-nodtb.bin
IFDTOOL_FLAGS += -w $(CONFIG_SYS_X86_START16):$(objtree)/u-boot-x86-16bit.bin
IFDTOOL_FLAGS += -C

ifneq ($(CONFIG_HAVE_INTEL_ME),)
IFDTOOL_ME_FLAGS  = -D $(srctree)/board/$(BOARDDIR)/descriptor.bin
IFDTOOL_ME_FLAGS += -i ME:$(srctree)/board/$(BOARDDIR)/me.bin
endif

ifneq ($(CONFIG_HAVE_MRC),)
IFDTOOL_FLAGS += -w $(CONFIG_X86_MRC_ADDR):$(srctree)/board/$(BOARDDIR)/mrc.bin
endif

ifneq ($(CONFIG_HAVE_FSP),)
IFDTOOL_FLAGS += -w $(CONFIG_FSP_ADDR):$(srctree)/board/$(BOARDDIR)/$(CONFIG_FSP_FILE)
endif

ifneq ($(CONFIG_HAVE_CMC),)
IFDTOOL_FLAGS += -w $(CONFIG_CMC_ADDR):$(srctree)/board/$(BOARDDIR)/$(CONFIG_CMC_FILE)
endif

ifneq ($(CONFIG_HAVE_VGA_BIOS),)
IFDTOOL_FLAGS += -w $(CONFIG_VGA_BIOS_ADDR):$(srctree)/board/$(BOARDDIR)/$(CONFIG_VGA_BIOS_FILE)
endif

quiet_cmd_ifdtool = IFDTOOL $@
cmd_ifdtool  = $(IFDTOOL) -c -r $(CONFIG_ROM_SIZE) u-boot.tmp;
ifneq ($(CONFIG_HAVE_INTEL_ME),)
cmd_ifdtool += $(IFDTOOL) $(IFDTOOL_ME_FLAGS) u-boot.tmp;
endif
cmd_ifdtool += $(IFDTOOL) $(IFDTOOL_FLAGS) u-boot.tmp;
cmd_ifdtool += mv u-boot.tmp $@

u-boot.rom: u-boot-x86-16bit.bin u-boot.bin FORCE
	$(call if_changed,ifdtool)

OBJCOPYFLAGS_u-boot-x86-16bit.bin := -O binary -j .start16 -j .resetvec
u-boot-x86-16bit.bin: u-boot FORCE
	$(call if_changed,objcopy)
endif

ifneq ($(CONFIG_SUNXI),)
OBJCOPYFLAGS_u-boot-sunxi-with-spl.bin = -I binary -O binary \
				   --pad-to=$(CONFIG_SPL_PAD_TO) --gap-fill=0xff
u-boot-sunxi-with-spl.bin: spl/sunxi-spl.bin u-boot.img FORCE
	$(call if_changed,pad_cat)
endif

ifneq ($(CONFIG_TEGRA),)
OBJCOPYFLAGS_u-boot-nodtb-tegra.bin = -O binary --pad-to=$(CONFIG_SYS_TEXT_BASE)
u-boot-nodtb-tegra.bin: spl/u-boot-spl u-boot-nodtb.bin FORCE
	$(call if_changed,pad_cat)

OBJCOPYFLAGS_u-boot-tegra.bin = -O binary --pad-to=$(CONFIG_SYS_TEXT_BASE)
u-boot-tegra.bin: spl/u-boot-spl u-boot.bin FORCE
	$(call if_changed,pad_cat)

u-boot-dtb-tegra.bin: u-boot-tegra.bin FORCE
	$(call if_changed,copy)
endif

OBJCOPYFLAGS_u-boot-app.efi := $(OBJCOPYFLAGS_EFI)
u-boot-app.efi: u-boot FORCE
	$(call if_changed,zobjcopy)

u-boot.bin.o: u-boot.bin FORCE
	$(call if_changed,efipayload)

u-boot-payload.lds: $(LDSCRIPT_EFI) FORCE
	$(call if_changed_dep,cpp_lds)

quiet_cmd_u-boot_payload ?= LD      $@
      cmd_u-boot_payload ?= $(LD) $(LDFLAGS_EFI_PAYLOAD) -o $@ \
      -T u-boot-payload.lds arch/x86/cpu/call32.o \
      lib/efi/efi.o lib/efi/efi_stub.o u-boot.bin.o \
      $(addprefix arch/$(ARCH)/lib/efi/,$(EFISTUB))

u-boot-payload: u-boot.bin.o u-boot-payload.lds FORCE
	$(call if_changed,u-boot_payload)

OBJCOPYFLAGS_u-boot-payload.efi := $(OBJCOPYFLAGS_EFI)
u-boot-payload.efi: u-boot-payload FORCE
	$(call if_changed,zobjcopy)

u-boot-img.bin: spl/u-boot-spl.bin u-boot.img FORCE
	$(call if_changed,cat)

MKIMAGEFLAGS_u-boot-spl.pbl = -n $(srctree)/$(CONFIG_SYS_FSL_PBL_RCW:"%"=%) \
		-R $(srctree)/$(CONFIG_SYS_FSL_PBL_PBI:"%"=%) -T pblimage \
		-A $(ARCH) -a $(CONFIG_SPL_TEXT_BASE)

spl/u-boot-spl.pbl: spl/u-boot-spl.bin FORCE
	$(call if_changed,mkimage)

ifeq ($(ARCH),arm)
UBOOT_BINLOAD := u-boot.img
else
UBOOT_BINLOAD := u-boot.bin
endif

OBJCOPYFLAGS_u-boot-with-spl-pbl.bin = -I binary -O binary --pad-to=$(CONFIG_SPL_PAD_TO) \
			  --gap-fill=0xff

u-boot-with-spl-pbl.bin: spl/u-boot-spl.pbl $(UBOOT_BINLOAD) FORCE
	$(call if_changed,pad_cat)

OBJCOPYFLAGS_u-boot-img-spl-at-end.bin := -I binary -O binary \
	--pad-to=$(CONFIG_UBOOT_PAD_TO) --gap-fill=0xff
u-boot-img-spl-at-end.bin: u-boot.img spl/u-boot-spl.bin FORCE
	$(call if_changed,pad_cat)

u-boot.elf: u-boot.bin
	@$(OBJCOPY)  -B aarch64 -I binary -O elf64-littleaarch64 \
		$< u-boot-elf.o
	@$(LD) u-boot-elf.o -o $@ \
		--defsym=_start=$(CONFIG_SYS_TEXT_BASE) \
		-Ttext=$(CONFIG_SYS_TEXT_BASE)

quiet_cmd_u-boot__ ?= LD      $@
      cmd_u-boot__ ?= $(LD) $(LDFLAGS) $(LDFLAGS_u-boot) -o $@ \
      -T u-boot.lds $(u-boot-init)                             \
      --start-group $(u-boot-main) --end-group                 \
      $(PLATFORM_LIBS) -Map u-boot.map

quiet_cmd_smap = GEN     common/system_map.o
cmd_smap = \
	smap=`$(call SYSTEM_MAP,u-boot) | \
		awk '$$2 ~ /[tTwW]/ {printf $$1 $$3 "\\\\000"}'` ; \
	$(CC) $(c_flags) -DSYSTEM_MAP="\"$${smap}\"" \
		-c $(srctree)/common/system_map.c -o common/system_map.o

u-boot:	$(u-boot-init) $(u-boot-main) u-boot.lds FORCE
	$(call if_changed,u-boot__)
ifeq ($(CONFIG_KALLSYMS),y)
	$(call cmd,smap)
	$(call cmd,u-boot__) common/system_map.o
endif

quiet_cmd_sym ?= SYM     $@
      cmd_sym ?= $(OBJDUMP) -t $< > $@
u-boot.sym: u-boot FORCE
	$(call if_changed,sym)

$(sort $(u-boot-init) $(u-boot-main)): $(u-boot-dirs) ;

PHONY += $(u-boot-dirs)
$(u-boot-dirs): prepare scripts
	$(Q)$(MAKE) $(build)=$@

tools: prepare

$(filter-out tools, $(u-boot-dirs)): tools

examples: $(filter-out examples, $(u-boot-dirs))

define filechk_uboot.release
	echo "$(UBOOTVERSION)$$($(CONFIG_SHELL) $(srctree)/scripts/setlocalversion $(srctree))"
endef

include/config/uboot.release: include/config/auto.conf FORCE
	$(call filechk,uboot.release)

PHONY += prepare archprepare prepare0 prepare1 prepare2 prepare3

prepare3: include/config/uboot.release
ifneq ($(KBUILD_SRC),)
	@$(kecho) '  Using $(srctree) as source for U-Boot'
	$(Q)if [ -f $(srctree)/.config -o -d $(srctree)/include/config ]; then \
		echo >&2 "  $(srctree) is not clean, please run 'make mrproper'"; \
		echo >&2 "  in the '$(srctree)' directory.";\
		/bin/false; \
	fi;
endif

prepare2: prepare3 outputmakefile

prepare1: prepare2 $(version_h) $(timestamp_h) \
                   include/config/auto.conf
ifeq ($(CONFIG_HAVE_GENERIC_BOARD),)
ifeq ($(CONFIG_SYS_GENERIC_BOARD),y)
	@echo >&2 "  Your architecture does not support generic board."
	@echo >&2 "  Please undefine CONFIG_SYS_GENERIC_BOARD in your board config file."
	@/bin/false
endif
endif
ifeq ($(wildcard $(LDSCRIPT)),)
	@echo >&2 "  Could not find linker script."
	@/bin/false
endif

archprepare: prepare1 scripts_basic

prepare0: archprepare FORCE
	$(Q)$(MAKE) $(build)=.

prepare: prepare0

define filechk_version.h
	(echo \#define PLAIN_VERSION \"$(UBOOTRELEASE)\"; \
	echo \#define U_BOOT_VERSION \"U-Boot \" PLAIN_VERSION; \
	echo \#define CC_VERSION_STRING \"$$($(CC) --version | head -n 1)\"; \
	echo \#define LD_VERSION_STRING \"$$($(LD) --version | head -n 1)\"; )
endef

define filechk_timestamp.h
	(if test -n "$${SOURCE_DATE_EPOCH}"; then \
		SOURCE_DATE="@$${SOURCE_DATE_EPOCH}"; \
		DATE=""; \
		for date in gdate date.gnu date; do \
			$${date} -u -d "$${SOURCE_DATE}" >/dev/null 2>&1 && DATE="$${date}"; \
		done; \
		if test -n "$${DATE}"; then \
			LC_ALL=C $${DATE} -u -d "$${SOURCE_DATE}" +'#define U_BOOT_DATE "%b %d %C%y"'; \
			LC_ALL=C $${DATE} -u -d "$${SOURCE_DATE}" +'#define U_BOOT_TIME "%T"'; \
			LC_ALL=C $${DATE} -u -d "$${SOURCE_DATE}" +'#define U_BOOT_TZ "%z"'; \
			LC_ALL=C $${DATE} -u -d "$${SOURCE_DATE}" +'#define U_BOOT_DMI_DATE "%m/%d/%Y"'; \
		else \
			return 42; \
		fi; \
	else \
		LC_ALL=C date +'#define U_BOOT_DATE "%b %d %C%y"'; \
		LC_ALL=C date +'#define U_BOOT_TIME "%T"'; \
		LC_ALL=C date +'#define U_BOOT_TZ "%z"'; \
		LC_ALL=C date +'#define U_BOOT_DMI_DATE "%m/%d/%Y"'; \
	fi)
endef

$(version_h): include/config/uboot.release FORCE
	$(call filechk,version.h)

$(timestamp_h): $(srctree)/Makefile FORCE
	$(call filechk,timestamp.h)

quiet_cmd_cpp_lds = LDS     $@
cmd_cpp_lds = $(CPP) -Wp,-MD,$(depfile) $(cpp_flags) $(LDPPFLAGS) -ansi \
		-D__ASSEMBLY__ -x assembler-with-cpp -P -o $@ $<

u-boot.lds: $(LDSCRIPT) prepare FORCE
	$(call if_changed_dep,cpp_lds)

spl/u-boot-spl.bin: spl/u-boot-spl
	@:
spl/u-boot-spl: tools prepare $(if $(CONFIG_OF_SEPARATE),dts/dt.dtb)
	$(Q)$(MAKE) obj=spl -f $(srctree)/scripts/Makefile.spl all

spl/sunxi-spl.bin: spl/u-boot-spl
	@:

spl/u-boot-spl.sfp: spl/u-boot-spl
	@:

spl/boot.bin: spl/u-boot-spl
	@:

tpl/u-boot-tpl.bin: tools prepare
	$(Q)$(MAKE) obj=tpl -f $(srctree)/scripts/Makefile.spl all

TAG_SUBDIRS := $(patsubst %,$(srctree)/%,$(u-boot-dirs) include)

FIND := find
FINDFLAGS := -L

tags ctags:
		ctags -w -o ctags `$(FIND) $(FINDFLAGS) $(TAG_SUBDIRS) \
						-name '*.[chS]' -print`
		ln -s ctags tags

etags:
		etags -a -o etags `$(FIND) $(FINDFLAGS) $(TAG_SUBDIRS) \
						-name '*.[chS]' -print`
cscope:
		$(FIND) $(FINDFLAGS) $(TAG_SUBDIRS) -name '*.[chS]' -print > \
						cscope.files
		cscope -b -q -k

SYSTEM_MAP = \
		$(NM) $1 | \
		grep -v '\(compiled\)\|\(\.o$$\)\|\( [aUw] \)\|\(\.\.ng$$\)\|\(LASH[RL]DI\)' | \
		LC_ALL=C sort
System.map:	u-boot
		@$(call SYSTEM_MAP,$<) > $@

checkdtc:
	@if test $(call dtc-version) -lt 0104; then \
		echo '*** Your dtc is too old, please upgrade to dtc 1.4 or newer'; \
		false; \
	fi

checkarmreloc: u-boot
	@RELOC="`$(CROSS_COMPILE)readelf -r -W $< | cut -d ' ' -f 4 | \
		grep R_A | sort -u`"; \
	if test "$$RELOC" != "R_ARM_RELATIVE" -a \
		 "$$RELOC" != "R_AARCH64_RELATIVE"; then \
		echo "$< contains unexpected relocations: $$RELOC"; \
		false; \
	fi

env: scripts_basic
	$(Q)$(MAKE) $(build)=tools/$@

tools-only: scripts_basic $(version_h) $(timestamp_h)
	$(Q)$(MAKE) $(build)=tools

tools-all: export HOST_TOOLS_ALL=y
tools-all: env tools ;

cross_tools: export CROSS_BUILD_TOOLS=y
cross_tools: tools ;

.PHONY : CHANGELOG
CHANGELOG:
	git log --no-merges U-Boot-1_1_5.. | \
	unexpand -a | sed -e 's/\s\s*$$//' > $@

include/license.h: tools/bin2header COPYING
	cat COPYING | gzip -9 -c | ./tools/bin2header license_gzip > include/license.h

CLEAN_DIRS  += $(MODVERDIR) \
	       $(foreach d, spl tpl, $(patsubst %,$d/%, \
			$(filter-out include, $(shell ls -1 $d 2>/dev/null))))

CLEAN_FILES += include/bmp_logo.h include/bmp_logo_data.h \
	       boot* u-boot* MLO* SPL System.map

MRPROPER_DIRS  += include/config include/generated spl tpl \
		  .tmp_objdiff
MRPROPER_FILES += .config .config.old include/autoconf.mk* include/config.h \
		  ctags etags TAGS cscope* GPATH GTAGS GRTAGS GSYMS

clean: rm-dirs  := $(CLEAN_DIRS)
clean: rm-files := $(CLEAN_FILES)

clean-dirs	:= $(foreach f,$(u-boot-alldirs),$(if $(wildcard $(srctree)/$f/Makefile),$f))

clean-dirs      := $(addprefix _clean_, $(clean-dirs) doc/DocBook)

PHONY += $(clean-dirs) clean archclean
$(clean-dirs):
	$(Q)$(MAKE) $(clean)=$(patsubst _clean_%,%,$@)

clean: $(clean-dirs)
	$(call cmd,rmdirs)
	$(call cmd,rmfiles)
	@find $(if $(KBUILD_EXTMOD), $(KBUILD_EXTMOD), .) $(RCS_FIND_IGNORE) \
		\( -name '*.[oas]' -o -name '*.ko' -o -name '.*.cmd' \
		-o -name '*.ko.*' -o -name '*.su' -o -name '*.cfgtmp' \
		-o -name '.*.d' -o -name '.*.tmp' -o -name '*.mod.c' \
		-o -name '*.symtypes' -o -name 'modules.order' \
		-o -name modules.builtin -o -name '.tmp_*.o.*' \
		-o -name '*.gcno' \) -type f -print | xargs rm -f

mrproper: rm-dirs  := $(wildcard $(MRPROPER_DIRS))
mrproper: rm-files := $(wildcard $(MRPROPER_FILES))
mrproper-dirs      := $(addprefix _mrproper_,scripts)

PHONY += $(mrproper-dirs) mrproper archmrproper
$(mrproper-dirs):
	$(Q)$(MAKE) $(clean)=$(patsubst _mrproper_%,%,$@)

mrproper: clean $(mrproper-dirs)
	$(call cmd,rmdirs)
	$(call cmd,rmfiles)
	@rm -f arch/*/include/asm/arch

PHONY += distclean

distclean: mrproper
	@find $(srctree) $(RCS_FIND_IGNORE) \
		\( -name '*.orig' -o -name '*.rej' -o -name '*~' \
		-o -name '*.bak' -o -name '\#*\#' -o -name '.*.orig' \
		-o -name '.*.rej' -o -name '*%' -o -name 'core' \
		-o -name '*.pyc' \) \
		-type f -print | xargs rm -f
	@rm -f boards.cfg

backup:
	F=`basename $(srctree)` ; cd .. ; \
	gtar --force-local -zcvf `LC_ALL=C date "+$$F-%Y-%m-%d-%T.tar.gz"` $$F

help:
	@echo  'Cleaning targets:'
	@echo  '  clean		  - Remove most generated files but keep the config'
	@echo  '  mrproper	  - Remove all generated files + config + various backup files'
	@echo  '  distclean	  - mrproper + remove editor backup and patch files'
	@echo  ''
	@echo  'Configuration targets:'
	@$(MAKE) -f $(srctree)/scripts/kconfig/Makefile help
	@echo  ''
	@echo  'Other generic targets:'
	@echo  '  all		  - Build all necessary images depending on configuration'
	@echo  '* u-boot	  - Build the bare u-boot'
	@echo  '  dir/            - Build all files in dir and below'
	@echo  '  dir/file.[oisS] - Build specified target only'
	@echo  '  dir/file.lst    - Build specified mixed source/assembly target only'
	@echo  '                    (requires a recent binutils and recent build (System.map))'
	@echo  '  tags/ctags	  - Generate ctags file for editors'
	@echo  '  etags		  - Generate etags file for editors'
	@echo  '  cscope	  - Generate cscope index'
	@echo  '  ubootrelease	  - Output the release version string (use with make -s)'
	@echo  '  ubootversion	  - Output the version stored in Makefile (use with make -s)'
	@echo  ''
	@echo  'Static analysers'
	@echo  '  checkstack      - Generate a list of stack hogs'
	@echo  ''
	@echo  'Documentation targets:'
	@$(MAKE) -f $(srctree)/doc/DocBook/Makefile dochelp
	@echo  ''
	@echo  '  make V=0|1 [targets] 0 => quiet build (default), 1 => verbose build'
	@echo  '  make V=2   [targets] 2 => give reason for rebuild of target'
	@echo  '  make O=dir [targets] Locate all output files in "dir", including .config'
	@echo  '  make C=1   [targets] Check all c source with $$CHECK (sparse by default)'
	@echo  '  make C=2   [targets] Force check of all c source with $$CHECK'
	@echo  '  make RECORDMCOUNT_WARN=1 [targets] Warn about ignored mcount sections'
	@echo  '  make W=n   [targets] Enable extra gcc checks, n=1,2,3 where'
	@echo  '		1: warnings which may be relevant and do not occur too often'
	@echo  '		2: warnings which occur quite often but may still be relevant'
	@echo  '		3: more obscure warnings, can most likely be ignored'
	@echo  '		Multiple levels can be combined with W=12 or W=123'
	@echo  ''
	@echo  'Execute "make" or "make all" to build all targets marked with [*] '
	@echo  'For further info see the ./README file'

%docs: scripts_basic FORCE
	$(Q)$(MAKE) $(build)=scripts build_docproc
	$(Q)$(MAKE) $(build)=doc/DocBook $@

PHONY += prepare scripts
prepare: ;
scripts: ;

endif
endif

PHONY += checkstack ubootrelease ubootversion

checkstack:
	$(OBJDUMP) -d u-boot $$(find . -name u-boot-spl) | \
	$(PERL) $(src)/scripts/checkstack.pl $(ARCH)

ubootrelease:
	@echo "$(UBOOTVERSION)$$($(CONFIG_SHELL) $(srctree)/scripts/setlocalversion $(srctree))"

ubootversion:
	@echo $(UBOOTVERSION)

ifeq ($(KBUILD_EXTMOD),)
        build-dir  = $(patsubst %/,%,$(dir $@))
        target-dir = $(dir $@)
else
        zap-slash=$(filter-out .,$(patsubst %/,%,$(dir $@)))
        build-dir  = $(KBUILD_EXTMOD)$(if $(zap-slash),/$(zap-slash))
        target-dir = $(if $(KBUILD_EXTMOD),$(dir $<),$(dir $@))
endif

%.s: %.c prepare scripts FORCE
	$(Q)$(MAKE) $(build)=$(build-dir) $(target-dir)$(notdir $@)
%.i: %.c prepare scripts FORCE
	$(Q)$(MAKE) $(build)=$(build-dir) $(target-dir)$(notdir $@)
%.o: %.c prepare scripts FORCE
	$(Q)$(MAKE) $(build)=$(build-dir) $(target-dir)$(notdir $@)
%.lst: %.c prepare scripts FORCE
	$(Q)$(MAKE) $(build)=$(build-dir) $(target-dir)$(notdir $@)
%.s: %.S prepare scripts FORCE
	$(Q)$(MAKE) $(build)=$(build-dir) $(target-dir)$(notdir $@)
%.o: %.S prepare scripts FORCE
	$(Q)$(MAKE) $(build)=$(build-dir) $(target-dir)$(notdir $@)
%.symtypes: %.c prepare scripts FORCE
	$(Q)$(MAKE) $(build)=$(build-dir) $(target-dir)$(notdir $@)

/: prepare scripts FORCE
	$(cmd_crmodverdir)
	$(Q)$(MAKE) KBUILD_MODULES=$(if $(CONFIG_MODULES),1) \
	$(build)=$(build-dir)
%/: prepare scripts FORCE
	$(cmd_crmodverdir)
	$(Q)$(MAKE) KBUILD_MODULES=$(if $(CONFIG_MODULES),1) \
	$(build)=$(build-dir)
%.ko: prepare scripts FORCE
	$(cmd_crmodverdir)
	$(Q)$(MAKE) KBUILD_MODULES=$(if $(CONFIG_MODULES),1)   \
	$(build)=$(build-dir) $(@:.ko=.o)
	$(Q)$(MAKE) -f $(srctree)/scripts/Makefile.modpost

quiet_cmd_rmdirs = $(if $(wildcard $(rm-dirs)),CLEAN   $(wildcard $(rm-dirs)))
      cmd_rmdirs = rm -rf $(rm-dirs)

quiet_cmd_rmfiles = $(if $(wildcard $(rm-files)),CLEAN   $(wildcard $(rm-files)))
      cmd_rmfiles = rm -f $(rm-files)

targets := $(wildcard $(sort $(targets)))
cmd_files := $(wildcard .*.cmd $(foreach f,$(targets),$(dir $(f)).$(notdir $(f)).cmd))

ifneq ($(cmd_files),)
  $(cmd_files): ;
  include $(cmd_files)
endif

endif

PHONY += FORCE
FORCE:

.PHONY: $(PHONY)
