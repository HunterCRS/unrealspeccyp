FLAVOR := release
#FLAVOR := debug

#DEVICE := a320
DEVICE := gcw0

$(info Building for $(DEVICE) in $(FLAVOR) mode...)

SRC_PATH := ../..
OUTPUT_BASE := $(DEVICE)-$(FLAVOR)

PROJECT_NAME := unreal_speccy_portable
SRCFOLDERS := . z80 tools tools/tinyxml snapshot ui platform platform/custom_ui platform/linux platform/sdl devices devices/fdd devices/input devices/sound
CXXSRCS := $(foreach dir, $(SRCFOLDERS), $(wildcard $(SRC_PATH)/$(dir)/*.cpp))
CSRCS := $(foreach dir, $(SRCFOLDERS), $(wildcard $(SRC_PATH)/$(dir)/*.c))
CSRCS += $(SRC_PATH)/tools/zlib/ioapi.c $(SRC_PATH)/tools/zlib/unzip.c

CXX := mipsel-$(DEVICE)-linux-uclibc-g++
C := mipsel-$(DEVICE)-linux-uclibc-gcc
SYSROOT := /opt/$(DEVICE)-toolchain/usr/mipsel-$(DEVICE)-linux-uclibc/sysroot/

DIST_PATH := $(OUTPUT_BASE)/dist
OBJ_PATH := $(OUTPUT_BASE)/obj
CXX_OBJS := $(patsubst $(SRC_PATH)/%.cpp,$(OBJ_PATH)/%.o,$(CXXSRCS))
CXX_DEPS := $(patsubst $(SRC_PATH)/%.cpp,$(OBJ_PATH)/%.d,$(CXXSRCS))
C_OBJS := $(patsubst $(SRC_PATH)/%.c,$(OBJ_PATH)/%.o,$(CSRCS))
C_DEPS := $(patsubst $(SRC_PATH)/%.c,$(OBJ_PATH)/%.d,$(CSRCS))
BIN_TARGET := $(DIST_PATH)/$(PROJECT_NAME)
OPK_TARGET := $(DIST_PATH)/$(PROJECT_NAME).opk

DESKTOP_ENTRY := default.$(DEVICE).desktop
ICON := icon-32x32.png

STD_OPTS := -G0 -Wall -c -fmessage-length=0 -fno-builtin -fno-exceptions -fno-pic
ifeq ($(FLAVOR),debug)
STD_OPTS += -g -O0 -DDEBUG
else
STD_OPTS += -O3 -fomit-frame-pointer -DNDEBUG
endif
CXXFLAGS := $(STD_OPTS) -fno-rtti -fno-threadsafe-statics \
		-D_LINUX -DUSE_SDL -DSDL_KEYS_DINGOO \
		$(shell $(SYSROOT)/usr/bin/sdl-config --cflags) \
		$(shell $(SYSROOT)/usr/bin/libpng-config --cflags)
CFLAGS   := $(STD_OPTS)
LFLAGS   := -s $(shell $(SYSROOT)/usr/bin/sdl-config --libs) \
		$(shell $(SYSROOT)/usr/bin/libpng-config --libs) -lz

all: build

$(CXX_OBJS): $(OBJ_PATH)/%.o : $(SRC_PATH)/%.cpp
	@mkdir -p $(@D)
	$(CXX) $(CXXFLAGS) -o $@ $<

$(C_OBJS): $(OBJ_PATH)/%.o : $(SRC_PATH)/%.c
	@mkdir -p $(@D)
	$(C) $(CFLAGS) -o $@ $<

$(BIN_TARGET): $(CXX_OBJS) $(C_OBJS)
	@mkdir -p $(@D)
	$(CXX) $(LFLAGS) $^ -o$(BIN_TARGET)

$(OPK_TARGET): $(BIN_TARGET) $(DESKTOP_ENTRY) $(ICON)
	mksquashfs $^ $(SRC_PATH)/res $@ -noappend -comp gzip -all-root

build: $(OPK_TARGET)

clean:
	rm -rf $(CXX_OBJS)
	rm -rf $(CXX_DEPS)
	rm -rf $(C_OBJS)
	rm -rf $(C_DEPS)
	rm -rf $(BIN_TARGET)

.PHONY: mkdirs clean build all
