dnl Process this -*-m4-*- file with autoconf to produce a configure script.

dnl Disable caching
define([AC_CACHE_LOAD],)
define([AC_CACHE_SAVE],)

AC_INIT
AC_CONFIG_SRCDIR([base/nagios.c])
# config.h.in is manually managed so we don't want autoheader to overwrite it when running autoreconf
AC_CONFIG_HEADERS(include/ignored_config.h include/config.h lib/snprintf.h lib/iobroker.h)
AC_PREFIX_DEFAULT(/usr/local/nagios)

PKG_NAME=nagios

PKG_VERSION="4.5.9"
PKG_HOME_URL="https://www.nagios.org/"
PKG_REL_DATE="2024-12-19"

dnl Figure out how to invoke "install" and what install options to use.
AC_PROG_INSTALL
AC_SUBST(INSTALL)

dnl What OS are we running?
AC_CANONICAL_HOST

AC_CONFIG_FILES([html/index.php])

dnl Checks for programs.
AC_PROG_CC
AC_PROG_MAKE_SET
AC_PROG_LN_S
AC_PATH_PROG([STRIP],[strip],[true])

dnl Checks for header files.
AC_HEADER_SYS_WAIT
AC_CHECK_HEADERS(arpa/inet.h ctype.h dirent.h errno.h fcntl.h getopt.h grp.h)
AC_CHECK_HEADERS(libgen.h limits.h math.h netdb.h netinet/in.h pwd.h regex.h)
AC_CHECK_HEADERS(signal.h socket.h stdarg.h string.h strings.h sys/loadavg.h)
AC_CHECK_HEADERS(sys/mman.h sys/types.h sys/time.h sys/resource.h sys/wait.h)
AC_CHECK_HEADERS(sys/socket.h sys/stat.h sys/timeb.h sys/un.h sys/ipc.h)
AC_CHECK_HEADERS(sys/msg.h sys/poll.h syslog.h uio.h unistd.h locale.h wchar.h)
AC_CHECK_HEADERS(sys/prctl.h)

dnl Checks for typedefs, structures, and compiler characteristics.
AC_C_CONST
AC_STRUCT_TM
AC_STRUCT_TIMEZONE
AC_TYPE_MODE_T
AC_TYPE_PID_T
AC_TYPE_SIZE_T

AC_TYPE_GETGROUPS

dnl Get Nagios autoconf-macros
AX_NAGIOS_GET_OS
AX_NAGIOS_GET_DISTRIB_TYPE
AX_NAGIOS_GET_INIT
AX_NAGIOS_GET_INETD
AX_NAGIOS_GET_PATHS
AX_NAGIOS_GET_FILES

dnl We only have these options available so we need to make them work until there are more
if test "x$src_init" != "xdefault-init" -a \
        "x$src_init" != "xdefault-service" -a \
        "x$src_init" != "xopenrc-init" -a \
        "x$src_init" != "xnewbsd-init" -a \
        "x$src_init" != "xopenbsd-init" -a \
        "x$src_init" != "xupstart-init"; then

	src_init="default-init"
	init_type="sysv"
	initname="nagios"

	if test "x$initdiroverridden" != "xyes"; then
		initdir="/etc/init.d"
	fi
fi

dnl We need to check for a lock file specified
dnl And if the built in check didn't work (the N/A)
dnl then we need to find one!
if test "x$subsyslockdir" = "xN/A"; then
	if test -d "/var/run"; then
		subsyslockdir="/var/run"
		subsyslockfile="/var/run/$INIT_PROG.lock"
	else
		subsyslockdir="/usr/local/nagios/var/"
		subsyslockfile="/usr/local/nagios/var/$INIT_PROG.lock"
	fi
fi
AC_ARG_WITH(lockfile,
	AS_HELP_STRING([--with-lockfile=<path>],
		[sets path for lockfile]),
	subsyslockfile=$withval,
	subsyslockfile=$subsyslockfile
)


dnl Check for asprintf() and friends...
AC_CACHE_CHECK([for va_copy],ac_cv_HAVE_VA_COPY,[
AC_LINK_IFELSE(
	[AC_LANG_PROGRAM([[#include <stdarg.h>
		va_list ap1,ap2;]],
		[[va_copy(ap1,ap2);]])],
	ac_cv_HAVE_VA_COPY=yes,
	ac_cv_HAVE_VA_COPY=no)])
if test "x$ac_cv_HAVE_VA_COPY" = "xyes"; then
    AC_DEFINE(HAVE_VA_COPY,1,[Whether va_copy() is available])
else    
    AC_CACHE_CHECK([for __va_copy],ac_cv_HAVE___VA_COPY,[
	AC_LINK_IFELSE(
		[AC_LANG_PROGRAM([[#include <stdarg.h>
				va_list ap1,ap2;]],
			[[__va_copy(ap1,ap2);]])],
		ac_cv_HAVE___VA_COPY=yes,
		ac_cv_HAVE___VA_COPY=no)])
    if test "x$ac_cv_HAVE___VA_COPY" = "xyes"; then
        AC_DEFINE(HAVE___VA_COPY,1,[Whether __va_copy() is available])
    fi
fi

AC_CHECK_FUNC(vsnprintf,,SNPRINTF_O=../lib/snprintf.o)
AC_CHECK_FUNC(snprintf,,SNPRINTF_O=../lib/snprintf.o)
AC_CHECK_FUNC(asprintf,,SNPRINTF_O=../lib/snprintf.o)
AC_CHECK_FUNC(vasprintf,,SNPRINTF_O=../lib/snprintf.o)
AC_CHECK_FUNCS(sigaction)

AC_CACHE_CHECK([for C99 vsnprintf],ac_cv_HAVE_C99_VSNPRINTF,[
AC_RUN_IFELSE(
	[AC_LANG_SOURCE([[
		#include <sys/types.h>
		#include <stdarg.h>
		#include <stdio.h>
		#include <stdlib.h>
		#include <string.h>
		void foo(const char *format, ...) {
			va_list ap;
			int len;
			char buf[5];

			va_start(ap, format);
			len = vsnprintf(buf, 0, format, ap);
			va_end(ap);
			if (len != 5) exit(1);

			va_start(ap, format);
			len = vsnprintf(0, 0, format, ap);
			va_end(ap);
			if (len != 5) exit(1);

			if (snprintf(buf, 3, "hello") != 5 || strcmp(buf, "he") != 0) exit(1);

			exit(0);
		}
		int main(void) { foo("hello"); }
	]])],
	ac_cv_HAVE_C99_VSNPRINTF=yes,ac_cv_HAVE_C99_VSNPRINTF=no,ac_cv_HAVE_C99_VSNPRINTF=cross)])
if test "x$ac_cv_HAVE_C99_VSNPRINTF" = "xyes"; then
    AC_DEFINE(HAVE_C99_VSNPRINTF,1,[Define if system has C99 compatible vsnprintf])
fi

dnl AC_CHECK_FUNC(snprintf,AC_DEFINE(HAVE_SNPRINTF),SNPRINTF_O=../common/snprintf.o)
AC_SUBST(SNPRINTF_O)


dnl Checks for library functions.
AC_SEARCH_LIBS([getservbyname],[nsl],
	[if test "x$ac_cv_search_getservbyname" != "xnone required"; then
		SOCKETLIBS="$SOCKETLIBS -lnsl"
	fi])
AC_SEARCH_LIBS([connect],[socket],
	[if test "x$ac_cv_search_connect" != "xnone required"; then
		SOCKETLIBS="$SOCKETLIBS -lsocket"
	fi])
AC_SUBST(SOCKETLIBS)
AC_CHECK_FUNCS(initgroups setenv strdup strstr strtoul unsetenv)

AC_MSG_CHECKING(for type of socket size)
AC_COMPILE_IFELSE(
	[AC_LANG_PROGRAM([[#include <stdlib.h>
		#include <sys/types.h>
		#include <sys/socket.h>]],
		[[int a = send(1, (const void *) 0, (size_t) 0, (int) 0);]])],
	[AC_DEFINE(SOCKET_SIZE_TYPE, size_t, [typedef for socket size]) AC_MSG_RESULT(size_t)],
	[AC_DEFINE(SOCKET_SIZE_TYPE, int, [typedef for socket size]) AC_MSG_RESULT(int)])


dnl Does user want to check for SSL?
AC_ARG_ENABLE([ssl],
	AS_HELP_STRING([--disable-ssl],[disables native SSL support @<:@default=check@:>@]),[
	if test x$enableval = xyes; then
		check_for_ssl=yes
	else
		check_for_ssl=no
	fi
	],check_for_ssl=yes)

need_dh=no

dnl Optional SSL library and include paths
if test x$check_for_ssl = xyes; then
	# need_dh should only be set for NRPE
	#need_dh=yes
	AX_NAGIOS_GET_SSL
fi

dnl Solaris needs rt or posix4 libraries for nanosleep()
AC_SEARCH_LIBS(nanosleep,[rt posix4],,[
	echo "Error: nanosleep() needed for timing operations."
	exit 1
	])

dnl TODO: host_os needs changed to reflect AC_NAGIOS
case $host_os in
*bsd*|darwin*)
	root_grp=wheel
	;;
*)
	root_grp=root
	;;
esac

AC_ARG_WITH(root_group,AS_HELP_STRING([--with-root-group=<grp>],[sets group name for installing init]),root_grp=$withval)
AC_SUBST(root_grp)
INIT_OPTS="-o root -g $root_grp"
AC_SUBST(INIT_OPTS)

dnl Just enable all of the testing things
dnl --enable-debugging --enable-coverage --with-this-user
dnl This is here because if not, the user override doesnt happen
ENABLE_TESTING=no
AC_ARG_ENABLE(testing,AS_HELP_STRING([--enable-testing],[DEBUGGING ONLY - Enable testing options]),
	[ENABLE_TESTING=$enableval])

not_a_user="^^^^^"
AC_ARG_WITH(this_user,AS_HELP_STRING([--with-this-user],[sets all user/group info for current user running ./configure]),this_user=$USER,this_user=$not_a_user)
if test "x$ENABLE_TESTING" = "xyes"; then
	this_user=$USER
fi

AC_ARG_WITH(nagios_user,AS_HELP_STRING([--with-nagios-user=<user>],[sets user name to run nagios]),nagios_user=$withval,nagios_user=nagios)
AC_ARG_WITH(nagios_group,AS_HELP_STRING([--with-nagios-group=<grp>],[sets group name to run nagios]),nagios_grp=$withval,nagios_grp=nagios)

if test "x$this_user" != "x$not_a_user"; then
	nagios_user=$this_user
	nagios_grp=$this_user
fi

AC_SUBST(nagios_user)
AC_SUBST(nagios_grp)
AC_DEFINE_UNQUOTED(DEFAULT_NAGIOS_USER,"$nagios_user",[user name to run nagios])
AC_DEFINE_UNQUOTED(DEFAULT_NAGIOS_GROUP,"$nagios_grp",[group name to run nagios])
INSTALL_OPTS="-o $nagios_user -g $nagios_grp"
AC_SUBST(INSTALL_OPTS)

AC_ARG_WITH(command_user,AS_HELP_STRING([--with-command-user=<user>],[sets user name for command access]),command_user=$withval,command_user=$nagios_user)
AC_ARG_WITH(command_group,AS_HELP_STRING([--with-command-group=<grp>],[sets group name for command access]),command_grp=$withval,command_grp=$nagios_grp)

if test "x$this_user" != "x$not_a_user"; then
	command_user=$this_user
	command_grp=$this_user
fi

AC_SUBST(command_user)
AC_SUBST(command_grp)
COMMAND_OPTS="-o $command_user -g $command_grp"
AC_SUBST(COMMAND_OPTS)

dnl Check for location of mail program
MAIL_PROG=no
AC_ARG_WITH(mail,
	AS_HELP_STRING([--with-mail=<path_to_mail>],
		[sets path to equivalent program to mail]),
	MAIL_PROG=$withval,
	MAIL_PROG=no
) 
if test "x$MAIL_PROG" = "xno"; then
	AC_PATH_PROG(MAIL_PROG,mail) 
fi 
dnl Fix for systems that don't (yet) have mail/mailx installed...
if test "x$MAIL_PROG" = "x"; then
	if which mail >/dev/null; then
		MAIL_PROG=`which mail`
	elif which sendmail >/dev/null; then
		MAIL_PROG=`which sendmail`
	else
		MAIL_PROG="/bin/mail"
	fi
fi
AC_SUBST(MAIL_PROG)

dnl Determine location of the rm binary
BIN_RM=`which rm`
if test $? -ne 0; then
	BIN_RM='/bin/rm'
fi
AC_SUBST(BIN_RM)

dnl Determine location of the kill binary
BIN_KILL=`which kill`
if test $? -ne 0; then
	BIN_KILL='/bin/kill'
fi
AC_SUBST(BIN_KILL)

dnl Check for location of Apache conf.d directory
HTTP_CONF=no
AC_ARG_WITH(httpd_conf,
	AS_HELP_STRING([--with-httpd-conf=<path_to_conf>],
		[sets path to Apache conf.d directory]),
	HTTPD_CONF=$withval,
	HTTPD_CONF=no
) 
LN_HTTPD_SITES_ENABLED=0
if test "x$HTTPD_CONF" = "xno"; then
	if test -d "/etc/httpd/conf.d"; then
		HTTPD_CONF="/etc/httpd/conf.d"
	elif test -d "/etc/apache2/conf.d"; then
		HTTPD_CONF="/etc/apache2/conf.d"
	elif test -d "/etc/apache2/sites-available"; then
		HTTPD_CONF="/etc/apache2/sites-available"
		if test -d "/etc/apache2/sites-enabled"; then
			LN_HTTPD_SITES_ENABLED=1
		fi
	elif test -d "/etc/apache2/sites-enabled"; then
		HTTPD_CONF="/etc/apache2/sites-enabled"
	elif test -d "/etc/apache/conf.d"; then
		HTTPD_CONF="/etc/apache/conf.d"
	else
		HTTPD_CONF="/etc/httpd/conf.d"
	fi
fi 
AC_SUBST(HTTPD_CONF)
AC_SUBST(LN_HTTPD_SITES_ENABLED)

dnl Check if RLIMIT_PROC exists
AC_COMPILE_IFELSE(
	[AC_LANG_PROGRAM([[#ifdef __STDC__
		#include <sys/time.h>
		#include <sys/resource.h>
		#endif]],
		[[struct rlimit rlim; int ilim = getrlimit(RLIMIT_NPROC, &rlim);]])],
	[AC_DEFINE(DETECT_RLIMIT_PROBLEM,1,[System has RLIMIT_PROC]) AC_MSG_RESULT([checking for RLIMIT_PROC... yes])],
	[AC_MSG_RESULT([checking for RLIMIT_PROC... no])])

dnl Location of check result path
CHECKRESULTDIR=no
AC_ARG_WITH(checkresult-dir,
	AS_HELP_STRING([--with-checkresult-dir=<path>],
		[sets path to check results spool directory]),
	CHECKRESULTDIR=$withval,
	CHECKRESULTDIR=no
) 
if test "x$CHECKRESULTDIR" = "xno"; then
	CHECKRESULTDIR="$localstatedir/spool/checkresults"
fi 
AC_SUBST(CHECKRESULTDIR)

dnl Location of check result path
dnl TODO: this should be moved to autoconf-macros
TMPDIR=no
AC_ARG_WITH(temp-dir,
	AS_HELP_STRING([--with-temp-dir=<path>],
		[sets path to temp directory]),
	TMPDIR=$withval,
	TMPDIR=no
) 
if test "x$TMPDIR" = "xno"; then
	TMPDIR="/tmp"
fi 
AC_SUBST(TMPDIR)


dnl Determine the library to be used by the iobroker

dnl epoll_*() is linux specific and was added to glibc 2.3.2, so we
dnl check for 2.4 and use epoll() if we're on that version or later.
dnl But we still need to set the variable GLIBC_NEWER_2_4 since
dnl it is used later in calculations perhaps by OSes not linux

GLIBC_NEWER_2_4=0

case $host_os in
linux*)
	AC_MSG_CHECKING(for glibc at least version 2.4)
	AC_PREPROC_IFELSE([AC_LANG_SOURCE([[
#include <stdio.h>
#if defined(__GLIBC__) && defined(__linux)
#include <features.h>
# if !__GLIBC_PREREQ(2, 4)
#  error GLIBC too old
# endif
#else
# error Not using GLIBC
#endif
		]])],[GLIBC_NEWER_2_4=1; AC_MSG_RESULT(yes)],[GLIBC_NEWER_2_4=0; AC_MSG_RESULT(no)
	])
	if test "$GLIBC_NEWER_2_4" -eq 1; then
		AC_CHECK_HEADER([sys/epoll.h])
	fi
	;;
esac
AC_CHECK_HEADER([sys/select.h])

AC_ARG_WITH(iobroker,
	AS_HELP_STRING([--with-iobroker=<method>],
		[specify the method to use with iobroker: epoll, poll, or select]),
	IOBROKER_METHOD=$withval,
	IOBROKER_METHOD="none"
)

case $IOBROKER_METHOD in
epoll*)
	if test "$GLIBC_NEWER_2_4" -eq 1 -a "x$ac_cv_header_sys_epoll_h" = "xyes"; then
		AC_DEFINE([IOBROKER_USES_EPOLL], [], [iobroker uses epoll method])
	else
		echo "\"epoll\" is not available as an iobroker method."
		echo "Please use one of the other options."
		exit 1
	fi
	;;
poll*)
	if test "x$ac_cv_header_sys_poll_h" = "xyes" -o "x$ac_cv_header_poll_h" = "xyes"; then
		AC_DEFINE([IOBROKER_USES_POLL], [], [iobroker uses poll method])
	else
		echo "\"poll\" is not available as an iobroker method."
		echo "Please use one of the other options."
		exit 1
	fi
	;;
select*)
	if test "x$ac_cv_header_sys_select_h" = "xyes"; then
		AC_DEFINE([IOBROKER_USES_SELECT], [], [iobroker uses select method])
	else
		echo "\"select\" is not available as an iobroker method."
		echo "Please use one of the other options."
		exit 1
	fi
	;;
none*)
	if test "$GLIBC_NEWER_2_4" -eq 1 -a "x$ac_cv_header_sys_epoll_h" = "xyes"; then
		AC_DEFINE([IOBROKER_USES_EPOLL])
		IOBROKER_METHOD="epoll"
	elif test "x$ac_cv_header_sys_poll_h" = "xyes" -o "x$ac_cv_header_poll_h" = "xyes"; then
		AC_DEFINE([IOBROKER_USES_POLL])
		IOBROKER_METHOD="poll"
	elif test "x$ac_cv_header_sys_select_h" = "xyes"; then
		AC_DEFINE([IOBROKER_USES_SELECT])
		IOBROKER_METHOD="select"
	else
		echo "There are no available options for iobroker polling"
		exit 1
	fi
	;;
*)
	echo "\"$IOBROKER_METHOD\" is not a valid method for --with-iobroker"
	exit 1
	;;
esac

dnl Optional GD library and include paths
AC_ARG_WITH(gd-lib,
	AS_HELP_STRING([--with-gd-lib=DIR],[sets location of the gd library]),
	[
		LDFLAGS="${LDFLAGS} -L${withval}"
		LD_RUN_PATH="${withval}${LD_RUN_PATH:+:}${LD_RUN_PATH}"
	]
)
AC_ARG_WITH(gd-inc,
	AS_HELP_STRING([--with-gd-inc=DIR],[sets location of the gd include files]),
	[CFLAGS="${CFLAGS} -I${withval}"]
)


TRYGD=yes

dnl 'NERD' is now disabled by default. enable it for functionality
NERD=no
NERD_O=""
AC_ARG_ENABLE(nerd,
	AS_HELP_STRING([--enable-nerd],
		[enables Nagios Event Radio Dispatch (NERD)]),
	NERD=yes
)
if test "x$NERD" = "xyes"; then
	AC_DEFINE(ENABLE_NERD,1,[defined if user enabled NERD (radio dispatcher)])
	NERD_O="nerd.o"
fi
AC_SUBST(NERD_O)

dnl 'corewindow' is now disabled. Allow it to be enabled if someone wants it
COREWINDOW=no
AC_ARG_ENABLE(corewindow,
	AS_HELP_STRING([--enable-corewindow],
		[enables the 'corewindow' URL argument]),
	COREWINDOW=yes
)
AC_SUBST(COREWINDOW)

dnl statusmap CGI enabled by default, unless users chooses not to use it
TRYSTATUSMAP=yes
AC_ARG_ENABLE(statusmap,
	AS_HELP_STRING([--disable-statusmap],
		[disables compilation of statusmap CGI]),
	TRYSTATUSMAP=nope
)


dnl statuswrl CGI enabled by default, unless users chooses not to use it
TRYSTATUSWRL=yes
AC_ARG_ENABLE(statuswrl,
	AS_HELP_STRING([--disable-statuswrl],
		[disables compilation of statuswrl (VRML) CGI]),
	TRYSTATUSWRL=nope
)

if test "x$TRYSTATUSWRL" = "xyes"; then
	AC_DEFINE_UNQUOTED(USE_STATUSWRL,,[statuswrl CGI enabled by default, unless users chooses not to use it])
	CGIEXTRAS="$CGIEXTRAS statuswrl.cgi"
fi


dnl JMD_CHECK_LIB_ORDER(LIBRARY, FUNCTION, ORDER [, ACTION-IF-FOUND [, ACTION-IF-NOT-FOUND
dnl              [, OTHER-LIBRARIES]]])
	AC_DEFUN([JMD_CHECK_LIB_ORDER],
	[AC_MSG_CHECKING([for $2 in -l$1 (order $3)])
	dnl Use a cache variable name containing both the library and function name,
	dnl because the test really is for library $1 defining function $2, not
	dnl just for library $1.  Separate tests with the same $1 and different $2s
	dnl may have different results.
	ac_lib_var=`echo $1['_']$2['_']$3 | sed 'y%./+-%__p_%'`
	AC_CACHE_VAL(ac_cv_lib_$ac_lib_var,
	[ac_save_LIBS="$LIBS"
	LIBS="-l$1 $6 $LIBS"
	AC_LINK_IFELSE([AC_LANG_PROGRAM(dnl
	ifelse($2, main, , dnl Avoid conflicting decl of main.
/* We use char because int might match the return type of a gcc2
    builtin and then its argument prototype would still apply.  */
char $2();
), [[$2()]])],[eval "ac_cv_lib_$ac_lib_var=yes"],[eval "ac_cv_lib_$ac_lib_var=no"])
	LIBS="$ac_save_LIBS"
	])dnl
	if eval "test \"`echo '$ac_cv_lib_'$ac_lib_var`\" = yes"; then
	  AC_MSG_RESULT(yes)
	  ifelse([$4], ,
	[changequote(, )dnl
	  ac_tr_lib=HAVE_LIB`echo $1 | sed -e 's/[^a-zA-Z0-9_]/_/g' \
	    -e 'y/abcdefghijklmnopqrstuvwxyz/ABCDEFGHIJKLMNOPQRSTUVWXYZ/'`
	changequote([, ])dnl
	  AC_DEFINE_UNQUOTED($ac_tr_lib)
	  LIBS="-l$1 $LIBS"
	], [$4])
	else
	  AC_MSG_RESULT(no)
	ifelse([$5], , , [$5
	])dnl
	fi
	])



dnl Should we try and detect the GD libs?
if test "x$TRYGD" = "xyes"; then

	dnl libiconv is required on some systems - tack it on if found
	AC_CHECK_LIB(iconv,main,ICONV=-liconv,)

	dnl See if the GD lib is available and supports PNG images...

	dnl GD > 1.8.3 requires the TrueType library to be present as well, so test for that first...
	JMD_CHECK_LIB_ORDER(gd,gdImagePng,1,[
			GDLIBFOUND=yes
			GDLIBS="-lgd -lttf -lpng -ljpeg -lz -lm"
			],:,[-lttf -lpng -ljpeg -lz -lm])
 
	dnl GD > 1.8.1 requires the jpeg library to be present as well, so test for that...
	if test "x$GDLIBFOUND" = "x"; then
		JMD_CHECK_LIB_ORDER(gd,gdImagePng,2,[
			GDLIBFOUND=yes
			GDLIBS="-lgd $ICONV -lpng -ljpeg -lz -lm"
        		],:,[$ICONV -lpng -ljpeg -lz -lm])
	fi

	dnl If we failed the first test, try without jpeg library
	if test "x$GDLIBFOUND" = "x"; then
		JMD_CHECK_LIB_ORDER(gd,gdImagePng,3,[
			GDLIBFOUND=yes
			GDLIBS="-lgd $ICONV -lz -lm -lpng"
		        ],:,[$ICONV -lz -lm -lpng])
	fi

	dnl We failed again, so try a different library ordering (without jpeg libs)
	if test "x$GDLIBFOUND" = "x"; then
		JMD_CHECK_LIB_ORDER(gd,gdImagePng,4,[
			GDLIBFOUND=yes
			GDLIBS="-lgd $ICONV -lpng -lz -lm"
	        	],:,[$ICONV -lpng -lz -lm])
	fi

	dnl Did we find the necessary GD libraries?
	if test "x$GDLIBFOUND" = "x"; then
		echo ""
		echo ""
		echo "*** GD, PNG, and/or JPEG libraries could not be located... *********"
		echo ""
		echo "Boutell's GD library is required to compile the statusmap, trends"
		echo "and histogram CGIs. Get it from https://github.com/libgd/libgd, compile"
		echo "it, and use the --with-gd-lib and --with-gd-inc arguments to specify"
		echo "the locations of the GD library and include files."
		echo ""
		echo "NOTE: In addition to the gd-devel library, you'll also need to make"
		echo "      sure you have the png-devel and jpeg-devel libraries installed"
		echo "      on your system."
		echo ""
		echo "NOTE: After you install the necessary libraries on your system:"
		echo "      1. Make sure /etc/ld.so.conf has an entry for the directory in"
		echo "         which the GD, PNG, and JPEG libraries are installed."
		echo "      2. Run 'ldconfig' to update the run-time linker options."
		echo "      3. Run 'make clean' in the Nagios distribution to clean out"
		echo "         any old references to your previous compile."
		echo "      4. Rerun the configure script."
		echo ""
		echo "NOTE: If you can't get the configure script to recognize the GD libs"
		echo "      on your system, get over it and move on to other things.  The"
		echo "      CGIs that use the GD libs are just a small part of the entire"
		echo "      Nagios package.  Get everything else working first and then"
		echo "      revisit the problem.  Make sure to check the nagios-users"
		echo "      mailing list archives for possible solutions to GD library"
		echo "      problems when you resume your troubleshooting."
		echo ""
		echo "********************************************************************"
		echo ""
		echo ""

	dnl We found the GD lib!
	else
		AC_MSG_CHECKING([for GD library])
		AC_MSG_RESULT([yes])
		if test "x$TRYSTATUSMAP" = "xyes"; then
			AC_DEFINE_UNQUOTED(USE_STATUSMAP,,[defined if the user chose to include status map])
			CGIEXTRAS="$CGIEXTRAS statusmap.cgi"
			AC_CHECK_LIB(gd,gdImageCreateTrueColor,
				AC_DEFINE(HAVE_GDIMAGECREATETRUECOLOR,1,
					[Define if your gd library has gdImageCreateTrueColor])) 
		fi

		dnl compile trends CGI
		AC_DEFINE_UNQUOTED(USE_TRENDS,,[compile trends CGI])
		CGIEXTRAS="$CGIEXTRAS trends.cgi"
	
		dnl compile histogram CGI
		AC_DEFINE_UNQUOTED(USE_HISTOGRAM,,[compile histogram CGI])
	        CGIEXTRAS="$CGIEXTRAS histogram.cgi"
	fi
fi

AC_ARG_WITH(cgiurl,
	AS_HELP_STRING([--with-cgiurl=<local-url>],
		[sets URL for cgi programs (do not use a trailing slash)]),
	cgiurl=$withval,
	cgiurl=/nagios/cgi-bin
)
AC_ARG_WITH(htmurl,
	AS_HELP_STRING([--with-htmurl=<local-url>],
		[sets URL for public html]),
	htmurl=$withval,
	htmurl=/nagios
)
AC_SUBST(htmurl)
AC_SUBST(cgiurl)

USE_NANOSLEEP=yes
AC_ARG_ENABLE(nanosleep,
	AS_HELP_STRING([--enable-nanosleep],
		[enables use of nanosleep (instead of sleep) in event timing]),
	USE_NANOSLEEP=$enableval,
	USE_NANOSLEEP=yes
)
if test "x$USE_NANOSLEEP" = "xyes"; then
	AC_DEFINE_UNQUOTED(USE_NANOSLEEP,,[enables use of nanosleep (instead of sleep)])
fi

USE_EVENTBROKER=yes
AC_ARG_ENABLE(event-broker,
	AS_HELP_STRING([--enable-event-broker],
		[enables integration of event broker routines]),
	USE_EVENTBROKER=$enableval,
	USE_EVENTBROKER=yes
)

BROKER_LDFLAGS=""
BROKERLIBS=""
some_dl_found=no
if test "x$USE_EVENTBROKER" = "xyes"; then

	dnl Which loader library should we use? libtdl or dl?
	dnl Hopefully this will be portable and not give us headaches...
	AC_CHECK_HEADER(ltdl.h,[
		AC_CHECK_LIB(ltdl,lt_dlinit,[
			AC_DEFINE(HAVE_LTDL_H,,[Which loader library should we use? libtdl or dl?])
			some_dl_found=yes
			BROKERLIBS="$BROKERLIBS -lltdl"
			])
	        ])
	if test "x$some_dl_found" != "xyes"; then
	AC_CHECK_HEADER(dlfcn.h,[
		AC_CHECK_LIB(dl,dlopen,[
			AC_DEFINE(HAVE_DLFCN_H,,[Which loader library should we use? libtdl or dl?])
			some_dl_found=yes
			BROKERLIBS="$BROKERLIBS -ldl"
			])
	        ])
	fi

	dnl - Modified from www.erlang.org
	# Check how to export functions from the broker executable, needed
	# when dynamically loaded drivers are loaded (so that they can find
	# broker functions).
	# OS'es with ELF executables using the GNU linker (Linux and recent *BSD,
	# in rare cases Solaris) typically need '-Wl,-export-dynamic' (i.e. pass
	# -export-dynamic to the linker - also known as -rdynamic and some other
	# variants); some sysVr4 system(s) instead need(s) '-Wl,-Bexport'.
	# AIX 4.x (perhaps only for x>=2) wants -Wl,-bexpall,-brtl and doesn't
	# reliably return an error for others, thus we separate it out.
	# Otherwise we assume that if the linker accepts the flag, it is needed.
	AC_MSG_CHECKING(for extra flags needed to export symbols)
	case $host_os in
		aix4*|aix5*)
			BROKER_LDFLAGS="$BROKER_LDFLAGS -Wl,-bexpall,-brtl"
		;;
		bsdi*)
    			BROKER_LDFLAGS="$BROKER_LDFLAGS -rdynamic" 
	    	;;
		*)
			save_ldflags="$LDFLAGS"
			LDFLAGS=-Wl,-export-dynamic
			AC_LINK_IFELSE([AC_LANG_PROGRAM(,)],[BROKER_LDFLAGS="$BROKER_LDFLAGS -Wl,-export-dynamic"], [
				LDFLAGS=-Wl,-Bexport
				AC_LINK_IFELSE([AC_LANG_PROGRAM(,)],[BROKER_LDFLAGS="$BROKER_LDFLAGS -Wl,-Bexport"],
					AC_MSG_RESULT(none))])
			LDFLAGS="$save_ldflags"
		;;
	esac
	AC_SUBST(BROKER_LDFLAGS)
	AC_SUBST(BROKERLIBS)
	test "x$BROKER_LDFLAGS" != "x" && AC_MSG_RESULT([$BROKER_LDFLAGS])


	dnl - Modified version from www.erlang.org
	dnl - Some 12/15/05 mods made after reading http://xaxxon.slackworks.com/phuku/dl.html
	AC_MSG_CHECKING(for linker flags for loadable modules)
	case $host_os in
		solaris2*|sysv4*)
			MOD_LDFLAGS="-G"
			MOD_CFLAGS="-fPIC"
		;;
		aix4*|aix5*)
			#MOD_LDFLAGS="-G -bnoentry -bexpall"
			MOD_LDFLAGS="-G -bM:SRE -bnoentry -bexpall"
		;;
		freebsd2*)
			# Non-ELF GNU linker
			MOD_LDFLAGS="-Bshareable"
		;;
		darwin*)
			# Mach-O linker, a shared lib and a loadable
			# object file is not the same thing.
			MOD_LDFLAGS="-bundle -flat_namespace -undefined suppress"
			MOD_CFLAGS="$MOD_CFLAGS -fno-common"
		;;
		linux* | k*bsd*-gnu*)
			# assume GNU linker and ELF
			MOD_LDFLAGS="-shared"
			MOD_CFLAGS="-fPIC"
		;;
		*bsd*)
			MOD_LDFLAGS="-shared"
			MOD_CFLAGS="-fPIC"
		;;
		*)
			# assume GNU linker and ELF
			MOD_LDFLAGS="-shared"
		;;
	esac
	AC_MSG_RESULT([$MOD_LDFLAGS])
	AC_SUBST(MOD_CFLAGS)
	AC_SUBST(MOD_LDFLAGS)

	dnl - flags for compiling workers
	WORKER_CFLAGS="-I ../../include -I ../.."
	WORKER_LDFLAGS="-L ../../lib -l nagios"
	AC_SUBST(WORKER_CFLAGS)
	AC_SUBST(WORKER_LDFLAGS)

	AC_DEFINE_UNQUOTED(USE_EVENT_BROKER,,[defined to bring in the event broker objects])
	BROKER_O="broker.o nebmods.o"
	AC_SUBST(BROKER_O)
	BROKER_H="\$(SRC_INCLUDE)/broker.h \$(SRC_INCLUDE)/nebmods.h \$(SRC_INCLUDE)/nebmodules.h \$(SRC_INCLUDE)/nebcallbacks.h \$(SRC_INCLUDE)/neberrors.h"
	AC_SUBST(BROKER_H)
fi


dnl Option for compiling under CYGWIN
nagios_name=nagios
nagiostats_name=nagiostats
cygwin=no
AC_ARG_ENABLE(cygwin,
	AS_HELP_STRING([--enable-cygwin],
		[enables building under the CYGWIN environment]),
	[cygwin=$enableval]
)
if test "x$cygwin" = "xyes"; then
	CFLAGS="${CFLAGS} -DCYGWIN" 
	nagios_name=nagios.exe;
	nagiostats_name=nagiostats.exe;
fi
AC_SUBST(nagios_name)
AC_SUBST(nagiostats_name)


dnl Should predictive failure routines be compiled in?
dnl AC_ARG_ENABLE(failure-prediction,--enable-failure-prediction will enable integration with failure prediction module (NOT HERE YET!),[
dnl	AC_DEFINE_UNQUOTED(PREDICT_FAILURES)
dnl	BASEEXTRALIBS="$BASEEXTRALIBS \$(FDATALIBS)"
dnl	echo "Failure prediction routines (incomplete!) will be compiled in..."
dnl	])

dnl Find traceroute
AC_PATH_PROG(PATH_TO_TRACEROUTE,traceroute)
AC_DEFINE_UNQUOTED(TRACEROUTE_COMMAND,"$PATH_TO_TRACEROUTE",[traceroute command to use])

dnl Enable debugging?
ENABLE_DEBUGGING=no
AC_ARG_ENABLE(debugging,AS_HELP_STRING([--enable-debugging],[DEBUGGING ONLY - Enable some debugging for Nagios Core]),
	[ENABLE_DEBUGGING=$enableval])
if test "x$ENABLE_DEBUGGING" = "xyes" -o "x$ENABLE_TESTING" = "xyes"; then
	CFLAGS="${CFLAGS} -O0 -ggdb3 -g3"
fi

dnl Enable coverage?
ENABLE_COVERAGE=no
AC_ARG_ENABLE(coverage,AS_HELP_STRING([--enable-coverage],[DEBUGGING ONLY - Enable coverage reports (use with debugging)]),
	[ENABLE_COVERAGE=$enableval])
if test "x$ENABLE_COVERAGE" = "xyes" -o "x$ENABLE_TESTING" = "xyes"; then
	CFLAGS="${CFLAGS} -fprofile-arcs -ftest-coverage"
fi

dnl Package directory for Solaris pkgmk (and other OSs, eventually)
dnl VERSION=`grep 1.0 include/common.h | cut -d ' ' -f 3 | sed 's/"//g'`
VERSION=$PKG_VERSION
PACKDIR=`pwd`/pkg
AC_SUBST(PACKDIR)
AC_SUBST(VERSION)
 
AC_MSG_CHECKING(for type va_list)
AC_COMPILE_IFELSE(
	[AC_LANG_PROGRAM([[#ifdef __STDC__
		#include <stdio.h>
		#include <stdlib.h>
		#include <stdarg.h>
		#else
		#include <sys/types.h>
		#include <stdio.h>
		#include <varargs.h>
		#endif]],
		[[va_list args;]])],
	[AC_MSG_RESULT(yes)],
	[AC_DEFINE(NEED_VA_LIST,,[defined if va_list fails to compile]) AC_MSG_RESULT(no)])

dnl Check if we should build local libtap
dnl From Nagios Plugins
dnl Have disabled autodetection of system library until later
AC_ARG_ENABLE(libtap,
  AS_HELP_STRING([--enable-libtap],
                [Enable built-in libtap for unit-testing (default: no).]),
        [enable_libtap=$enableval],
        [enable_libtap=no])
#Disabled for moment
#AM_CONDITIONAL([USE_LIBTAP_LOCAL],[test "$enable_libtap" = "yes"])

# Disabled for moment
# If not local, check if we can use the system one
#if test "$enable_libtap" != "yes"; then
#        dnl Check for libtap, to run perl-like tests
#        AC_CHECK_LIB(tap, plan_tests,
#                enable_libtap="yes"
#                )
#fi

# Finally, define tests if we use libtap
if test "x$enable_libtap" = "xyes" -o "x$ENABLE_TESTING" = "xyes"; then
	AC_CONFIG_SUBDIRS([tap])
	USE_LIBTAP=yes
else
	USE_LIBTAP=no
fi


AC_SUBST(USE_LIBTAP)
AC_SUBST(CGIEXTRAS)
AC_SUBST(GDLIBS)
AC_SUBST(BASEEXTRALIBS)
AC_SUBST(USE_EVENTBROKER)

AC_PATH_PROG(PERL,perl)
if test -z "$PERL"; then
	AC_MSG_ERROR([Cannot continue without perl!])
fi

AC_PATH_PROG(UNZIP,unzip)
if test -z "$UNZIP"; then
	AC_MSG_ERROR([Cannot continue without unzip!])
fi

dnl Need this here because of removing it from autoconf-macros
eval sbindir=$sbindir

AC_CONFIG_FILES([
	Makefile
	lib/Makefile
	base/Makefile
	common/Makefile
	contrib/Makefile
	cgi/Makefile
	html/Makefile
	include/Makefile
	module/Makefile
	worker/Makefile
	worker/ping/Makefile
	xdata/Makefile
	subst
	pkginfo
	startup/openrc-init
	startup/default-init
	startup/default-service
	startup/upstart-init
	t/Makefile 
	t-tap/Makefile
])
AC_OUTPUT

if test ! -d "html/js"; then
	mkdir -p "html/js"
fi

perl subst include/locations.h
perl subst html/config.inc.php
dnl perl subst daemon-service


echo ""
echo "Creating sample config files in sample-config/ ..."

if test ! -d "sample-config/template-object"; then
	mkdir -p "sample-config/template-object"
fi

perl subst sample-config/nagios.cfg
perl subst sample-config/cgi.cfg
perl subst sample-config/resource.cfg
perl subst sample-config/httpd.conf
perl subst sample-config/mrtg.cfg

perl subst sample-config/template-object/templates.cfg
perl subst sample-config/template-object/commands.cfg
perl subst sample-config/template-object/timeperiods.cfg
perl subst sample-config/template-object/contacts.cfg

perl subst sample-config/template-object/localhost.cfg
perl subst sample-config/template-object/windows.cfg
perl subst sample-config/template-object/printer.cfg
perl subst sample-config/template-object/switch.cfg


incdir=`eval echo $includedir`
if test "x$incdir" = "x$prefix/include"; then
	includedir=$prefix/include/nagios
fi

if test "x$USE_EVENTBROKER" != "xyes"; then
	USE_EVENTBROKER=no
fi


dnl Review options
echo ""
echo ""
AC_MSG_RESULT([*** Configuration summary for $PKG_NAME $PKG_VERSION $PKG_REL_DATE ***:])

echo ""
echo " General Options:"
echo " -------------------------"

AC_MSG_RESULT([        Nagios executable:  $nagios_name])
AC_MSG_RESULT([        Nagios user/group:  $nagios_user,$nagios_grp])
AC_MSG_RESULT([       Command user/group:  $command_user,$command_grp])
AC_MSG_RESULT([             Event Broker:  $USE_EVENTBROKER])
AC_MSG_RESULT([        Install \${prefix}:  $prefix])
AC_MSG_RESULT([    Install \${includedir}:  $includedir])
AC_MSG_RESULT([                Lock file:  $subsyslockfile])
AC_MSG_RESULT([   Check result directory:  $CHECKRESULTDIR])
AC_MSG_RESULT([           Init directory:  $initdir])
AC_MSG_RESULT([  Apache conf.d directory:  $HTTPD_CONF])
AC_MSG_RESULT([             Mail program:  $MAIL_PROG])
AC_MSG_RESULT([                  Host OS:  $host_os])
AC_MSG_RESULT([          IOBroker Method:  $IOBROKER_METHOD])

echo ""
echo " Web Interface Options:"
echo " ------------------------"

AC_MSG_RESULT([                 HTML URL:  http://localhost$htmurl/])
AC_MSG_RESULT([                  CGI URL:  http://localhost$cgiurl/])
AC_MSG_RESULT([ Traceroute (used by WAP):  $PATH_TO_TRACEROUTE])

echo ""
echo ""
echo "Review the options above for accuracy.  If they look okay,"
echo "type 'make all' to compile the main program and CGIs."
echo ""
