LINUX=$(shell uname | grep Linux | wc -l | xargs echo)
DEPS:=../deps

#
# In the event we have a system zeromq, we can skip this whole build process and use that
#
ifndef ZEROMQ_PREFIX
# poor man's realpath
#ZEROMQ_PREFIX := $(shell readlink -f "$(DEPS)/local")
ZEROMQ_PREFIX := $(abspath $(DEPS)/local)
LOCAL_ZEROMQ=libzmq.a
endif


ZMQ_FLAGS_BASE = --with-libsodium=$(ZEROMQ_PREFIX) --prefix $(ZEROMQ_PREFIX)

ifeq ($(LINUX),1)
ZMQ_FLAGS=--with-pic --with-pgm $(ZMQ_FLAGS_BASE)
else
ZMQ_FLAGS=$(ZMQ_FLAGS_BASE)
endif

ifndef ZEROMQ_VERSION
ZEROMQ_VERSION=4.0.5
endif

ifndef LIBSODIUM_VERSION
LIBSODIUM_VERSION=1.0.2
endif

all: $(LOCAL_ZEROMQ)

clean:
	if test -e $(DEPS)/zeromq4/Makefile; then \
		cd $(DEPS)/zeromq4; make clean; \
	else \
		true; \
	fi
	if test -e $(DEPS)/libsodium/Makefile; then \
		cd $(DEPS)/libsodium; make clean;\
	else \
		true; \
	fi
	rm -rf $(DEPS)/local

distclean:
	@rm -rf $(DEPS)

$(DEPS):
	@mkdir -p $(DEPS)

# Libzmq
$(DEPS)/zeromq-$(ZEROMQ_VERSION).tar.gz: $(DEPS)
	@curl http://download.zeromq.org/zeromq-$(ZEROMQ_VERSION).tar.gz -o $(DEPS)/zeromq-$(ZEROMQ_VERSION).tar.gz	

$(DEPS)/zeromq4: $(DEPS)/zeromq-$(ZEROMQ_VERSION).tar.gz
	@mkdir -p $(DEPS)/zeromq4
	@cd $(DEPS) && tar xzfp zeromq-$(ZEROMQ_VERSION).tar.gz -C zeromq4 --strip-components=1

$(ZEROMQ_PREFIX)/lib/libzmq.a: $(DEPS)/zeromq4 $(ZEROMQ_PREFIX)/lib/libsodium.a
	cd $(DEPS)/zeromq4 && ./configure $(ZMQ_FLAGS) && make && make install

libzmq.a: $(ZEROMQ_PREFIX)/lib/libzmq.a


# Libsodium
$(DEPS)/libsodium-$(LIBSODIUM_VERSION).tgz: $(DEPS)
	@curl https://github.com/jedisct1/libsodium/tarball/$(LIBSODIUM_VERSION) -o $(DEPS)/libsodium-$(LIBSODIUM_VERSION).tgz -L

$(DEPS)/libsodium: $(DEPS)/libsodium-$(LIBSODIUM_VERSION).tgz
	@mkdir -p $(DEPS)/libsodium
	@tar xzfp $(DEPS)/libsodium-$(LIBSODIUM_VERSION).tgz -C $(DEPS)/libsodium --strip-components=1

$(ZEROMQ_PREFIX)/lib/libsodium.a: $(DEPS)/libsodium
	cd $(DEPS)/libsodium && ./autogen.sh && ./configure --prefix $(ZEROMQ_PREFIX) && make && make install

libsodium.a: $(ZEROMQ_PREFIX)/lib/libsodium.a

