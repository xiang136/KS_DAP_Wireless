TARGET ?= f723host
BOARD ?= stm32f723e_disco

BOARD_PATH = ../boards/$(BOARD)
include $(BOARD_PATH)/board.mk
LOG_INFO ?= 1
# TeenyDT format descriptor
TEENYDT = host_desc.lua
USBMEM = 512
USBEP = 5
# project source
SOURCES  = main.c retarget_stdio.c msc_diskio.c
INCLUDES = . 
DEFINES  = __forceinline=inline NO_DEVICE LOG_INFO=$(LOG_INFO)
# FatFs source
SOURCES  += ../../third_part/fatfs/source/ff.c
INCLUDES += ../../third_part/fatfs/source

# board source
SOURCES  += $(BOARD_SRC)
INCLUDES += $(BOARD_INC)
DEFINES  += $(BOARD_DEF)

# Teeny USB source
TUSB_PATH = ../../usb_stack
SOURCES  += $(wildcard $(TUSB_PATH)/src/*.c)
INCLUDES += $(TUSB_PATH)/inc
INCLUDES += $(TUSB_PATH)/../mcu_lib/st_driver/CMSIS/Include

# Teeny USB device class source
INCLUDES += $(TUSB_PATH)/class

# Teeny USB host class source
INCLUDES += $(TUSB_PATH)/class/host
SOURCES  += $(wildcard $(TUSB_PATH)/class/host/*.c)

TOOLSET     ?= arm-none-eabi-
CC           = $(TOOLSET)gcc
LD           = $(TOOLSET)gcc
AR           = $(TOOLSET)ar
AS           = $(TOOLSET)as
SIZE         = $(TOOLSET)size
OBJCOPY      = $(TOOLSET)objcopy
RM           = rm -rf
CP           = cp
MKDIR        = mkdir -p

CPU ?= cortex-m3
CCFLAGS     += -Wall -mcpu=$(CPU) -mthumb -g -Os -MMD
CCFLAGS     += $(addprefix -D, $(DEFINES)) 
CCFLAGS     += $(addprefix -I, $(INCLUDES))
CCFLAGS     += -ffunction-sections -fdata-sections -specs=nano.specs -specs=nosys.specs
# used for static assert
CCFLAGS     += -Wno-unused-local-typedefs
LDFLAGS     += $(CCFLAGS)
LDFLAGS     += -Wl,--script='$(LDSCRIPT)' -Wl,-Map=$(OUTDIR)/$(TARGET).map
LDFLAGS     += -Wl,--gc-sections

OUTDIR = ./output
OBJDIR = ./obj/$(TARGET)
OBJECTS      = $(addprefix $(OBJDIR)/, $(addsuffix .o, $(notdir $(basename $(SOURCES)))))
DEPENDS      = $(addprefix $(OBJDIR)/, $(addsuffix .d, $(notdir $(basename $(filter %c, $(SOURCES))))))
SRCPATH      = $(sort $(dir $(SOURCES)))
vpath %.c $(SRCPATH)
vpath %.S $(SRCPATH)
vpath %.s $(SRCPATH)

all: f107 f407 f723 f767

f107:
	@echo Building stm32f107vc
	@$(MAKE) demo TARGET='f107host' BOARD='stm32f107vc'
f407:
	@echo Building stm32f407igt6
	@$(MAKE) demo TARGET='f407host' BOARD='stm32f407_evk'
f723:
	@echo Building stm32f723iek6
	@$(MAKE) demo TARGET='f723host' BOARD='stm32f723e_disco'
f767:
	@echo Building stm32f767zit6
	@$(MAKE) demo TARGET='f767host' BOARD='stm32f767zi_nucleo'

demo: $(OUTDIR) $(OUTDIR)/$(TARGET).bin $(OUTDIR)/$(TARGET).hex

$(OBJDIR)/%.o:%.c
	@echo building $<
	@$(CC) $(CCFLAGS) -c $< -o $@
$(OBJDIR)/%.o:%.s
	@echo building $<
	@$(CC) $(ASFLAGS) -c $< -o $@
$(OBJDIR)/%.o:%.S
	@echo building $<
	@$(CC) $(ASFLAGS) -c $< -o $@

$(OBJDIR):
	@$(MKDIR) $@
$(OUTDIR):
	@$(MKDIR) $@

$(OUTDIR)/$(TARGET).hex:$(OUTDIR)/$(TARGET).elf
	@echo generating $@
	@$(OBJCOPY) -Oihex $< $@
$(OUTDIR)/$(TARGET).bin:$(OUTDIR)/$(TARGET).elf
	@echo generating $@
	@$(OBJCOPY) -Obinary $< $@
$(OUTDIR)/$(TARGET).elf: $(OBJDIR) $(OBJECTS)
	@echo linking $@
	@$(LD) $(LDFLAGS) $(OBJECTS) -o $@
	@$(SIZE) $@

USBMEM ?= 512
USBEP  ?= 7

desc: teeny_usb_desc.c

PWD = $(shell pwd)
teeny_usb_init.h teeny_usb_desc.c:$(TEENYDT)
	@echo generate USB descriptor
	cd $(PWD)/../../TeenyDT && \
	lua gen_descriptor.lua $(PWD)/$< -maxmem=$(USBMEM) -maxep=$(USBEP) && \
	cd $(PWD)

clean:
	$(RM) $(OUTDIR)/* $(OBJDIR)/*
clear:
	$(RM) $(OUTDIR) obj

.PHONY: all clean clear

-include $(DEPENDS)
