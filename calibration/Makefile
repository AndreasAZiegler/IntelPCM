all: calibration

#CC=g++ -Wall
CC=/usr/local/bin/intel/bin/icpc -Wall
OPT= -g -O3 -no-vec -no-ipo -no-ip -mkl=sequential

UNAME:=$(shell uname)

ifeq ($(UNAME), Linux)
LIBDIR= ../
LIB= -lpthread -lrt -lmeasuringcore
endif
ifeq ($(UNAME), Darwin)
LIB= -lpthread -lPcmMsr
endif
ifeq ($(UNAME), FreeBSD)
LIB= -lpthread
endif

calibration: ../cpucounters.o ../msr.o ../pci.o ../measuring_core.o
	 $(CC) $(CPPFLAGS) $(OPT) ../msr.o ../pci.o ../cpucounters.o ../measuring_core.o calibration.cpp -o calibration -static -L $(LIBDIR) $(LIB) 

clean:
	rm -rf calibration
