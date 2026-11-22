# caveutils - small tools that have no GNU counterpart (or do something
# different from it). One file each, built to bin/NAME.
#
#   make              build everything
#   make cloc         build one tool: bin/cloc
#   make test         run tests/run, fzf, gtop, snip, notts, cb (wayland and x11)
#   make install      PREFIX=/usr/local, binaries and man pages (sudo is NOT setuid)
#   make clean
#
# Headers the tools share live in deps/ (a handful of single-file headers).

.DEFAULT_GOAL := all
CC      ?= cc
CFLAGS  ?= -O2
CSTD     = -std=gnu11 -D_GNU_SOURCE -U_FORTIFY_SOURCE   # hardened glibc headers clash with base.h
OUT     ?= bin
PREFIX  ?= /usr/local
INC      = -Ideps -Ideps/utils -Ideps/libutil -Ideps/singlefile/abstract

TOOLS   := cb cloc colorado dv fzf gtop man netcat notts scenario shot snip sudo tgates tree

LIBS_cb   = -lX11 -lwayland-client
LIBS_sudo = -lpam
LIBS_fzf  = -pthread -lm
LIBS_gtop = -lm

# cb (wayland backend): protocol glue is generated with wayland-scanner at build time
EXT_XML  ?= /usr/share/wayland-protocols/staging/ext-data-control/ext-data-control-v1.xml
WLR_XML   = deps/wlr-data-control-unstable-v1.xml
GEN       = $(OUT)/gen
INC_cb  = -I$(GEN)
SRC_cb  = $(GEN)/ext-data-control-v1-protocol.c $(GEN)/wlr-data-control-unstable-v1-protocol.c \
            $(GEN)/ext-data-control-v1-client-protocol.h $(GEN)/wlr-data-control-unstable-v1-client-protocol.h

$(GEN)/ext-data-control-v1-protocol.c: $(EXT_XML) | $(GEN)
	wayland-scanner private-code $< $@
$(GEN)/ext-data-control-v1-client-protocol.h: $(EXT_XML) | $(GEN)
	wayland-scanner client-header $< $@
$(GEN)/wlr-data-control-unstable-v1-protocol.c: $(WLR_XML) | $(GEN)
	wayland-scanner private-code $< $@
$(GEN)/wlr-data-control-unstable-v1-client-protocol.h: $(WLR_XML) | $(GEN)
	wayland-scanner client-header $< $@
$(GEN):
	mkdir -p $(GEN)

all: $(addprefix $(OUT)/,$(TOOLS))

$(OUT):
	mkdir -p $(OUT)

define tool
$(OUT)/$(1): src/$(1).c $$(SRC_$(1)) | $(OUT)
	$$(CC) $$(CSTD) $$(CFLAGS) -w $$(INC) $$(INC_$(1)) -o $$@ src/$(1).c $$(filter %.c,$$(SRC_$(1))) $$(LIBS_$(1))
.PHONY: $(1)
$(1): $(OUT)/$(1)
endef
$(foreach t,$(TOOLS),$(eval $(call tool,$(t))))

test: all
	r=0; sh tests/run || r=1; sh tests/fzf || r=1; sh tests/gtop || r=1; sh tests/snip || r=1; sh tests/notts || r=1; sh tests/cb || r=1; sh tests/cb-x11 || r=1; exit $$r

install: all
	mkdir -p $(DESTDIR)$(PREFIX)/bin $(DESTDIR)$(PREFIX)/share/man/man1
	for t in $(TOOLS); do install -m 755 $(OUT)/$$t $(DESTDIR)$(PREFIX)/bin/$$t; done
	for t in $(TOOLS); do [ -f manpages/$$t.1 ] && install -m 644 manpages/$$t.1 $(DESTDIR)$(PREFIX)/share/man/man1/$$t.1; done; true
	mkdir -p $(DESTDIR)$(PREFIX)/share/gtop/commands
	install -m 755 share/gtop/commands/* $(DESTDIR)$(PREFIX)/share/gtop/commands/
	@echo "sudo was installed WITHOUT the setuid bit; add it yourself if you mean to use it"

clean:
	rm -rf $(OUT)

.PHONY: all test install clean
