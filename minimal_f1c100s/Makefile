#
# Machine makefile
#

.PHONY:clean
.PHONY:write
.PHONY:mktool
.PHONY:dump

BUILD ?= build
RM = rm
ECHO = @echo
CP = cp
MKDIR = mkdir
SED = sed
PYTHON = python

CROSS_COMPILE = ../gcc-linaro-5.3.1-2016.05-x86_64_arm-eabi/bin/arm-eabi-
CC	          = $(CROSS_COMPILE)gcc
AS	          = $(CROSS_COMPILE)gcc -x assembler-with-cpp
LD	          = $(CROSS_COMPILE)ld
OBJCOPY	      = $(CROSS_COMPILE)objcopy
OBJDUMP       = $(CROSS_COMPILE)objdump
SIZE 		  = $(CROSS_COMPILE)size

SRC_C = \
	main.c \
	machine/sys-clock.c \
	machine/sys-dram.c \
	machine/sys-uart.c \
	machine/sys-copyself.c \
	machine/sys-spi-flash.c \
	machine/sys-mmu.c \
	machine/exception.c \

# drivers
SRC_C += \
	driver/gpio-f1c100s.c \
	driver/pwm-f1c100s.c \
	driver/reset-f1c100s.c \
	lib/malloc.c \
	lib/dma.c \
	driver/clk-f1c100s-pll.c \
	driver/fb-f1c100s.c \


SRC_ASM = machine/start.S \
		  arch/arm32/lib/memcpy.S \
		  arch/arm32/lib/memset.S \

OBJ = $(addprefix $(BUILD)/, $(SRC_C:.c=.o)) $(addprefix $(BUILD)/, $(SRC_ASM:.S=.o))

DEFINES		+= -D__ARM32_ARCH__=5 -D__ARM926EJS__

ASFLAGS		:= -g -ggdb -Wall -O3 -ffreestanding -std=gnu99 $(DEFINES)
CFLAGS		:= -g -ggdb -Wall -O3 -ffreestanding -std=gnu99 $(DEFINES)
CXXFLAGS	:= -g -ggdb -Wall -O3 -ffreestanding -std=gnu99 $(DEFINES)
LDFLAGS		:= -T f1c100s.ld -nostdlib
MCFLAGS		:= -march=armv5te -mtune=arm926ej-s -mfloat-abi=soft -marm -mno-thumb-interwork

LIBDIRS		:=
LIBS 		:= -lgcc
INCDIRS		:= -Imachine/include -Iarch/arm32/include
SRCDIRS		:=


INCDIRS		+= -Idriver/include -Ilib

# $(BUILD)/firmware_boot.bin:$(BUILD)/firmware.bin
# 	./tools/mksunxiboot/mksunxiboot $^ $@

$(BUILD)/firmware.bin: $(BUILD)/firmware.elf
	$(OBJCOPY) -v -O binary $^ $@
	@echo Make header information for brom booting
	@$(MKSUNXI) $@

$(BUILD)/firmware.elf: $(OBJ)
	$(ECHO) "LINK $@"
	$(CC) $(LDFLAGS) -Wl,--cref,-Map=$@.map -o $@ $^ $(LIBS)
	$(SIZE) $@


$(BUILD)/%.o: %.S
	$(ECHO) "AS $<"
	$(AS) $(MCFLAGS) $(ASFLAGS) -c -o $@ $<
	
$(BUILD)/%.o: %.s
	$(ECHO) "AS $<"
	$(AS) $(MCFLAGS) $(CFLAGS) -o $@ $<

define compile_c
$(ECHO) "CC $<"
$(CC) $(INCDIRS) $(MCFLAGS) $(CFLAGS) -c -MD -o $@ $<
@# The following fixes the dependency file.
@# See http://make.paulandlesley.org/autodep.html for details.
@# Regex adjusted from the above to play better with Windows paths, etc.
@$(CP) $(@:.o=.d) $(@:.o=.P); \
  $(SED) -e 's/#.*//' -e 's/^.*:  *//' -e 's/ *\\$$//' \
      -e '/^$$/ d' -e 's/$$/ :/' < $(@:.o=.d) >> $(@:.o=.P); \
  $(RM) -f $(@:.o=.d)
endef

$(BUILD)/%.o: %.c
	$(call compile_c)

OBJ_DIRS = $(sort $(dir $(OBJ)))
$(OBJ): | $(OBJ_DIRS)
$(OBJ_DIRS):
	$(MKDIR) -p $@

clean:
	find . -name "*.o"  | xargs rm -f
	find . -name "*.bin"  | xargs rm -f
	find . -name "*.elf"  | xargs rm -f
	find . -name "*.P"  | xargs rm -f
#rm -rf *.o *.bin 
#rm -rf $(BUILD)/*.o $(BUILD)/*.bin

write:
	sudo sunxi-fel -p spiflash-write 0 $(BUILD)/firmware.bin

mktool:
	cd tools/mksunxiboot && make
	cd tools/mksunxi && make

dump:
	$(OBJDUMP) -S myboot.o | less

MKSUNXI		:= tools/mksunxi/mksunxi

mkboot:
	@echo Make header information for brom booting
	@$(MKSUNXI) $(BUILD)/firmware.bin
