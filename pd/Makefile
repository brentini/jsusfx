#DEBUG_OPT=-g
OBJS=jsusfx_pd.o ../jsusfx.o ../WDL/eel2/nseel-compiler.o ../WDL/eel2/nseel-eval.o ../WDL/eel2/nseel-ram.o ../WDL/eel2/nseel-yylex.o ../WDL/eel2/nseel-cfunc.o  ../WDL/fft.o 
OBJS2=
PD_INCLUDE = $(PD_PATH)/include/pd

ALL_CFLAGS = -I"$(PD_INCLUDE)" -DWDL_FFT_REALSIZE=8 $(DEBUG_OPT)
ALL_LDFLAGS = $(DEBUG_OPT)

UNAME_S := $(shell uname -s)

ifeq ($(UNAME_S),Darwin)
	## OS X
	EXTENSION=pd_darwin
	OS = macosx
	PD_PATH = /Applications/Pd-extended.app/Contents/Resources
	OPT_CFLAGS = -ftree-vectorize -fast
	FAT_FLAGS = -arch i386 -arch x86_64 -mmacosx-version-min=10.5
	ALL_CFLAGS += $(FAT_FLAGS) -fPIC $(OPT_CFLAGS)
	BUNDLE_LOADER = $(shell test ! -e $(PD_PATH)/bin/pd || echo -bundle_loader $(PD_PATH)/bin/pd)
	ALL_LDFLAGS += $(FAT_FLAGS) -headerpad_max_install_names -bundle $(BUNDLE_LOADER) -undefined dynamic_lookup
	SHARED_LDFLAGS += $(FAT_FLAGS) -dynamiclib -undefined dynamic_lookup -compatibility_version 1 -current_version 1.0
	STRIP = strip -x
else
	## Linux
	# USE TO native (non-portable) flag with caution, it requires php, nasm and execstack must be run on pd
	#PORTABLE=true	
	EXTENSION=pd_linux
  	ARCH := $(shell uname -m)
	OS = linux
	PD_PATH = /usr
	OPT_CFLAGS = -funroll-loops -fomit-frame-pointer -O6 
	ALL_CFLAGS += -fPIC $(OPT_CFLAGS) -Wformat -Werror=format-security
	ALL_LDFLAGS += -shared -fPIC -Wl,-rpath,"\$$ORIGIN",--enable-new-dtags -rdynamic
	STRIP = strip --strip-unneeded -R .note -R .comment
endif

ifdef PORTABLE
	CFLAGS +=  -DEEL_TARGET_PORTABLE
else
	ifeq ($(UNAME_S),Darwin)
		# by default, we take for granted that OS X builds are FAT (i386 + x86_64)
		OBJS2 += ../WDL/eel2/asm-nseel-x64-macho.o    
	endif
	ifeq ($(UNAME_S),Linux)
		ifeq ($(ARCH),x86_64)
		  OBJS2 += ../WDL/eel2/asm-nseel-x64.o
		endif
	endif
endif

default: jsusfx~.$(EXTENSION)

%.o : %.c
	$(CC) $(ALL_CFLAGS) $(CFLAGS) -c -o $@ $^

%.o : %.cpp
	$(CXX) $(ALL_CFLAGS) $(CXXFLAGS) -c -o $@ $^

jsusfx~.$(EXTENSION): jsusfx_pd.o $(OBJS) $(OBJS2)
	$(CXX) $(ALL_LDFLAGS) $(LDFLAGS) -o "jsusfx~.$(EXTENSION)" $(OBJS) $(OBJS2)

# This is the assembly part that is only required for x64 ; php and nasm are required
../WDL/eel2/asm-nseel-x64-macho.o: 
	cd ../WDL/eel2 && php a2x64.php macho64x

../WDL/eel2/asm-nseel-x64.o: 
	cd ../WDL/eel2 && php a2x64.php elf64

clean:
	rm -f jsusfx~.$(EXTENSION) $(OBJS) $(OBJS2) jsusfx_$(EXTENSION).zip

test: jsusfx~.$(EXTENSION)
	pd jsusfx_test.pd

debug: jsusfx~.$(EXTENSION)
	gdb --args pd -batch jsusfx_test.pd

dist: default
	zip -j jsusfx_$(EXTENSION).zip ../scripts/liteon/*
	zip -j jsusfx_$(EXTENSION).zip ../README.md ../LICENSE jsusfx~.$(EXTENSION) jsusfx_test.pd gain.jsfx
