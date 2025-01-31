program = openmc
prefix = /usr/local

source = $(wildcard *.F90)
objects = $(source:.F90=.o)
xml_lib = -Lxml/lib -lxml

#===============================================================================
# User Options
#===============================================================================

COMPILER = ibm
DEBUG    = no
PROFILE  = no
OPTIMIZE = yes
MPI      = no
OPENMP   = yes
HDF5     = no
PETSC    = no
MACHINE  = bluegeneq

#===============================================================================
# External Library Paths
#===============================================================================

MPI_DIR   = /opt/mpich/3.0.4-$(COMPILER)
HDF5_DIR  = /opt/hdf5/1.8.11-$(COMPILER)
PHDF5_DIR = /opt/phdf5/1.8.11-$(COMPILER)
PETSC_DIR = /opt/petsc/3.4.2-$(COMPILER)

#===============================================================================
# Add git SHA-1 hash
#===============================================================================

GIT_SHA1 = $(shell git log -1 2>/dev/null | head -n 1 | awk '{print $$2}')

#===============================================================================
# GNU Fortran compiler options
#===============================================================================

ifeq ($(COMPILER),gnu)
  F90 = gfortran
  F90FLAGS := -cpp -std=f2008 -fbacktrace
  LDFLAGS =

  # Debugging
  ifeq ($(DEBUG),yes)
    F90FLAGS += -g -Wall -pedantic -fbounds-check \
                -ffpe-trap=invalid,overflow,underflow
    LDFLAGS  += -g
  endif

  # Profiling
  ifeq ($(PROFILE),yes)
    F90FLAGS += -pg
    LDFLAGS  += -pg
  endif

  # Optimization
  ifeq ($(OPTIMIZE),yes)
    F90FLAGS += -O3
  endif
endif

#===============================================================================
# Intel Fortran compiler options
#===============================================================================

ifeq ($(COMPILER),intel)
  F90 = ifort
  F90FLAGS := -fpp -std08 -assume byterecl -traceback
  LDFLAGS =

  # Debugging
  ifeq ($(DEBUG),yes)
    F90FLAGS += -g -warn -ftrapuv -fp-stack-check -check all -fpe0
    LDFLAGS  += -g
  endif

  # Profiling
  ifeq ($(PROFILE),yes)
    F90FLAGS += -pg
    LDFLAGS  += -pg
  endif

  # Optimization
  ifeq ($(OPTIMIZE),yes)
    F90FLAGS += -O3
  endif
endif

#===============================================================================
# PGI compiler options
#===============================================================================

ifeq ($(COMPILER),pgi)
  F90 = pgf90
  F90FLAGS := -Mpreprocess -DNO_F2008 -Minform=inform -traceback
  LDFLAGS =

  # Debugging
  ifeq ($(DEBUG),yes)
    F90FLAGS += -g -Mbounds -Mchkptr -Mchkstk
    LDFLAGS  += -g
  endif

  # Profiling
  ifeq ($(PROFILE),yes)
    F90FLAGS += -pg
    LDFLAGS  += -pg
  endif

  # Optimization
  ifeq ($(OPTIMIZE),yes)
    F90FLAGS += -fast -Mipa
  endif
endif

#===============================================================================
# IBM XL compiler options
#===============================================================================

ifeq ($(COMPILER),ibm)
  F90 = xlf2003
  F90FLAGS := -WF,-DNO_F2008 -O2

  # Debugging
  ifeq ($(DEBUG),yes)
    F90FLAGS += -g -C -qflag=i:i -u
    LDFLAGS  += -g
  endif

  # Profiling
  ifeq ($(PROFILE),yes)
    F90FLAGS += -p
    LDFLAGS  += -p
  endif

  # Optimization
  ifeq ($(OPTIMIZE),yes)
    F90FLAGS += -O3
  endif
endif

#===============================================================================
# Cray compiler options
#===============================================================================

ifeq ($(COMPILER),cray)
  F90 = ftn
  F90FLAGS := -e Z -m 0

  # Debugging
  ifeq ($(DEBUG),yes)
    F90FLAGS += -g -R abcnsp -O0
    LDFLAGS  += -g
  endif
endif

#===============================================================================
# Setup External Libraries
#===============================================================================

# MPI for distributed-memory parallelism and HDF5 for I/O

ifeq ($(MPI),yes)
  ifeq ($(HDF5),yes)
    F90 = $(PHDF5_DIR)/bin/h5pfc
    F90FLAGS += -DHDF5
  else
    F90 = $(MPI_DIR)/bin/mpif90
  endif
  F90FLAGS += -DMPI
else
  ifeq ($(HDF5),yes)
    F90 = $(HDF5_DIR)/bin/h5fc
    F90FLAGS += -DHDF5
  endif
endif

# OpenMP for shared-memory parallelism

ifeq ($(OPENMP),yes)
  ifeq ($(COMPILER),intel)
    F90FLAGS += -openmp
    LDFLAGS += -openmp
  endif

  ifeq ($(COMPILER),gnu)
    F90FLAGS += -fopenmp
    LDFLAGS += -fopenmp
  endif

  ifeq ($(COMPILER),ibm)
    F90FLAGS += -qsmp=omp
    LDFLAGS += -qsmp=omp
  endif
endif

# PETSC for CMFD functionality

ifeq ($(PETSC),yes)
  # Check to make sure MPI is set
  ifneq ($(MPI),yes)
    $(error MPI must be enabled to compile with PETSC!)
  endif

  # Set up PETSc environment
  include $(PETSC_DIR)/conf/petscvariables
  F90FLAGS += -I$(PETSC_DIR)/include -DPETSC
  LDFLAGS += $(PETSC_LIB)
endif

#===============================================================================
# Machine-specific setup
#===============================================================================

# IBM Blue Gene/P ANL supercomputer

ifeq ($(MACHINE),bluegenep)
  F90 = /bgsys/drivers/ppcfloor/comm/xl/bin/mpixlf2003
  F90FLAGS = -WF,-DNO_F2008,-DMPI,-DRESTRICTED_ASSOCIATED_BUG -O3
  LDFLAGS = -lmpich.cnkf90
endif

# Cray XK6 ORNL Titan supercomputer

ifeq ($(MACHINE),crayxk6)
  F90 = ftn
  F90FLAGS += -DMPI
endif

# IBM Blue Gene/Q ANL supercomputer
ifeq ($(MACHINE),bluegeneq)

	# Unset compiler and flags
  F90 =
  F90FLAGS =
  LDFLAGS =

	# OpenMP for shared-memory parallelism
  ifeq ($(OPENMP), yes)
	  # Use thread-safe compiler wrapper
    F90       = mpixlf2008_r
    F90FLAGS += -qsmp=omp
    LDFLAGS  += -qsmp=omp
  else
	  # Use default compiler wrapper
    F90 = mpixlf2008
  endif

	# MPI for distributed-memory parallelism 
	# (HDF5 for parallel I/O has been untested on BG/Q)
  ifeq ($(MPI), yes)
    F90FLAGS += -WF,-DMPI,-DRESTRICTED_ASSOCIATED_BUG
  else
    F90FLAGS += -WF,-DRESTRICTED_ASSOCIATED_BUG
  endif

  # Debugging
  ifeq ($(DEBUG),yes)
    F90FLAGS += -g
    LDFLAGS  += -g
  endif

  # Profiling
  ifeq ($(PROFILE),yes)
    F90FLAGS += -p
    LDFLAGS  += -p
  endif

  # Optimization
  ifeq ($(OPTIMIZE),yes)
    F90FLAGS += -O5
  endif

endif

#===============================================================================
# Targets
#===============================================================================

all: xml $(program)
xml:
	cd xml; make MACHINE=$(MACHINE) F90=$(F90) F90FLAGS="$(F90FLAGS)" LDFLAGS="$(LDFLAGS)"
$(program): $(objects)
	$(F90) $(objects) $(xml_lib) $(LDFLAGS) -o $@
install:
	@install -D $(program) $(DESTDIR)$(prefix)/bin/$(program)
	@install -D utils/statepoint_cmp.py $(DESTDIR)$(prefix)/bin/statepoint_cmp
	@install -D utils/statepoint_histogram.py $(DESTDIR)$(prefix)/bin/statepoint_histogram
	@install -D utils/statepoint_meshplot.py $(DESTDIR)$(prefix)/bin/statepoint_meshplot
	@install -D ../man/man1/openmc.1 $(DESTDIR)$(prefix)/share/man/man1/openmc.1
	@install -D ../LICENSE $(DESTDIR)$(prefix)/share/doc/$(program)/copyright
uninstall:
	@rm $(DESTDIR)$(prefix)/bin/$(program)
	@rm $(DESTDIR)$(prefix)/bin/statepoint_cmp
	@rm $(DESTDIR)$(prefix)/bin/statepoint_histogram
	@rm $(DESTDIR)$(prefix)/bin/statepoint_meshplot
	@rm $(DESTDIR)$(prefix)/share/man/man1/openmc.1
	@rm $(DESTDIR)$(prefix)/share/doc/$(program)/copyright
distclean: clean
	cd xml; make clean
clean:
	@rm -f *.o *.mod $(program)
neat:
	@rm -f *.o *.mod

#===============================================================================
# Rules
#===============================================================================

.SUFFIXES: .F90 .o
.PHONY: all xml install uninstall clean neat distclean 

%.o: %.F90
	$(F90) $(F90FLAGS) -DGIT_SHA1="\"$(GIT_SHA1)\"" -Ixml/include -c $<

#===============================================================================
# Dependencies
#===============================================================================

include DEPENDENCIES
