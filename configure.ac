#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.69])
AC_INIT([Curses Sharp],[0.8],[curses-sharp-bugs@lists.sourceforge.net])
AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_SRCDIR([CursesWrapper/wrapper.c])

AC_ARG_ENABLE(wide, 
	AS_HELP_STRING([--disable-wide],
		[disable support for curses wide character functions]),
	[curses_wide=no],
	[curses_wide=yes]
)
AC_ARG_ENABLE(panel, 
	AS_HELP_STRING([--disable-panel],
		[disable support for curses panel module]),
	[curses_panel=no],
	[curses_panel=yes]
)
AC_ARG_ENABLE(mouse, 
	AS_HELP_STRING([--disable-mouse],
		[disable support for curses mouse]),
	[curses_mouse=no],
	[curses_mouse=yes]
)
AC_ARG_ENABLE(debug, 
	AS_HELP_STRING([--enable-debug],
		[link to libraries with debugging symbols]),
	[wrapper_debug=yes],
	[wrapper_debug=no]
)


AC_CANONICAL_TARGET

AM_INIT_AUTOMAKE([foreign])

dnl checks for programs

AC_PROG_CC		
AC_PROG_CC_C_O
AC_PROG_CC_C89
AC_PROG_CPP_WERROR
LT_INIT

dnl pkg-config
PKG_PROG_PKG_CONFIG
if test "x$PKG_CONFIG" = "x"; then
	AC_MSG_ERROR([You need to install pkg-config])
fi

dnl Mono runtime
AC_PATH_PROG([MONO], [mono], [no])
if test "x$MONO" = "xno"; then
	AC_MSG_ERROR([Mono runtime not found])
fi
if test "x$wrapper_debug" != "xno"; then
    MONO="$MONO --debug"
fi

dnl C# compiler
AC_PATH_PROG([CSC], [mcs], [no])
if test "x$CSC" = "xno"; then
	AC_MSG_ERROR([Cannot find "mcs" in your path])
fi

CursesSharp_CSCFLAGS="${CursesSharp_CSCFLAGS}"

CPPFLAGS="${CPPFLAGS}"
CFLAGS="${CFLAGS}"
LDFLAGS="${LDFLAGS}"
CSCFLAGS="${CSCFLAGS}"

CFLAGS="-Werror $CFLAGS"

case "$target_os" in
	freebsd*)
		CPPFLAGS="$CPPFLAGS -I/usr/local/include"
		LDFLAGS="$LDFLAGS -L/usr/local/lib"
	;;
esac

if test "x$wrapper_debug" != "xno"; then
    CSCFLAGS="-debug -define:DEBUG -define:TRACE $CSCFLAGS"
fi

dnl checks for libraries

dnl check for a curses library, preferably with wide character functions
curses_search_libs=""
if test "x$wrapper_debug" != "xno"; then
	curses_search_libs="$curses_search_libs ncursesw_g ncurses_g curses_g"
fi
curses_search_libs="$curses_search_libs ncursesw ncurses curses"
CS_SEARCH_LIBS([initscr move], [$curses_search_libs], [], [], [dl])

dnl checks for header files

dnl enable additional functions in curses
AC_DEFINE([_XOPEN_SOURCE], [1], [X/Open conformance])
AC_DEFINE([_XOPEN_SOURCE_EXTENDED], [1], [X/Open Extended conformance])

dnl standard headers
AC_HEADER_ASSERT
AC_HEADER_STDC

dnl check for basic wide character types
AC_CHECK_HEADERS([wchar.h])
if test "x$curses_wide" != "xno" -a "x$ac_cv_header_wchar_h" != "xyes"; then
	AC_MSG_NOTICE([Disabling wide character support])
	curses_wide=no
fi

dnl check for curses headers
AC_CHECK_HEADERS([ncursesw/ncurses.h ncurses.h curses.h])
AC_CHECK_HEADERS([panel.h])

if test "x$curses_panel" != "xno" -a "x$ac_cv_header_panel_h" != "xyes"; then
	AC_MSG_NOTICE([Disabling curses panel support])
	curses_panel=no
fi

dnl checks for types
AC_TYPE_SIZE_T

wchar_includes="\
#ifdef HAVE_WCHAR_H
#include <wchar.h>
#endif"

have_wchar_types=yes
AC_CHECK_TYPES([wchar_t, wint_t], [], [have_wchar_types=no],
	[$wchar_includes]
)
AC_CHECK_SIZEOF([wchar_t], [], [$wchar_includes])

if test "x$curses_wide" != "xno" -a "x$have_wchar_types" != "xyes"; then
	AC_MSG_NOTICE([Disabling wide character support])
	curses_wide=no
fi


dnl checks for declarations

curses_includes="\
#if defined(HAVE_NCURSESW_NCURSES_H)
#include <ncursesw/ncurses.h>
#endif
#if defined(HAVE_NCURSES_H)
#include <ncurses.h>
#endif
#if defined(HAVE_CURSES_H)
#include <curses.h>
#endif"

have_curses_mouse=yes
AC_CHECK_DECL([NCURSES_MOUSE_VERSION], [], 
	[have_curses_mouse=no],
	[$curses_includes]
)
if test "x$curses_mouse" != "xno" -a "x$have_curses_mouse" != "xyes"; then
	AC_MSG_NOTICE([Disabling curses mouse support])
	curses_mouse=no
fi

dnl check for compiler characteristics
AC_C_INLINE

dnl checks for library functions
AM_ICONV

dnl check curses headers for declarations of wide character functions
have_wide_funcs=yes
AC_CHECK_FUNCS(
	[waddnwstr mvwaddnwstr wgetn_wstr mvwgetn_wstr	\
	 wins_nwstr mvwins_nwstr winnwstr mvwinnstr	\
	 slk_wset],
	[],
	[have_wide_funcs=no]
)
AC_CHECK_FUNCS([has_mouse])

if test "x$curses_wide" != "xno" -a "x$have_wide_funcs" != "xyes"; then
	AC_MSG_NOTICE([Disabling wide character support])
	curses_wide=no
fi

dnl if test "x$curses_wide" != "xyes"; then
dnl	AC_MSG_ERROR([Required wide character curses implementation not found])
dnl fi
if test "x$curses_wide" != "xno"; then
	AC_DEFINE([CURSES_WIDE], [1],
		[Define if using curses wide character functions]
	)
fi


AM_CONDITIONAL([HAVE_CURSES_PANEL], [test "x$curses_panel" != "xno"])
if test "x$curses_panel" != "xno"; then
	AC_DEFINE([HAVE_CURSES_PANEL], [1],
		[Define if using curses panel module]
	)
	CursesSharp_CSCFLAGS="-define:HAVE_CURSES_PANEL $CursesSharp_CSCFLAGS"
fi

AM_CONDITIONAL([HAVE_CURSES_MOUSE], [test "x$curses_mouse" != "xno"])
if test "x$curses_mouse" != "xno"; then
	AC_DEFINE([HAVE_CURSES_MOUSE], [1],
		[Define if using curses mouse API]
	)
	CursesSharp_CSCFLAGS="-define:HAVE_CURSES_MOUSE $CursesSharp_CSCFLAGS"
fi

AC_SUBST([CursesSharp_CSCFLAGS])

AC_SUBST([CPPFLAGS])
AC_SUBST([CFLAGS])
AC_SUBST([LDFLAGS])
AC_SUBST([CSCFLAGS])


AC_CONFIG_HEADERS([config.h])
AC_CONFIG_FILES([
	Makefile 
	CursesWrapper/Makefile 
	AttributeGen/Makefile
	CursesSharp/Makefile
	CursesSharp.pc
])
AC_CONFIG_FILES([
	FireworkDemo/Makefile
	FireworkDemo/FireworkDemo
	RainDemo/Makefile
	RainDemo/RainDemo
	UnicodeDemo/Makefile
	UnicodeDemo/UnicodeDemo
	HelloWorldDemo/Makefile
	HelloWorldDemo/HelloWorldDemo
])

AC_OUTPUT

echo ""
echo "Settings:"
echo "  Wide character functions: $curses_wide"
echo "  Panel support           : $curses_panel"
echo "  Mouse support           : $curses_mouse"
echo ""

