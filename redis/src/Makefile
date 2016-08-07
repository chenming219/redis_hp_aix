# Redis Makefile
# Copyright (C) 2009 Salvatore Sanfilippo <antirez at gmail dot com>
# This file is released under the BSD license, see the COPYING file
#
# The Makefile composes the final FINAL_CFLAGS and FINAL_LDFLAGS using
# what is needed for Redis plus the standard CFLAGS and LDFLAGS passed.
# However when building the dependencies (Jemalloc, Lua, Hiredis, ...)
# CFLAGS and LDFLAGS are propagated to the dependencies, so to pass
# flags only to be used when compiling / linking Redis itself REDIS_CFLAGS
# and REDIS_LDFLAGS are used instead (this is the case of 'make gcov').
#
# Dependencies are stored in the Makefile.dep file. To rebuild this file
# Just use 'make dep', but this is only needed by developers.

uname_S=${OS}
OPTIMIZATION?=-O2
DEPENDENCY_TARGETS=hiredis linenoise lua

CC_AIX=cc
CFLAG_AIX=-qlanglvl=stdc99 -q64 -c -g
BFLAG_AIX=-qlanglvl=stdc99 -q64 -g
DFLAG_AIX= -q64 -qmkshrobj
AR_AIX=ar  -X64 ruv
LIBS_AIX= -lpthread  -ldl -lrt -lxnet -lm -lcrypto

CC_HP-UX=cc
CFLAG_HP-UX=-AC99 +DD64 -c -g
BFLAG_HP-UX=-AC99 +DD64
DFLAG_HP-UX=-AC99 +DD64 -b
AR_HP-UX=ar ruv
LIBS_HP-UX= -lpthread  -ldl -lrt -lxnet -lm -lcrypto

CC_Linux=cc
CFLAG_Linux=-c -g -fPIC
DFLAG_Linux=-shared -fPIC
AR_Linux=ar ruv
LIBS_Linux= -lpthread  -ldl -lrt -lm

CC=${CC_${OS}}
CFLAG=${CFLAG_${OS}}
BFLAG=${BFLAG_${OS}}
DFLAG=${DFLAG_${OS}}
AR=${AR_${OS}}
LIBS=${LIBS_${OS}}



# Default settings
STD=-std=c99 -pedantic
WARN=-Wall -W
OPT=$(OPTIMIZATION)

PREFIX?=/usr/local
INSTALL_BIN=$(PREFIX)/bin
INSTALL=install

# Default allocator
MALLOC=libc

# Backwards compatibility for selecting an allocator

# Override default settings if possible
#-include .make-settings

CFLAGS=${BFLAG}
FINAL_LDFLAGS=$(LDFLAGS) $(REDIS_LDFLAGS) $(DEBUG)
FINAL_LIBS=-lm
DEBUG=-g -ggdb

FINAL_LDFLAGS= ${BFLAG}
FINAL_LIBS=${LIBS}

# Include paths to dependencies
FINAL_CFLAGS= -I../deps/hiredis -I../deps/linenoise -I../deps/lua/src


REDIS_CC=$(CC) ${BFLAG}
REDIS_LD=$(QUIET_LINK)$(CC) $(FINAL_LDFLAGS)
REDIS_INSTALL=$(QUIET_INSTALL)$(INSTALL)


REDIS_SERVER_NAME=redis-server
REDIS_SENTINEL_NAME=redis-sentinel
REDIS_SERVER_OBJ=adlist.o ae.o anet.o dict.o redis.o sds.o zmalloc.o lzf_c.o lzf_d.o pqsort.o zipmap.o sha1.o ziplist.o release.o networking.o util.o object.o db.o replication.o rdb.o t_string.o t_list.o t_set.o t_zset.o t_hash.o config.o aof.o pubsub.o multi.o debug.o sort.o intset.o syncio.o cluster.o crc16.o endianconv.o slowlog.o scripting.o bio.o rio.o rand.o memtest.o crc64.o bitops.o sentinel.o notify.o setproctitle.o blocked.o hyperloglog.o latency.o sparkline.o
REDIS_CLI_NAME=redis-cli
REDIS_CLI_OBJ=anet.o sds.o adlist.o redis-cli.o zmalloc.o release.o anet.o ae.o crc64.o
REDIS_BENCHMARK_NAME=redis-benchmark
REDIS_BENCHMARK_OBJ=ae.o anet.o redis-benchmark.o sds.o adlist.o zmalloc.o redis-benchmark.o
REDIS_CHECK_DUMP_NAME=redis-check-dump
REDIS_CHECK_DUMP_OBJ=redis-check-dump.o lzf_c.o lzf_d.o crc64.o
REDIS_CHECK_AOF_NAME=redis-check-aof
REDIS_CHECK_AOF_OBJ=redis-check-aof.o

all: $(REDIS_SERVER_NAME) 
	@echo ""
	@echo "Hint: It's a good idea to run 'make test' ;)"
	@echo ""

.PHONY: all

# Deps (use make dep to generate this)
include Makefile.dep

dep:
	$(REDIS_CC) -MM *.c > Makefile.dep

.PHONY: dep

persist-settings: distclean
	echo STD=$(STD) >> .make-settings
	echo WARN=$(WARN) >> .make-settings
	echo OPT=$(OPT) >> .make-settings
	echo MALLOC=$(MALLOC) >> .make-settings
	echo CFLAGS=$(CFLAGS) >> .make-settings
	echo LDFLAGS=$(LDFLAGS) >> .make-settings
	echo REDIS_CFLAGS=$(REDIS_CFLAGS) >> .make-settings
	echo REDIS_LDFLAGS=$(REDIS_LDFLAGS) >> .make-settings
	echo PREV_FINAL_CFLAGS=$(FINAL_CFLAGS) >> .make-settings
	echo PREV_FINAL_LDFLAGS=$(FINAL_LDFLAGS) >> .make-settings
	-(cd ../deps && $(MAKE) $(DEPENDENCY_TARGETS))

.PHONY: persist-settings



# redis-server
$(REDIS_SERVER_NAME): $(REDIS_SERVER_OBJ)
	${CC} ${BFLAG} -o $@ $(REDIS_SERVER_OBJ) ../deps/hiredis/libhiredis.a ../deps/lua/src/liblua.a $(FINAL_LIBS)

# Because the jemalloc.h header is generated as a part of the jemalloc build,
# building it should complete before building any other object. Instead of
# depending on a single artifact, build all dependencies first.
.c.o:
	${REDIS_CC} ${FINAL_CFLAGS} -g -c $<

clean:
	rm -rf $(REDIS_SERVER_NAME) $(REDIS_SENTINEL_NAME) $(REDIS_CLI_NAME) $(REDIS_BENCHMARK_NAME) $(REDIS_CHECK_DUMP_NAME) $(REDIS_CHECK_AOF_NAME) *.o *.gcda *.gcno *.gcov redis.info lcov-html

.PHONY: clean

distclean: clean
	-(cd ../deps && $(MAKE) distclean)
	-(rm -f .make-*)

.PHONY: distclean

test: $(REDIS_SERVER_NAME) $(REDIS_CHECK_AOF_NAME)
	@(cd ..; ./runtest)

test-sentinel: $(REDIS_SENTINEL_NAME)
	@(cd ..; ./runtest-sentinel)

check: test

lcov:
	$(MAKE) gcov
	@(set -e; cd ..; ./runtest --clients 1)
	@geninfo -o redis.info .
	@genhtml --legend -o lcov-html redis.info

.PHONY: lcov

bench: $(REDIS_BENCHMARK_NAME)
	./$(REDIS_BENCHMARK_NAME)

32bit:
	@echo ""
	@echo "WARNING: if it fails under Linux you probably need to install libc6-dev-i386"
	@echo ""
	$(MAKE) CFLAGS="-m32" LDFLAGS="-m32"

gcov:
	$(MAKE) REDIS_CFLAGS="-fprofile-arcs -ftest-coverage -DCOVERAGE_TEST" REDIS_LDFLAGS="-fprofile-arcs -ftest-coverage"

noopt:
	$(MAKE) OPTIMIZATION="-O0"

valgrind:
	$(MAKE) OPTIMIZATION="-O0" MALLOC="libc"

src/help.h:
	@../utils/generate-command-help.rb > help.h

install: all
	@mkdir -p $(INSTALL_BIN)
	$(REDIS_INSTALL) $(REDIS_SERVER_NAME) $(INSTALL_BIN)
	$(REDIS_INSTALL) $(REDIS_BENCHMARK_NAME) $(INSTALL_BIN)
	$(REDIS_INSTALL) $(REDIS_CLI_NAME) $(INSTALL_BIN)
	$(REDIS_INSTALL) $(REDIS_CHECK_DUMP_NAME) $(INSTALL_BIN)
	$(REDIS_INSTALL) $(REDIS_CHECK_AOF_NAME) $(INSTALL_BIN)
	@ln -sf $(REDIS_SERVER_NAME) $(INSTALL_BIN)/$(REDIS_SENTINEL_NAME)
