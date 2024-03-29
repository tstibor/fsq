# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.69])

# Obtain GIT tag version.
m4_define([GIT_VERSION_STRING], m4_esyscmd([git describe --tags | sed 's/\(.*\)-.*/\1/' | tr -d '\n']))

# Define package information.
AC_INIT([fsq], [GIT_VERSION_STRING], [t.stibor@gsi.de], [], [https://github.com/tstibor/fsq])

m4_include([m4/ax_pthread.m4])
AM_INIT_AUTOMAKE

AC_CONFIG_SRCDIR([src/fsqd.c])
AC_CONFIG_HEADERS([config.h])

# Checks for programs.
AC_PROG_AWK
AC_PROG_CC
AC_PROG_CC_STDC
AC_PROG_CPP
AC_PROG_INSTALL
AC_PROG_LN_S
AC_PROG_MAKE_SET
AC_GNU_SOURCE

# Use the C language and compiler for the following checks.
AC_LANG([C])

LT_INIT
AC_CONFIG_MACRO_DIR([m4])

#  Default-disabled fsqd.
AC_ARG_ENABLE([fsqd],
              AS_HELP_STRING([--enable-fsqd], [enable and build fsqd]))
AM_CONDITIONAL([BUILD_FSQD], [test "x$enable_fsqd" = "xyes"])

build_fsqd="no"
# Fsqd build is enabled, make sure all requred header files and libraries are provided.
if test "x$enable_fsqd" = "xyes"; then
  # Path to Lustre header files directory, needed to compile fsqd.
  AC_ARG_WITH([lustre-headers],
              AS_HELP_STRING([--with-lustre-headers[=PATH]],[path to location of Lustre header files [default=/usr/include/lustre]]),
   	      LUSTRE_SRC_DIR="$withval", LUSTRE_SRC_DIR="/usr/include/lustre")

  lapiheader="no"
  AC_CHECK_FILE("$LUSTRE_SRC_DIR/lustreapi.h", [lapiheader="yes"],
	        [AC_MSG_ERROR([cannot find Lustre header files, use --with-lustre-headers=PATH])])

  AS_IF([test "x$lapiheader" = "xyes"],
   	[
       	  # We found the required Lustre source files, so let's check for Lustre API library.
	  AC_CHECK_LIB([lustreapi],
		       [llapi_error],
		       [],
		       [AC_MSG_ERROR([cannot find Lustre API library lustreapi required for fsqd, provide library path e.g. ./configure LDFLAGS='-L/<PATH_TO_LIB> -Wl,-rpath,<PATH_TO_LIB>'])]
                      )
        ]
       )

  # Path to TSM src directory.
  AC_ARG_WITH([tsm-headers], AS_HELP_STRING([--with-tsm-headers[=PATH]],[path to location of IBM TSM header files [default=/opt/tivoli/tsm/client/api/bin64/sample]]),
	      TSM_SRC_DIR="$withval", TSM_SRC_DIR="/opt/tivoli/tsm/client/api/bin64/sample")

  tsmheader="no"
  AC_CHECK_FILE("$TSM_SRC_DIR/dsmapitd.h", [tsmheader="yes"],
		[AC_MSG_ERROR([cannot find proprietary IBM TSM header files, use --with-tsm-headers=PATH])])
  AS_IF([test "x$tsmheader" = "xyes"],
	[
	  # We found the required TSM header files, so let's check for TSM API library.
	  AC_CHECK_LIB([ApiTSM64],
                       [dsmInitEx],
		       [],
		       [AC_MSG_ERROR([cannot find proprietary IBM library ApiTSM64 required for , provide library path e.g. ./configure LDFLAGS='-L/<PATH_TO_LIB> -Wl,-rpath,<PATH_TO_LIB>'])]
                      )
	]
       )

  # Path to LTSM header file directory.
  AC_ARG_WITH([ltsm-headers], AS_HELP_STRING([--with-ltsm-headers[=PATH]],[path to location of ltsmapi header files [default=/usr/include]]),
	      LTSM_SRC_DIR="$withval", LTSM_SRC_DIR="/usr/include/ltsm")

  ltsmheader="no"
  AC_CHECK_FILE("$LTSM_SRC_DIR/ltsmapi.h", [ltsmheader="yes"],
		[AC_MSG_ERROR([cannot find ltsmapi header files, use --with-ltsm-headers=PATH])])
  AS_IF([test "x$ltsmheader" = "xyes"],
	[
	  # We found the required LTSM header file, so let's check for LTSM API library.
	  AC_CHECK_LIB([ltsmapi],
                       [tsm_init],
		       [],
		       [AC_MSG_ERROR([cannot find library ltsmapi, provide library path e.g. ./configure LDFLAGS='-L/<PATH_TO_LIB> -Wl,-rpath,<PATH_TO_LIB>'])]
                      )
	]
       )

  # Check for required zlib library.
  AC_CHECK_LIB([z], [crc32], [], [AC_MSG_ERROR([cannot find library zlib required for fsqd, install zlib package or provide library path e.g. ./configure LDFLAGS='-L/<PATH_TO_LIB> -Wl,-rpath,<PATH_TO_LIB>'])])

  # Check for required xattr library, otherwise we cannot get Lustre stripe information and FSQ library.
  AC_CHECK_LIB([attr], [fgetxattr], [], [AC_MSG_ERROR([cannot find library libattr required for fsqd, install libattr package or provide library path e.g. ./configure LDFLAGS='-L/<PATH_TO_LIB> -Wl,-rpath,<PATH_TO_LIB>'])])

  # Check for required uuid library.
  AC_CHECK_LIB([uuid], [uuid_clear], [], [AC_MSG_ERROR([cannot find library uuid required for fsqd, install uuid package or provide library path e.g. ./configure LDFLAGS='-L/<PATH_TO_LIB> -Wl,-rpath,<PATH_TO_LIB>'])])


  build_fsqd="yes"
  AC_DEFINE([BUILD_FSQD], [1], [define if enabling fsqd build])
  AM_CONDITIONAL([BUILD_FSQD], [true])
fi

# Checks for other libraries.
AC_CHECK_LIB([m],[ceil])

# Checks for header files.
AC_CHECK_HEADERS([fcntl.h stdint.h stdlib.h string.h strings.h sys/param.h sys/time.h unistd.h pthread.h float.h attr/xattr.h arpa/inet.h limits.h netdb.h netinet/in.h sys/socket.h uuid/uuid.h])

AX_PTHREAD([AC_DEFINE(HAVE_PTHREAD, 1, [Define if you have POSIX threads libraries and header files.])
            CLIBS="$PTHREAD_LIBS $LIBS"
            CFLAGS="$CFLAGS $PTHREAD_CFLAGS"
            LDFLAGS="$LDFLAGS $PTHREAD_CFLAGS"
            CC="$PTHREAD_CC"],[])
AC_SEARCH_LIBS(pthread_create, pthread)

#  Default-disabled tests.
AC_ARG_ENABLE([tests],
              AS_HELP_STRING([--enable-tests], [enable and build tests]))
AM_CONDITIONAL([BUILD_TESTS], [test "x$enable_tests" = "xyes"])

# Tests can be build only when building fsqd.
AS_IF([test "x$enable_tests" = "xyes" && test "x$build_fsqd" = "xno"],
      [AC_MSG_ERROR([cannot build tests without fsqd, use --enable-fsqd and --enable-tests together])])

# Enable measurements when tests are enabled.
if test "x$enable_tests" = "xyes"; then
    AC_DEFINE([WITH_MEASUREMENT], [1], [define if enabling tests])
else
    enable_tests="no"
fi

# Default-disabled benchmark.
AC_ARG_ENABLE([benchmark],
              AS_HELP_STRING([--enable-benchmark], [enable and build benchmark]))
AM_CONDITIONAL([BUILD_BENCHMARK], [test "x$enable_benchmark" = "xyes"])

# Enable measurements when benchmark is enabled.
if test "x$enable_benchmark" = "xyes"; then
     # Check for required zlib library.
       AC_CHECK_LIB([z], [crc32], [], [AC_MSG_ERROR([cannot find library zlib required for fsqbench, install zlib package or provide library path e.g. ./configure LDFLAGS='-L/<PATH_TO_LIB> -Wl,-rpath,<PATH_TO_LIB>'])])

    AC_DEFINE([WITH_MEASUREMENT], [1], [define if enabling benchmark])
else
    enable_benchmark="no"
fi

# Propage flags and dirs among final Makefiles.
AC_SUBST([AM_CFLAGS])
AC_SUBST([LUSTRE_SRC_DIR])
AC_SUBST([LTSM_SRC_DIR])
AC_SUBST([TSM_SRC_DIR])

# Checks for typedefs, structures, and compiler characteristics.
AC_CHECK_HEADER_STDBOOL
AC_TYPE_MODE_T
AC_TYPE_OFF_T
AC_TYPE_SIZE_T
AC_TYPE_SSIZE_T
AC_TYPE_UINT16_T
AC_TYPE_UINT32_T
AC_TYPE_UINT64_T
AC_TYPE_UINT8_T
AC_TYPE_UID_T

# Checks for library functions.
AC_FUNC_LSTAT_FOLLOWS_SLASHED_SYMLINK AC_FUNC_CHOWN
AC_CHECK_FUNCS([malloc realloc atexit bzero clock_gettime gettimeofday memmove memset mkdir realpath sqrt strerror strtol gethostbyname inet_ntoa socket])

AC_CONFIG_FILES([Makefile
                 src/Makefile
                 src/lib/Makefile
		 src/test/Makefile
		 src/benchmark/Makefile
		 man/Makefile])

# Remove unneeded libraries.
LDFLAGS="$LDFLAGS -Wl,--as-needed"

# Halt on warnings.
CFLAGS="$CFLAGS -Wall -Werror -fPIC"

# Allow truncate writing when compiling with gcc, though not needed for clang.
if test "$CC" = "gcc" ; then
   CFLAGS="$CFLAGS -Werror=format-truncation=0"
fi

# For building packages we extract from tag X.Y.Z-R, version: X.Y.Z and release: R
AC_SUBST(_VERSION, [m4_esyscmd([git describe --tags | cut -f1 -d"-"])])
AC_SUBST(_RELEASE, [m4_esyscmd([git describe --tags | sed 's/-g[a-z0-9]\{7\}//' | sed -n -e '/^.*-/!{q100}; s/^.*-//p' || echo '1'])])

AC_OUTPUT

echo
echo "configured fsq:"
echo
echo "build fsqc and fsqapi library : yes"
echo "build fsqd                    : ${build_fsqd}"
echo "build test suite              : ${enable_tests}"
echo "build benchmark               : ${enable_benchmark}"
echo
