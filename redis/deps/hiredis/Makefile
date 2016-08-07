# Hiredis Makefile
# Copyright (C) 2010-2011 Salvatore Sanfilippo <antirez at gmail dot com>
# Copyright (C) 2010-2011 Pieter Noordhuis <pcnoordhuis at gmail dot com>
# This file is released under the BSD license, see the COPYING file

OBJ=net.o hiredis.o sds.o async.o
EXAMPLES=hiredis-example hiredis-example-libevent hiredis-example-libev
TESTS=hiredis-test
LIBNAME=libhiredis

HIREDIS_MAJOR=0
HIREDIS_MINOR=11

# redis-server configuration used for testing
REDIS_PORT=56379
REDIS_SERVER=redis-server

# Fallback to gcc when $CC is not in $PATH.
#CC=aCC
OPTIMIZATION?=-O3
WARNINGS=-Wall -W -Wstrict-prototypes -Wwrite-strings
DEBUG?= -g -ggdb
REAL_CFLAGS=$(OPTIMIZATION) -fPIC $(CFLAGS) $(WARNINGS) $(DEBUG) $(ARCH)
REAL_LDFLAGS=$(LDFLAGS) $(ARCH)

DYLIBSUFFIX=so
STLIBSUFFIX=a
DYLIB_MINOR_NAME=$(LIBNAME).$(DYLIBSUFFIX).$(HIREDIS_MAJOR).$(HIREDIS_MINOR)
DYLIB_MAJOR_NAME=$(LIBNAME).$(DYLIBSUFFIX).$(HIREDIS_MAJOR)
DYLIBNAME=$(LIBNAME).$(DYLIBSUFFIX)
DYLIB_MAKE_CMD=$(CC) -shared -Wl,-soname,$(DYLIB_MINOR_NAME) -o $(DYLIBNAME) $(LDFLAGS)
STLIBNAME=$(LIBNAME).$(STLIBSUFFIX)
STLIB_MAKE_CMD=ar rcs $(STLIBNAME)

all: $(DYLIBNAME) $(STLIBNAME) clean

# Deps (use make dep to generate this)
net.o: net.c fmacros.h net.h hiredis.h
async.o: async.c async.h hiredis.h sds.h dict.c dict.h
hiredis.o: hiredis.c fmacros.h hiredis.h net.h sds.h
sds.o: sds.c sds.h
test.o: test.c hiredis.h


CC_AIX=cc
CFLAG_AIX=-qlanglvl=stdc99 -q64 -c -g
DFLAG_AIX= -q64 -qmkshrobj
AR_AIX=ar  -X64 ruv

CC_HP-UX=cc
CFLAG_HP-UX=-AC99 +DD64 -c -g
DFLAG_HP-UX=-AC99 +DD64 -b
AR_HP-UX=ar rcu

CC_Linux=cc
CFLAG_Linux=-c -g -fPIC
DFLAG_Linux=-shared -fPIC
AR_Linux=ar ruv

CC=${CC_${OS}}
CFLAG=${CFLAG_${OS}}
DFLAG=${DFLAG_${OS}}
AR=${AR_${OS}}



$(DYLIBNAME): $(OBJ)
	${CC} ${DFLAG} -o $@ $(OBJ)

$(STLIBNAME): $(OBJ)
	${AR} $(STLIBNAME)  $(OBJ)

dynamic: $(DYLIBNAME)
static: $(STLIBNAME)



.c.o:
	$(CC) ${CFLAG} $<


clean:
	rm -rf $(DYLIBNAME) $(TESTS) examples/hiredis-example* *.o *.gcda *.gcno *.gcov

dep:
	$(CC) -MM *.c

# Installation related variables and target
PREFIX?=/usr/local
INSTALL_INCLUDE_PATH= $(PREFIX)/include/hiredis
INSTALL_LIBRARY_PATH= $(PREFIX)/lib


