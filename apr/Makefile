APR_TOOLCHAIN_PREFIX = $(TOOLCHAIN_INSTALL_DIR)/usr/local

Project               = apr
UserType              = Administrator
ToolType              = Commands
GnuAfterInstall       = install-plist post-install munge-build-files
Extra_CC_Flags        = -Wno-error=implicit-function-declaration
Extra_Configure_Flags = --bindir=$(APR_TOOLCHAIN_PREFIX)/bin \
                        --with-installbuilddir=$(APR_TOOLCHAIN_PREFIX)/share/apr-1/build-1 \
                        --enable-dso=dlfcn
Extra_Environment     = apr_lock_method="USE_FLOCK_SERIALIZE" \
                        ac_cv_func_kqueue="yes" \
                        ac_cv_func_poll="yes"

include $(MAKEFILEPATH)/CoreOS/ReleaseControl/GNUSource.make

Install_Flags  = DESTDIR=$(DSTROOT)
Install_Target = install

post-install:
	$(CP) $(DSTROOT)/usr/lib/libapr-1.0.dylib $(SYMROOT)
	$(STRIP) -x $(DSTROOT)/usr/lib/libapr-1.0.dylib
	$(RM) $(DSTROOT)/usr/lib/apr.exp
	$(RM) $(DSTROOT)/usr/lib/libapr-1.la
	$(RM) $(DSTROOT)/usr/lib/libapr-1.a

munge-build-files:
	ed - $(DSTROOT)$(APR_TOOLCHAIN_PREFIX)/share/apr-1/build-1/apr_rules.mk < $(SRCROOT)/files/fix-apr_rules.mk.ed
	ed - $(DSTROOT)$(APR_TOOLCHAIN_PREFIX)/share/apr-1/build-1/libtool < $(SRCROOT)/files/fix-libtool.ed

# Automatic Extract & Patch
AEP_Project    = apr
AEP_Version    = 1.5.2
AEP_ProjVers   = $(AEP_Project)-$(AEP_Version)
AEP_Filename   = $(AEP_ProjVers).tar.bz2
AEP_ExtractDir = $(AEP_ProjVers)
AEP_Patches    = configure.diff \
                 PR4905364.diff \
                 legacy-32-bit-inode.diff \
                 apr_proc_spawn.diff \
                 remove-darwin10-hacks.diff \
                 apr-config.in.diff \
                 CVE-2017-12613.diff \
                 103583402.diff

# Extract the source.
install_source::
	$(TAR) -C $(SRCROOT) -jxf $(SRCROOT)/$(AEP_Filename)
	$(RMDIR) $(SRCROOT)/$(Project)
	$(MV) $(SRCROOT)/$(AEP_ExtractDir) $(SRCROOT)/$(Project)
	for patchfile in $(AEP_Patches); do \
		(cd $(SRCROOT)/$(Project) && patch -p0 -F0 < $(SRCROOT)/files/$$patchfile) || exit 1; \
	done

ConfigStamp2 = $(ConfigStamp)2

configure:: $(ConfigStamp2)

$(ConfigStamp2): $(ConfigStamp)
	$(_v) ed - $(OBJROOT)/include/apr.h < $(SRCROOT)/files/fix-apr.h.ed
	$(_v) ed - $(OBJROOT)/include/arch/unix/apr_private.h < $(SRCROOT)/files/fix-apr_private.h.ed

OSL = $(DSTROOT)/usr/local/OpenSourceLicenses

install-plist:
	$(MKDIR) $(OSL)
	$(INSTALL_FILE) $(Sources)/LICENSE $(OSL)/$(Project).txt
