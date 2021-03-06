AC_INIT([qbittorrent], [v4.2.0alpha], [bugs.qbittorrent.org], [], [https://www.qbittorrent.org/])
AC_CONFIG_AUX_DIR([build-aux])
AC_CONFIG_MACRO_DIR([m4])
AC_PROG_CC
AC_PROG_CXX
AC_PROG_SED
AC_LANG(C++)
AC_CANONICAL_HOST
AM_INIT_AUTOMAKE

# Define --wth-* and --enable-* arguments

AC_ARG_WITH(qtsingleapplication,
            [AS_HELP_STRING([--with-qtsingleapplication=@<:@system|shipped@:>@],
                            [Use the shipped qtsingleapplication library or the system one (default=shipped)])],
            [],
            [with_qtsingleapplication=shipped])

AC_ARG_ENABLE(debug,
              [AS_HELP_STRING([--enable-debug],
                              [Enable debug build])],
              [],
              [enable_debug=no])

AC_ARG_ENABLE(stacktrace,
              [AS_HELP_STRING([--enable-stacktrace],
                              [Enable stacktrace feature (default=auto)])],
              [],
              [enable_stacktrace=auto])

AC_ARG_ENABLE(gui,
              [AS_HELP_STRING([--disable-gui],
                              [Disable the GUI for headless running. Disables QtDBus and the GeoIP Database.])],
              [],
              [enable_gui=yes])

AC_ARG_ENABLE(systemd,
              [AS_HELP_STRING([--enable-systemd],
                              [Install the systemd service file (headless only).])],
              [],
              [enable_systemd=no])

AC_ARG_ENABLE(webui,
              [AS_HELP_STRING([--disable-webui],
                              [Disable the WebUI.])],
              [],
              [enable_webui=yes])

AC_ARG_ENABLE(qt-dbus,
              [AS_HELP_STRING([--disable-qt-dbus],
                              [Disable use of QtDBus (GUI only)])],
              [],
              [enable_qt_dbus=yes])

# Detect OS
AC_MSG_CHECKING([whether OS is FreeBSD])
AS_IF([expr "$host_os" : ".*freebsd.*" > /dev/null],
      [AC_MSG_RESULT([yes])
      LIBS="-lexecinfo $LIBS"],
      [AC_MSG_RESULT([no])])

AC_MSG_CHECKING([whether OS is macOS])
AS_IF([expr "$host_os" : ".*darwin.*" > /dev/null],
      [AC_MSG_RESULT([yes])
      enable_qt_dbus=no],
      [AC_MSG_RESULT([no])])

# Require 0.23 pkg-config
PKG_PROG_PKG_CONFIG([0.23])
AS_IF([test "x$PKG_CONFIG" = "x"],
      [AC_MSG_ERROR([Could not find pkg-config])])

# Check which arguments were set and act accordingly
AC_MSG_CHECKING([whether to enable the Debug build])
AS_CASE(["x$enable_debug"],
        ["xno"],
              [AC_MSG_RESULT([no])
              QBT_ADD_CONFIG="$QBT_ADD_CONFIG release"
              QBT_REMOVE_CONFIG="$QBT_REMOVE_CONFIG debug"],
        ["xyes"],
               [AC_MSG_RESULT([yes])
               QBT_ADD_CONFIG="$QBT_ADD_CONFIG debug"
               QBT_REMOVE_CONFIG="$QBT_REMOVE_CONFIG release"],
        [AC_MSG_RESULT([$enable_debug])
        AC_MSG_ERROR([Unknown option "$enable_debug". Use either "yes" or "no".])])

AC_MSG_CHECKING([whether to enable the stacktrace feature])
AS_CASE(["x$enable_stacktrace"],
        ["xno"],
              [AC_MSG_RESULT([no])
              QBT_REMOVE_CONFIG="$QBT_REMOVE_CONFIG stacktrace"],
        ["xyes"],
               [AC_MSG_RESULT([yes])
               QBT_ADD_CONFIG="$QBT_ADD_CONFIG stacktrace"],
        ["xauto"],
                [AC_COMPILE_IFELSE([AC_LANG_SOURCE([[#include <execinfo.h>]])],
                [AC_MSG_RESULT([yes])
                QBT_ADD_CONFIG="$QBT_ADD_CONFIG stacktrace"],
                [AC_MSG_RESULT([no])
                QBT_REMOVE_CONFIG="$QBT_REMOVE_CONFIG stacktrace"])],
        [AC_MSG_RESULT([$enable_stacktrace])
        AC_MSG_ERROR([Unknown option "$enable_stacktrace". Use either "yes" or "no".])])

AC_MSG_CHECKING([whether to enable the GUI])
AS_CASE(["x$enable_gui"],
        ["xyes"],
               [AC_MSG_RESULT([yes])
               enable_systemd=[no]
               QBT_REMOVE_CONFIG="$QBT_REMOVE_CONFIG nogui"],
        ["xno"],
              [AC_MSG_RESULT([no])
              enable_qt_dbus=[no]
              QBT_ADD_CONFIG="$QBT_ADD_CONFIG nogui"],
        [AC_MSG_RESULT([$enable_gui])
        AC_MSG_ERROR([Unknown option "$enable_gui". Use either "yes" or "no".])])

AC_MSG_CHECKING([whether to install the systemd service file])
AS_CASE(["x$enable_systemd"],
        ["xyes"],
               [AC_MSG_RESULT([yes])
               QBT_ADD_CONFIG="$QBT_ADD_CONFIG systemd"],
        ["xno"],
              [AC_MSG_RESULT([no])
              QBT_REMOVE_CONFIG="$QBT_REMOVE_CONFIG systemd"],
        [AC_MSG_RESULT([$enable_systemd])
        AC_MSG_ERROR([Unknown option "$enable_systemd". Use either "yes" or "no".])])

AC_MSG_CHECKING([whether to enable the WebUI])
AS_CASE(["x$enable_webui"],
        ["xyes"],
               [AC_MSG_RESULT([yes])
               QBT_REMOVE_CONFIG="$QBT_REMOVE_CONFIG nowebui"],
        ["xno"],
              [AC_MSG_RESULT([no])
              QBT_ADD_CONFIG="$QBT_ADD_CONFIG nowebui"],
        [AC_MSG_RESULT([$enable_webui])
        AC_MSG_ERROR([Unknown option "$enable_webui". Use either "yes" or "no".])])

FIND_QT5()
AS_IF([test "x$QT_QMAKE" = "x"],
      [AC_MSG_ERROR([Could not find qmake])
      ])
AS_IF([test "x$enable_gui" = "xyes"],
      [PKG_CHECK_MODULES(Qt5Svg, [Qt5Svg >= 5.5.1])
      ])
AC_MSG_CHECKING([whether QtDBus should be enabled])
AS_CASE(["x$enable_qt_dbus"],
        ["xyes"],
               [AC_MSG_RESULT([yes])
               FIND_QTDBUS()
               AS_IF([test "x$HAVE_QTDBUS" = "xfalse"],
                     [AC_MSG_ERROR([Could not find QtDBus])],
                     [QBT_ADD_CONFIG="$QBT_ADD_CONFIG dbus"]
                    )],
        ["xno"],
              [AC_MSG_RESULT([no])
              QBT_REMOVE_CONFIG="$QBT_REMOVE_CONFIG dbus"],
        [AC_MSG_RESULT([$enable_qt_dbus])
        AC_MSG_ERROR([Unknown option "$enable_qt_dbus". Use either "yes" or "no".])])


AX_BOOST_BASE([1.35],
              [AC_MSG_NOTICE([Boost CXXFLAGS: "$BOOST_CPPFLAGS"])
               AC_MSG_NOTICE([Boost LDFLAGS: "$BOOST_LDFLAGS"])],
              [AC_MSG_ERROR([Could not find Boost])])
CXXFLAGS="$BOOST_CPPFLAGS $CXXFLAGS"
LDFLAGS="$BOOST_LDFLAGS $LDFLAGS"

# add workaround for problematic boost version
# taken from ax_boost_base.m4
m4_define([DETECT_BOOST_VERSION_PROGRAM],
    [AC_LANG_PROGRAM([[#include <boost/version.hpp>]],
                     [[(void) ((void)sizeof(char[1 - 2*!!((BOOST_VERSION) < ($1))]));]])])

AC_COMPILE_IFELSE([DETECT_BOOST_VERSION_PROGRAM(106000)], [],
    [QBT_ADD_DEFINES="$QBT_ADD_DEFINES BOOST_NO_CXX11_RVALUE_REFERENCES"])

AX_BOOST_SYSTEM()
AC_MSG_NOTICE([Boost.System LIB: "$BOOST_SYSTEM_LIB"])
LIBS="$BOOST_SYSTEM_LIB $LIBS"

AC_MSG_CHECKING([which qtsingleapplication to use])
AS_CASE(["x$with_qtsingleapplication"],
        ["xshipped"],
                   [AC_MSG_RESULT([shipped])
                   QBT_REMOVE_CONFIG="$QBT_REMOVE_CONFIG usesystemqtsingleapplication"],
        ["xsystem"],
                  [AC_MSG_RESULT([system])
                  QBT_ADD_CONFIG="$QBT_ADD_CONFIG usesystemqtsingleapplication"],
        [AC_MSG_RESULT([$with_qtsingleapplication])
        AC_MSG_ERROR([Unknown option "$with_qtsingleapplication". Use either "system" or "shipped".])])

PKG_CHECK_MODULES(libtorrent,
                  [libtorrent-rasterbar >= 1.0.6],
                  [CXXFLAGS="$libtorrent_CFLAGS $CXXFLAGS"
                  LIBS="$libtorrent_LIBS $LIBS"])

PKG_CHECK_MODULES(zlib,
                 [zlib >= 1.2.5.2],
                 [CXXFLAGS="$zlib_CFLAGS $CXXFLAGS"
                 LIBS="$zlib_LIBS $LIBS"])

# Check if already in >= C++11 mode because of the flags returned by one of the above packages
AC_MSG_CHECKING([if compiler is using C++11 or later mode])
AC_COMPILE_IFELSE([DETECT_CPP11_PROGRAM()],
                  [AC_MSG_RESULT([yes])
                   QBT_CXX11_FOUND="yes"],
                  [AC_MSG_RESULT([no])
                   QBT_CXX11_FOUND="no"])
                  
# In case of no, check if the compiler can support at least C++11
# and if yes, enable it leaving a warning to the user
AS_IF([test "x$QBT_CXX11_FOUND" = "xno"],
      [AC_MSG_CHECKING([if compiler supports C++11])
       TMP_CXXFLAGS="$CXXFLAGS"
       CXXFLAGS="$CXXFLAGS -std=c++11"
       AC_COMPILE_IFELSE([DETECT_CPP11_PROGRAM()],
                        [AC_MSG_RESULT([yes])
                         AC_MSG_CHECKING([if C++11 is disabled by the set compiler flags])
                         # prepend the flag so it won't override conflicting user defined flags
                         CXXFLAGS="-std=c++11 $TMP_CXXFLAGS"
                         AC_COMPILE_IFELSE([DETECT_CPP11_PROGRAM()],
                                           [AC_MSG_RESULT([no])
                                            CXXFLAGS="$TMP_CXXFLAGS -std=c++11"
                                            AC_MSG_WARN([C++11 mode is now force enabled.
Make sure you use the same C++ mode for qBittorrent and its dependencies.
To explicitly set qBittorrent to a later mode use CXXFLAGS.
Example: `CXXFLAGS="\$CXXFLAGS -std=c++14" ./configure`])],
                                           [AC_MSG_RESULT([yes])
                                            AC_MSG_ERROR([The compiler supports C++11 but the user or a dependency has explicitly enabled a lower mode.])])],
                        [AC_MSG_RESULT([no])
                         AC_MSG_ERROR([A compiler supporting C++11 is required.])])
      ])

# These are required because autoconf doesn't expand these **particular**
# vars automatically. And qmake cannot autoexpand them.
AX_DEFINE_DIR([EXPAND_PREFIX], [prefix])
AX_DEFINE_DIR([EXPAND_BINDIR], [bindir])
AX_DEFINE_DIR([EXPAND_DATADIR], [datadir])
AX_DEFINE_DIR([EXPAND_MANDIR], [mandir])

# Original extract() function contributed by pmzqla
# $*: Strings to parse
# Set $QBT_CONF_DEFINES, $QBT_CONF_INCLUDES, $QBT_CONF_EXTRA_CFLAGS
extract() {
  if [[ -z "$*" ]]; then
    echo "Input string required"
    return 1
  fi

  # BSD sed needs an actual newline character in the substitute command
  new_line='
'
  # Convert " -" to "\n" if not between quotes and remove possible leading white spaces
  string=$(echo " $*" | $SED -e "s: -:\\${new_line}:g" -e 's:"\(.*\)\n\(.*\)":\"\1 -\2":g' -e "s:'\(.*\)\n\(.*\)':\'\1 -\2':g" -e 's/^[[[:space:]]]*//')
  SAVEIFS=$IFS
  IFS=$(printf "\n\b")
  for i in $string; do
    case "$(echo "$i" | cut -c1)" in
      '') ;;
      D) QBT_CONF_DEFINES="$QBT_CONF_DEFINES $(echo $i | cut -c2-)";;
      I) QBT_CONF_INCLUDES="$QBT_CONF_INCLUDES $(echo $i | cut -c2-)";;
      *) QBT_CONF_EXTRA_CFLAGS="$QBT_CONF_EXTRA_CFLAGS -$i";;
    esac
  done
  IFS=$SAVEIFS
}

extract "$CFLAGS $CXXFLAGS"
QBT_ADD_DEFINES="$QBT_ADD_DEFINES $QBT_CONF_DEFINES"

# Substitute the values of these vars in conf.pri.in
AC_SUBST(QBT_CONF_INCLUDES)
AC_SUBST(QBT_CONF_EXTRA_CFLAGS)
AC_SUBST(QBT_ADD_CONFIG)
AC_SUBST(QBT_REMOVE_CONFIG)
AC_SUBST(QBT_ADD_DEFINES)
AC_SUBST(QBT_REMOVE_DEFINES)

AC_OUTPUT(conf.pri)
AS_IF([test "x$enable_systemd" = "xyes"],
      [AC_OUTPUT(dist/unix/systemd/qbittorrent-nox@.service)])



AC_MSG_NOTICE([Running qmake to generate the makefile...])
CONFDIR="$( cd "$( dirname "$0" )" && pwd )"

$QT_QMAKE -r [$CONFDIR]/qbittorrent.pro "QMAKE_LRELEASE=$QMAKE_LRELEASE"

ret="$?"

AS_ECHO()
AS_IF([test "x$ret" = "x0"],
      [AC_MSG_NOTICE([Good, your configure finished.])],
      [AC_MSG_ERROR([Failed running $QT_QMAKE to generate the makefile])])
AS_ECHO()
