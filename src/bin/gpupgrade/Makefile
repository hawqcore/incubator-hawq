#-------------------------------------------------------------------------
#
# Makefile for src/bin/gpupgrade
#
# Portions Copyright (c) 2007, Greenplum
#
# $PostgreSQL$
#
#-------------------------------------------------------------------------

PGFILEDESC = "gpupgrade - Greenplum upgrader"
subdir = src/bin/gpupgrade
top_builddir = ../../..
include $(top_builddir)/src/Makefile.global

# The frontend doesn't need everything that's in LIBS, some are backend only
LIBS := $(filter-out -lresolv -lbz2, $(LIBS))
LIBS := $(filter-out -lreadline -ledit -ltermcap -lncurses -lcurses -lcurl -lssl -lz -lpam -lkrb5 -lgssapi_krb5 -lcom_err -lk5crypto -lnuma -lkrb5support, $(LIBS))

MODCAT_OBJS= gpmodcatversion.o pg_crc.o crc32c.o
DATADIR=$(top_srcdir)/src/test/regress/data

UPGRADEFILES = $(DATADIR)/upgrade/upg2_catupgrade.sql \
			   $(DATADIR)/upgrade20/upg2_catupgrade_20.sql

DATAFILES = $(DATADIR)/upgrade20/pg_resqueue.data \
            $(DATADIR)/upgrade20/pg_database.data \
            $(DATADIR)/upgrade20/pg_proc.data \
            $(DATADIR)/upgrade20/pg_description.data

# This sets things up to create a "contrib" module for upgrade extensions
MODULE_big = gpupgrade
OBJS= gpviewcp.o
include $(top_srcdir)/contrib/contrib-global.mk

# In addition to the contrib module we are also building the gpmodcatversion program and the upgrade scripts
all: gpmodcatversion upgradesql

clean distclean maintainer-clean: clean-lib gpmodcat_clean upgradesql_clean

install: all installdirs installupgradefiles gpmodcat_install

uninstall: gpmodcat_uninstall

# build the gpmodcatversion program and the upgrade script
gpmodcatversion: $(MODCAT_OBJS)
	$(CC) $(CFLAGS) $^ $(LDFLAGS) $(LIBS) -o $@$(X)

pg_crc.c: $(top_srcdir)/src/backend/utils/hash/pg_crc.c
	rm -f $@ && $(LN_S) $< .
	
crc32c.c: $(top_srcdir)/src/backend/utils/hash/crc32c.c
	rm -f $@ && $(LN_S) $< .

upgradesql:
	@for i in $(UPGRADEFILES); \
	do \
        sed -e 's/\@gpupgradeschemaname\@/pg_catalog/g' -e 's/\@gpupgradetoolkitschema\@/hawq_toolkit/g' $$i.in > $$i; \
	done
	
gpmodcat_clean:
	rm -f gpmodcatversion$(X)
	rm -f $(MODCAT_OBJS)
	rm -f pg_crc.c
	rm -f crc32c.c
	
upgradesql_clean:
	rm -f $(DATADIR)/upgrade42/*.sql

gpmodcat_install: gpmodcatversion
	$(INSTALL_PROGRAM) gpmodcatversion$(X) \
		'$(DESTDIR)$(bindir)/lib'/gpmodcatversion$(X)

installdirs:
	$(MKDIR_P) '$(DESTDIR)$(bindir)' '$(DESTDIR)$(bindir)/lib'
	$(MKDIR_P) '$(DESTDIR)$(datadir)' '$(DESTDIR)$(datadir)/upgrade'

installupgradefiles: $(UPGRADEFILES) $(DATAFILES)
	@for i in $^; \
	do \
		$(INSTALL_DATA) $$i '$(DESTDIR)$(datadir)/upgrade/'`basename $$i`; \
	done

gpmodcat_unistall:
	rm -f '$(DESTDIR)$(bindir)'/lib, gpmodcatversion$(X)

