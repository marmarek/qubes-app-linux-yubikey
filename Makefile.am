# Written by Simon Josefsson <simon@yubico.com>.
# Copyright (c) 2006-2013 Yubico AB
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are
# met:
#
#     * Redistributions of source code must retain the above copyright
#       notice, this list of conditions and the following disclaimer.
#
#     * Redistributions in binary form must reproduce the above
#       copyright notice, this list of conditions and the following
#       disclaimer in the documentation and/or other materials provided
#       with the distribution.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
# "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
# LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
# A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
# OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
# SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
# LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
# DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
# THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

ACLOCAL_AMFLAGS = -I m4

AM_CFLAGS = $(WARN_CFLAGS)

libdir = $(PAMDIR)

lib_LTLIBRARIES = pam_qubes_yubico.la

#pam_yubico_la_SOURCES = pam_yubico.c util.h util.c drop_privs.h drop_privs.c
# dropping util as we are not doing challenge response in Qubes
# dropping drop_privs as we are not using user config files
pam_qubes_yubico_la_SOURCES = pam_qubes_yubico.c
# XXX add -Wl,-x too?  PAM documentation suggests it.
#pam_yubico_la_LIBADD = @LTLIBYKCLIENT@ @LIBLDAP@ @YKPERS_LIBS@ @LTLIBYUBIKEY@ -lpam
# dropping ykclient as validation is local
# dropping LDAP as single user
# dropping pers which is a dependancy for challenge response
pam_qubes_yubico_la_LIBADD = @LTLIBYUBIKEY@ -lpam
pam_qubes_yubico_la_LDFLAGS = -module -avoid-version
# dropping as no pers requirements.
#pam_yubico_la_CPPFLAGS = @YKPERS_CFLAGS@

DEFS = -DDEBUG_PAM -DPAM_DEBUG @DEFS@

# The command line tools.

#if YKPERS
#bin_PROGRAMS = ykpamcfg
#endif

#ykpamcfg_SOURCES = ykpamcfg.c util.c
#ykpamcfg_LDADD = @LTLIBYKCLIENT@ @YKPERS_LIBS@ @LTLIBYUBIKEY@
#ykpamcfg_CPPFLAGS = @YKPERS_CFLAGS@

#if YKPERS
#dist_man1_MANS = ykpamcfg.1
#endif

dist_man8_MANS = pam_qubes_yubico.8

# Self tests.

AM_LDFLAGS = -no-install
LDADD = ./pam_qubes_yubico.la

check_PROGRAMS = test
TESTS = $(check_PROGRAMS)

# Dist docs from wiki.
EXTRA_DIST = doc/ReadMe.asciidoc doc/TwoFactorPAMConfiguration.asciidoc doc/YubiKeyAndOpenVPNviaPAM.asciidoc

# Release

ChangeLog:
	cd $(srcdir) && git2cl > ChangeLog

check-doc-dist:
	cd $(srcdir) && git submodule update
	cd $(srcdir)/doc && git checkout master && git pull
	perl -pe "s,^EXTRA_DIST = .*,EXTRA_DIST = `cd $(srcdir) && ls doc/*.asciidoc | grep -v Todo.ascii | xargs echo`," \
		< $(srcdir)/Makefile.am > check-doc-dist.tmp
	diff -ur $(srcdir)/Makefile.am check-doc-dist.tmp || \
		(rm -f check-doc-dist.tmp; echo 'error: please update $(srcdir)/Makefile.am to include all docs'; exit 1)
	rm -f check-doc-dist.tmp
	diff -ur $(srcdir)/README $(srcdir)/doc/ReadMe.asciidoc || \
		(echo 'error: README and doc/ReadMe.asciidoc does not match!'; exit 1)

PROJECT = qubes-yubico-pam

release:
	@if test -z "$(KEYID)"; then \
		echo "Try this instead:"; \
		echo "  make release KEYID=[PGPKEYID]"; \
		echo "For example:"; \
		echo "  make release KEYID=2117364A"; \
		exit 1; \
	fi
	@head -3 $(srcdir)/NEWS | grep -q "Version $(VERSION) .released `date -I`" || \
		(echo 'error: Update date/version in $(srcdir)/NEWS.'; exit 1)
	@if test ! -d "$(QUBES_YUBICO_GITHUB_REPO)"; then \
		echo "qubes-yubico.github.com repo not found!"; \
		echo "Make sure that QUBES_YUBICO_GITHUB_REPO is set"; \
		exit 1; \
		fi
	rm -f ChangeLog
	make check-doc-dist ChangeLog distcheck
	gpg --detach-sign --default-key $(KEYID) $(PACKAGE)-$(VERSION).tar.gz
	gpg --verify $(PACKAGE)-$(VERSION).tar.gz.sig
	cd $(srcdir) && git push
	cd $(srcdir) && git tag -u $(KEYID)! -m $(VERSION) $(VERSION)
	cd $(srcdir) && git push --tags
	$(QUBES_YUBICO_GITHUB_REPO)/publish $(PROJECT) $(VERSION) $(PACKAGE)-$(VERSION).tar.gz*
