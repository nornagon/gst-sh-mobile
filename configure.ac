AC_PREREQ(2.52)

dnl initialize autoconf
dnl when going to/from release please set the nano (fourth number) right !
dnl releases only do Wall, git and prerelease does Werror too
AC_INIT(GStreamer SHVIDEO, 0.0.0.1,
    http://bugzilla.gnome.org/enter_bug.cgi?product=GStreamer,
    gst-shvideo)

AG_GST_INIT

dnl initialize automake
AM_INIT_AUTOMAKE

dnl define PACKAGE_VERSION_* variables
AS_VERSION

dnl check if this is a release version
AS_NANO(GST_CVS="no", GST_CVS="yes")

dnl can autoconf find the source ?
AC_CONFIG_SRCDIR([gst/shvideo/gstshvideodec.c])

dnl define the output header for config
AM_CONFIG_HEADER([config.h])

dnl can autoconf find the macros
AC_CONFIG_MACRO_DIR([common/m4])

dnl AM_MAINTAINER_MODE only provides the option to configure to enable it
AM_MAINTAINER_MODE

dnl sets host_* variables
AC_CANONICAL_HOST

dnl our libraries and install dirs use major.minor as a version
GST_MAJORMINOR=$PACKAGE_VERSION_MAJOR.$PACKAGE_VERSION_MINOR
dnl we override it here if we need to for the release candidate of new series
GST_MAJORMINOR=0.10
AC_SUBST(GST_MAJORMINOR)

AM_PROG_LIBTOOL

dnl *** required versions of GStreamer stuff ***
GST_REQ=0.10.22

dnl *** autotools stuff ****

dnl allow for different autotools
AS_AUTOTOOLS_ALTERNATE

dnl Add parameters for aclocal
AC_SUBST(ACLOCAL_AMFLAGS, "-I m4 -I common/m4")

dnl *** check for arguments to configure ***

AG_GST_ARG_WITH_PKG_CONFIG_PATH
AG_GST_ARG_WITH_PACKAGE_NAME
AG_GST_ARG_WITH_PACKAGE_ORIGIN
AG_GST_ARG_WITH_PLUGINS

dnl *** checks for platform ***

dnl * hardware/architecture *

dnl common/m4/gst-arch.m4
dnl check CPU type
AG_GST_ARCH

dnl *** checks for programs ***

dnl find a compiler
AC_PROG_CC

# FIXME: are these two lines needed ?
AM_PROG_CC_STDC
AS="${CC}"

dnl check if the compiler supports '-c' and '-o' options
AM_PROG_CC_C_O

# Check for pkg-config
AC_CHECK_PROG(HAVE_PKG_CONFIG, pkg-config, yes)

AC_PATH_PROG(VALGRIND_PATH, valgrind, no)
AM_CONDITIONAL(HAVE_VALGRIND, test ! "x$VALGRIND_PATH" = "xno")

dnl check for documentation tools

dnl *** checks for libraries ***

AC_CHECK_LIB(glib-2.0, strcmp)

dnl *** checks for header files ***

dnl check if we have ANSI C header files
AC_HEADER_STDC

dnl *** checks for types/defines ***

dnl *** checks for structures ***

dnl *** checks for compiler characteristics ***

dnl *** checks for library functions ***

dnl *** checks for dependancy libraries ***

AG_GST_CHECK_FUNCTION

AG_GST_GLIB_CHECK([2.12])

dnl checks for gstreamer
dnl uninstalled is selected preferentially -- see pkg-config(1)
AG_GST_CHECK_GST($GST_MAJORMINOR, [$GST_REQ], yes)
AG_GST_CHECK_GST_BASE($GST_MAJORMINOR, [$GST_REQ], yes)
AG_GST_CHECK_GST_GDP($GST_MAJORMINOR, [$GST_REQ], yes)
AG_GST_CHECK_GST_CONTROLLER($GST_MAJORMINOR, [$GST_REQ], yes)
AG_GST_CHECK_GST_PLUGINS_BASE($GST_MAJORMINOR, [$GST_REQ], yes)
dnl AG_GST_CHECK_GST_CHECK($GST_MAJORMINOR, [$GST_REQ], no)
dnl AM_CONDITIONAL(HAVE_GST_CHECK, test "x$HAVE_GST_CHECK" = "xyes")

AC_MSG_NOTICE(Using GStreamer Core Plugins in $GST_PLUGINS_DIR)

dnl
dnl Check for libuiomux
dnl

HAVE_UIOMUX=no
UIOMUX_SUPPORT=no

if test "x$HAVE_PKG_CONFIG" = "xyes" ; then
  PKG_CHECK_MODULES(UIOMUX, uiomux, HAVE_UIOMUX=yes, HAVE_UIOMUX=no)
fi

if test "x$HAVE_UIOMUX" = "xno" ; then
  AC_CHECK_LIB(uiomux, uiomux_init, HAVE_UIOMUX="maybe", , [-l...])
  if test "x$HAVE_UIOMUX" = xmaybe ; then
    AC_CHECK_HEADER([uiomux/uiomux.h], HAVE_UIOMUX="yes", HAVE_UIOMUX="no")
  fi
  if test "x$HAVE_UIOMUX" = xyes ; then
    UIOMUX_LIBS="-luiomux"
  fi
fi

if test "x$HAVE_UIOMUX" != "xyes" ; then
  AC_MSG_ERROR([libuiomux is required])
  AC_ERROR
fi

dnl
dnl Check for libshveu
dnl

PKG_CHECK_MODULES(SHVEU, shveu >= 0.5.0, HAVE_SHVEU="yes", HAVE_SHVEU="no")
if test "x$HAVE_SHVEU" != "xyes" ; then
  AC_MSG_ERROR([libshveu is required])
  AC_ERROR
fi
AC_SUBST(SHVEU_CFLAGS)
AC_SUBST(SHVEU_LIBS)

dnl
dnl Check for libshcodecs
dnl

AG_GST_CHECK_FEATURE(SHCODECS, [shcodecs], shvideo, [

	AG_GST_PKG_CHECK_MODULES(SHCODECS, shcodecs >= 0.9.5)
	if test "x$HAVE_SHCODECS" = "xyes"; then
		AC_CHECK_LIB(shcodecs,shcodecs_decoder_set_use_physical,,AC_ERROR,[$SHCODECS_LIBS])
		AC_CHECK_LIB(shcodecs,shcodecs_encoder_get_frame_num_delta,,AC_ERROR,[$SHCODECS_LIBS])
	else
		AC_MSG_WARN([libshcodecs not available, cannot build SHCODECS plugin])
	fi
])
AC_SUBST(SHCODECS_CFLAGS)
AC_SUBST(SHCODECS_LIBS)

dnl *** set variables based on configure arguments ***

dnl set location of plugin directory
AG_GST_SET_PLUGINDIR

dnl define an ERROR_CFLAGS Makefile variable
AG_GST_SET_ERROR_CFLAGS($GST_CVS)

dnl define correct level for debugging messages
AG_GST_SET_LEVEL_DEFAULT($GST_CVS)

dnl *** finalize CFLAGS, LDFLAGS, LIBS ***

AG_GST_DEFAULT_ELEMENTS

dnl Overview:
dnl GST_OPTION_CFLAGS:  common flags for profiling, debugging, errors, ...
dnl GST_*:              flags shared by built objects to link against GStreamer
dnl GST_ALL_LDFLAGS:    linker flags shared by all
dnl GST_LIB_LDFLAGS:    additional linker flags for all libaries
dnl GST_LT_LDFLAGS:     library versioning of our libraries
dnl GST_PLUGIN_LDFLAGS: flags to be used for all plugins

dnl GST_OPTION_CFLAGS
if test "x$USE_DEBUG" = xyes; then
   PROFILE_CFLAGS="-g"
fi
AC_SUBST(PROFILE_CFLAGS)

if test "x$GST_CVS" = "xyes"; then
  DEPRECATED_CFLAGS="-DGST_DISABLE_DEPRECATED"
else
  DEPRECATED_CFLAGS=""
fi
AC_SUBST(DEPRECATED_CFLAGS)

dnl every flag in GST_OPTION_CFLAGS can be overridden at make time
GST_OPTION_CFLAGS="\$(ERROR_CFLAGS) \$(DEBUG_CFLAGS) \$(PROFILE_CFLAGS) \$(GCOV_CFLAGS) \$(OPT_CFLAGS) \$(DEPRECATED_CFLAGS)"

AC_SUBST(GST_OPTION_CFLAGS)

dnl FIXME: do we want to rename to GST_ALL_* ?
dnl prefer internal headers to already installed ones
dnl also add builddir include for enumtypes and marshal
dnl add GST_OPTION_CFLAGS, but overridable
GST_CFLAGS="-I\$(top_srcdir)/gst-libs -I\$(top_builddir)/gst-libs $GST_CFLAGS \$(GST_OPTION_CFLAGS)"
AC_SUBST(GST_CFLAGS)
AC_SUBST(GST_LIBS)

GST_ALL_LDFLAGS="-no-undefined"
AC_SUBST(GST_ALL_LDFLAGS)

dnl this really should only contain flags, not libs - they get added before
dnl whatevertarget_LIBS and -L flags here affect the rest of the linking
GST_PLUGIN_LDFLAGS="-module -avoid-version -export-symbols-regex '^[_]*gst_plugin_desc\$\$' $GST_ALL_LDFLAGS"
AC_SUBST(GST_PLUGIN_LDFLAGS)

AC_CONFIG_FILES(
Makefile
gst/Makefile
gst/shvideo/Makefile
)
AC_OUTPUT

AG_GST_OUTPUT_PLUGINS
