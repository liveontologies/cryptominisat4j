#-----------------------------------------------------------------------#
#- GLOBAL DEFS ---------------------------------------------------------#
#-----------------------------------------------------------------------#

# Keep this as generic as possible.

NAME=${library.name}
VERSION=${library.version}
ARCHIVE=${library.archive.name}
URL=${library.download.url}

#-----------------------------------------------------------------------#
# Solver signatures have to be both valid file names and C symbols.
# Since Picosat uses a dash '-' for the package name, we have to
# differentiate between with (directory / package name) and without
# dash (the signature).

SIG=$(NAME)$(VERSION)$(ARCHIVE)
DIR=$(NAME)-$(ARCHIVE)
TARGET	?= dlib

#-----------------------------------------------------------------------#

# Make sets default value for CC, we want to override them
# but not if it set by the user
# see: https://stackoverflow.com/a/42958970
ifeq ($(origin CC),default)
export CC 		= gcc
endif
ifeq ($(origin CXX),default)
export CXX		= g++
endif
export AR		?= ar
# to build universal binary on MacOS, we switch ar rc -> -static -o  
AR += rc
export CXXFLAGS	+= -Wall -DNDEBUG -O3 -fPIC

#-----------------------------------------------------------------------#
#- REQUIRED TOP RULES --------------------------------------------------#
#-----------------------------------------------------------------------#

all: $(TARGET)

clean:
	rm -rf $(DIR)
	rm -f *.o *.a *.tar.gz *.dylib *.dll *.so

#-----------------------------------------------------------------------#
#- INVISIBLE INTERNAL SUB RULES ----------------------------------------#
#-----------------------------------------------------------------------#

# enable verbatim output
ifeq ($(VERB),)
VB=@
else
VB=
endif

# the location of this makefile (independent of the current directory)
MROOT := $(dir $(abspath $(lastword $(MAKEFILE_LIST))))

# cross-compilation
ifneq ($(CHOST),)
	CROSS_PREFIX = $(CHOST)-
	CC := $(CROSS_PREFIX)$(CC)
	CXX := $(CROSS_PREFIX)$(CXX)
	# prefer -posix version if there are any
	ifneq ($(shell command -v "$(CC)-posix"),)
		CC := $(CC)-posix
		CXX := $(CXX)-posix 
	endif	
	AR := $(CROSS_PREFIX)$(AR)
	UNAME := $(shell echo "$(CHOST)" | sed -e 's/^[^-]*-\([^-]*\)\s*$$/\1/' -e 's/^[^-]*-[^-]*-\([^-]*\)\s*$$/\1/' -e 's/^[^-]*-[^-]*-\([^-]*\)-.*$$/\1/')#						
endif

# determine operating system name
UNAME ?= $(shell uname -s)

# the build static library
STLIB = $(DIR)/build/lib/libipasircryptominisat5.a
# the build glue object file implementing the ipasir interface
IPASIRPATH=$(DIR)/build/cmsat5-src/CMakeFiles/ipasircryptominisat5.dir/
IPASIRGLUEOBJ=$(IPASIRPATH)/ipasir.cpp.o

ifeq ($(UNAME), Darwin)
 	DLIB = lib$(NAME).dylib
 	# build universal "fat" binaries
 	CMAKE_OPTIONS = -D CMAKE_OSX_ARCHITECTURES='x86_64;arm64'
 	CFLAGS += -arch x86_64 -arch arm64
 	AR = libtool -static -o
else ifeq ($(UNAME), Linux)
 	DLIB = lib$(NAME).so
else ifeq ($(UNAME), mingw32)
 	DLIB = $(NAME).dll  
 	CMAKE_OPTIONS=-D CMAKE_SYSTEM_NAME=Windows
 	IPASIRGLUEOBJ=$(IPASIRPATH)/ipasir.cpp.obj  
else
 	$(error Platform not supported: $(UNAME))
endif 

dlib: $(DLIB)


# shared library for Mac OS
lib$(NAME).dylib: $(STLIB)
	$(VB) $(CXX) $(CFLAGS) -g -I. -o $@ $(IPASIRGLUEOBJ) $^ -dynamiclib -install_name $@ 

# shared library for Linux
lib$(NAME).so: $(STLIB)
	$(VB) $(CXX) -o $@ $(IPASIRGLUEOBJ) $^ $(CFLAGS) -g -I. -shared -Wl,-soname,$@ 

# shared library for MinGW
$(NAME).dll: $(STLIB)	 	
	$(VB) $(CXX) -o $@ $(IPASIRGLUEOBJ) $^ $(CFLAGS) -static-libgcc -static-libstdc++ -g -I. -shared -Wl,--subsystem,windows -Wl,-Bstatic -lstdc++ -lpthread

# static solver library
$(STLIB): $(DIR)
	@#
	@# configure and build library
	@#
	cd $(DIR); sh ../build.sh $(CMAKE_OPTIONS)


$(DIR): $(ARCHIVE).tar.gz
	@#
	@# extract library
	@#
	$(VB) tar xf $<

$(ARCHIVE).tar.gz:
	@#
	@# download library
	@#
	wget "$(URL)$@"

#-----------------------------------------------------------------------#

.FORCE:
.PHONY: all clean
