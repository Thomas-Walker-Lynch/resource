# This makefile is usually called through a local makefile rather than directly.

# should add test that incdir is not pwd before deleting the include
# file. Currently it is working for us despite this problem because dist-clean is
# looking for $(module).h while the include file is always $(module).lib.h.

#--------------------------------------------------------------------------------
# useful macros, defines the directory structure, local targets, install target
#
-include $(RESOURCES)/library/makefile-env

#----------------------------a----------------------------------------------------
# local customizations
#  user must define things needed by the local makefile, the compiler, and flags

-include makefile-environment

#--------------------------------------------------------------------------------
# files named by convention
#   this implements the .lib.c  and the .cli.c convention
#

# these are the sources edited by the programmer
C_SOURCE_LIB=  $(wildcard $(SRCDIR)/*.lib.c)
C_SOURCE_CLI=  $(wildcard $(SRCDIR)/*.cli.c)
CC_SOURCE_LIB= $(wildcard $(SRCDIR)/*.lib.cc)
CC_SOURCE_CLI= $(wildcard $(SRCDIR)/*.cli.cc)

#remove the suffix to get base name
C_BASE_LIB=  $(sort $(patsubst %.lib.c,  %, $(notdir $(C_SOURCE_LIB))))
C_BASE_CLI=  $(sort $(patsubst %.cli.c,  %, $(notdir $(C_SOURCE_CLI))))
CC_BASE_LIB= $(sort $(patsubst %.lib.cc, %, $(notdir $(CC_SOURCE_LIB))))
CC_BASE_CLI= $(sort $(patsubst %.cli.cc, %, $(notdir $(CC_SOURCE_CLI))))

# two sets of object files, one for the lib, and one for the command line interface progs
OBJECT_LIB= $(patsubst %, $(TMPDIR)/%.lib.o, $(C_BASE_LIB) $(CC_BASE_LIB))
OBJECT_CLI= $(patsubst %, $(TMPDIR)/%.cli.o, $(C_BASE_CLI) $(CC_BASE_CLI))

# executables are made from CLI sources
EXEC= $(patsubst %, $(EXECDIR)/%, $(C_BASE_CLI) $(CC_BASE_CLI))


#--------------------------------------------------------------------------------
# import the rules
#

-include $(DEPFILE)


#--------------------------------------------------------------------------------
# targets

.PHONY: all
all: version

.PHONY: version
version:
	@echo makefile version 6.0
	if [ ! -z "$(C)" ]; then $(C) -v; fi
	if [ ! -z "$(CC)" ]; then $(CC) -v; fi
	make -v

.PHONY: info
info:
	@echo "SHELL: " $(SHELL)
	@echo "ECHO: " $(ECHO)
	@echo "C: " $(C)
	@echo "CFLAGS: " $(CFLAGS)
	@echo "CC: " $(CC)
	@echo "CCFLAGS: " $(CCFLAGS)
	@echo "LINKFLAGS: " $(LINKFLAGS)
	@echo "DEPRDIR: " $(DEPRDIR)
	@echo "DOCDIR: " $(DOCDIR)
	@echo "EXECDIR: " $(EXECDIR)
	@echo "INCDIR: " $(INCDIR)
	@echo "LIBDIR: " $(LIBDIR)
	@echo "TESTDIR: " $(TESTDIR)
	@echo "TMPDIR: " $(TMPDIR)
	@echo "TRYDIR: " $(TRYDIR)
	@echo "DEPFILE: " $(DEPFILE)
	@echo "LIBFILE: " $(LIBFILE)
	@echo "INCFILE: " $(INCFILE)
	@echo "C_SOURCE_LIB: " $(C_SOURCE_LIB)
	@echo "C_SOURCE_CLI: " $(C_SOURCE_CLI)
	@echo "CC_SOURCE_LIB: " $(CC_SOURCE_LIB)
	@echo "CC_SOURCE_CLI: " $(CC_SOURCE_CLI)
	@echo "C_BASE_LIB: " $(C_BASE_LIB)
	@echo "C_BASE_CLI: " $(C_BASE_CLI)
	@echo "CC_BASE_LIB: " $(CC_BASE_LIB)
	@echo "CC_BASE_CLI: " $(CC_BASE_CLI)
	@echo "OBJECT_LIB: " $(OBJECT_LIB)
	@echo "OBJECT_CLI: " $(OBJECT_CLI)
	@echo "EXEC: " $(EXEC)

# should be safe to run this in an already setup or partially setup directory
# gee looks like a candidate for a makefile function ..
.PHONY: setup
setup:
	[ ! -e $(DEPRDIR) ]  && mkdir $(DEPRDIR) || true
	[ ! -e $(DOCDIR)  ]  && mkdir $(DOCDIR)  || true
	[ ! -e $(EXECDIR) ]  && mkdir $(EXECDIR) || true
	[ ! -e $(INCDIR)  ]  && mkdir $(INCDIR)  || true
	[ ! -e $(LIBDIR)  ]  && mkdir $(LIBDIR)  || true
	[ ! -e $(SRCDIR)  ]  && mkdir $(SRCDIR)  || true
	[ ! -e $(TESTDIR) ]  && mkdir $(TESTDIR) || true
	[ ! -e $(TMPDIR)  ]  && mkdir $(TMPDIR)  || true
	[ ! -e $(TRYDIR)  ]  && mkdir $(TRYDIR)  || true

.PHONY: dep
dep:
	@if [ -z "$(CC)" ]; then\
	  if [ -z "$(C)" ]; then\
	    $(ECHO) "No compiler specified";\
	    exit 1;\
	  else\
	    echo "C compiler only deps" ;\
	    $(C) $(CFLAGS) -MM  $(C_SOURCE_LIB) $(C_SOURCE_CLI) | sed 's|^.*\.o|$(TMPDIR)/&|' >> $(DEPFILE);\
	    echo "deps for C linking";\
	    for i in $(C_BASE_CLI); do\
	      $(ECHO) >> $(DEPFILE);\
	      $(ECHO) "$(EXECDIR)/$$i : $(TMPDIR)/$$i.cli.o $(LIBFILE)" >> $(DEPFILE);\
	      $(ECHO) "	$(C) -o $(EXECDIR)/$$i $(TMPDIR)/$$i.cli.o $(LINKFLAGS)" >> $(DEPFILE);\
	    done;\
	  fi;\
	else\
	  $(CC) $(CCFLAGS) -MM  $(CC_SOURCE_LIB) $(CC_SOURCE_CLI) | sed 's|^.*\.o|$(TMPDIR)/&|' >> $(DEPFILE);\
	  if [ -z "$C()" ]; then\
	    echo "CC compiler only deps" ;\
	  else\
	    echo "CC and C mixed compile deps" ;\
	    $(C) $(CFLAGS) -MM   $(C_SOURCE_LIB) $(C_SOURCE_CLI) | sed 's|^.*\.o|$(TMPDIR)/&|' >> $(DEPFILE);\
	  fi;\
	  echo "deps for CC linking";\
	  for i in $(CC_BASE_CLI) $(C_BASE_CLI) ; do\
	    $(ECHO) >> $(DEPFILE);\
	    $(ECHO) "$(EXECDIR)/$$i : $(TMPDIR)/$$i.cli.o $(LIBFILE)" >> $(DEPFILE);\
	    $(ECHO) "	$(CC) -o $(EXECDIR)/$$i $(TMPDIR)/$$i.cli.o $(LINKFLAGS)" >> $(DEPFILE);\
	  done;\
	fi

.PHONY: lib
lib: $(LIBFILE)

$(LIBFILE): $(OBJECT_LIB)
	ar rcs $(LIBFILE) $(OBJECT_LIB)

.PHONY: exec
exec: $(LIBFILE)
	make sub_exec

.PHONY: sub_exec
sub_exec: $(EXEC)

.PHONY: share
share:
	if [ ! -z "$(wildcard $(LIBDIR)/*)" ]; then cp $(LIBDIR)/* $(DIST_TARGET)/lib; fi
	if [ ! -z "$(wildcard $(INCDIR)/*)" ]; then cp $(INCDIR)/* $(DIST_TARGET)/include; fi
	if [ ! -z "$(wildcard $(EXECDIR)/*)" ]; then cp $(EXECDIR)/* $(DIST_TARGET)/bin; fi

.PHONY: clean
clean:
	for i in $(wildcard tmp/*); do rm $$i || true; done 

.PHONY: dist-clean
dist-clean: clean
	for i in $(EXEC); do [ -e $$i ] && rm $$i || true; done 
	rm $(LIBFILE) || true

# recipes
 $(TMPDIR)/%.o : $(SRCDIR)/%.c
	$(C) $(CFLAGS) -o $@ -c $<

 $(TMPDIR)/%.o : $(SRCDIR)/%.cc
	$(CC) $(CCFLAGS) -o $@ -c $<


