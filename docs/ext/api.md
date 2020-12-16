# Luna C/C++ API

(under development)

## Linking against the Luna library

Include the `luna.h` header in your code

```
#include "luna.h"
```

and point the linker to `libluna.so` (or on Mac, `libluna.dylib`). As
needed, pass the compiler the appropriate `-I` and `-L` flags. 

Here is an example: f

```
include Makefile.inc 

OUTPUT = example

LIB = -lluna -lhpdf -lfftw3 -lsamplerate

SRC = example.cpp

HDR = 

OBJ = example.o

all : $(OUTPUT)

$(OUTPUT) :
        $(CXX) -o $(OUTPUT) $(OBJ) $(LDFLAGS) $(LIB)

ifeq ($(ARCH),MAC)
        install_name_tool -change libluna.dylib $(LUNA)/libluna.dylib $(OUTPUT)
endif


$(OBJ) : $(HDR)

.cpp.o : 
        $(CXX) $(CXXFLAGS) -c $*.cpp

.SUFFIXES : .cpp .o $(SUFFIXES)

$(OUTPUT) : $(OBJ) 

FORCE:

clean:
        rm -f *.o *~ ${OUTPUT}
```
where `Makefile.inc` (which sets the user-specific values) is
```
##
## Target architecture (MAC or LINUX)
##

ARCH = MAC

##
## path to luna base (for example)
##

LUNA = ${HOME}/src/luna-base/

##
## luna dependencies (as absolute paths), if they are not accessible system-wide
## (nb, for Mac, /usr/local has to be added explicitly)
##

LUNADEP = ${HOME}/src/luna-depends/
#LUNADEP = /usr/local/

##
## main flags  
##

CXX = g++
CXXFLAGS = -O2 -std=c++0x -I. -I$(LUNA) -I$(LUNADEP)/include -I/usr/local/include
LDFLAGS = -L. -L$(LUNA) -L$(LUNADEP)/lib
```






## Primary functions

### EDFs



### Evaluating commands

### Output

### Annotations


## Example 


```
#include "luna.h"

extern globals global;
extern writer_t writer;

int main(int argc , char ** argv ) 
{

  // required set-up

  global.init_defs();
  
  // this specifies some globals (in defs/defs.h)  

  std::cout << "luna " << globals::version << " " << globals::date << "\n";

  // otherwise run silently, i.e. no logging to console

  global.api(); 

  // We start by creating a new EDF object

  edf_t edf; 

  // attach file   

  bool okay = edf.attach( "/home/joe/data/file1.edf" , "id001" ) ;

  // check it worked

  if ( ! okay ) Helper::halt( "failed to attach file" );

  // to catch the output of evaluated command, use 
  // a retval_t struct...

  retval_t ret;

  // and attach it to the writer

  writer.use_retval( &ret );

  // we can then specify any series of Luna commands

  cmd_t cmd( "EPOCH & PSD epoch" );

  // and evaluate them against a given EDF

  okay = cmd.eval( edf ); 

  // after checking that it worked 

  if ( ! okay ) Helper::halt( "failed to run command" ) ;
  
  // we can look at the output (here dumped to std::cout)

  ret.dump();

  // all done  

  std::exit(0);

}
```