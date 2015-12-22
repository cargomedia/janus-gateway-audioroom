ACLOCAL_AMFLAGS = -I m4
CFLAGS = $(shell pkg-config --cflags glib-2.0)
CXXFLAGS = $(CFLAGS)
lib_LTLIBRARIES = libjanus_audioroom.la
libjanus_audioroom_la_SOURCES = libjanus_audioroom.c
libjanus_audioroom_la_LDFLAGS = -version-info 0:0:0 $(shell pkg-config --libs glib-2.0 opus)

LIBS = $(shell pkg-config --libs opus glib-2.0)

confdir = $(exec_prefix)/etc/janus
libdir = $(exec_prefix)/lib/janus/plugins
conf_DATA = conf/janus.plugin.cm.audioroom.cfg.sample

##
# Configuration
##

config:
	$(MKDIR_P) $(DESTDIR)$(confdir)
	$(foreach config,$(conf_DATA),cp "$(CURDIR)/$(config)" "$(DESTDIR)$(confdir)/$(notdir $(basename $(config) .sample))";)
