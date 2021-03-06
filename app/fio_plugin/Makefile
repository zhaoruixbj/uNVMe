ifeq ($(KV_DIR),)
export KV_DIR = $(shell pwd)/../../
endif

include $(KV_DIR)/mk/unvme.mk

# if you want specific version fio instead of system's fio, enable both  below options 
# ex) fio-3.3 -> FIO_MAJOR_VERSION=3, FIO_MINOR_VERSION=3
#FIO_MAJOR_VERSION=3
#FIO_MINOR_VERSION=3

ifneq ($(FIO_MAJOR_VERSION), )
ifneq ($(FIO_MINOR_VERSION), )
SYSTEM_FIO_VERSION = fio-$(FIO_MAJOR_VERSION).$(FIO_MINOR_VERSION)
endif
endif

ifeq ($(SYSTEM_FIO_VERSION), )
SYSTEM_FIO_VERSION := $(shell fio -version)
VERSION_ONLY = $(shell echo $(SYSTEM_FIO_VERSION) |cut -d'-' -f 2)
FIO_MAJOR_VERSION := $(shell echo $(VERSION_ONLY) |cut -d'.' -f 1)
FIO_MINOR_VERSION := $(shell echo $(VERSION_ONLY) |cut -d'.' -f 2)
endif

# if system fio does not existing
ifeq ($(SYSTEM_FIO_VERSION), )
$(error fio not found)
endif

ifeq (,$(wildcard $(KV_DIR)/app/external/fio/$(SYSTEM_FIO_VERSION).tar.gz))
NOT_SUPPORTED_FIO_VERSION = 1
FIO_MAJOR_VERSION = 2
FIO_MINOR_VERSION = 18
SYSTEM_FIO_VERSION = fio-$(FIO_MAJOR_VERSION).$(FIO_MINOR_VERSION)
endif

CURRENT_DIR=$(shell pwd)
FIO_SOURCE_DIR := fio-$(SYSTEM_FIO_VERSION)
LIBS += $(KV_DIR)/bin/libuio.a -lm -pthread -lrt -ldl -luuid
CFLAGS += -O2 -march=native -m64 -D_GNU_SOURCE -I../../include -I../../driver/include -I../external/fio/$(FIO_SOURCE_DIR) -I../../driver/external/$(SPDK_PATH)/include -std=gnu99 -fPIC -DFIO_MAJOR_VERSION=$(FIO_MAJOR_VERSION) -DFIO_MINOR_VERSION=$(FIO_MINOR_VERSION)
override LDFLAGS += -shared -rdynamic

APP = unvme2_fio_plugin
C_SRCS = $(APP:%=%.c)
OBJS = $(C_SRCS:.c=.o)
LINK_C = $(CC) -o $@ $(LDFLAGS) $(OBJS) $(LIBS)

ifeq (,$(wildcard $(KV_DIR)/app/external/fio/$(FIO_SOURCE_DIR)/config-host.h))
all: prepare $(APP)
else
all: $(APP)
endif

$(APP): $(OBJS)
	$(LINK_C)
	@cd $(KV_DIR)/app/external/fio/$(FIO_SOURCE_DIR) && cp fio $(CURRENT_DIR)/$(SYSTEM_FIO_VERSION)
ifdef NOT_SUPPORTED_FIO_VERSION
	$(info **** NOTE:: Your system's FIO version is not supported. Use default(fio-$(FIO_MAJOR_VERSION).$(FIO_MINOR_VERSION)))
endif
prepare:
	@cd $(KV_DIR)/app/external/fio && tar -xvf $(SYSTEM_FIO_VERSION).tar.gz
	@cd $(KV_DIR)/app/external/fio/$(FIO_SOURCE_DIR) && ./configure && make -j 4 
clean:
	rm -f $(OBJS) $(APP) fio-*
