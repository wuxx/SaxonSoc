#TOPLEVEL = Ice40up5kbevnNoXip
TOPLEVEL = Ice40up5kbevn
ROOT = ../../..
VERILOG = $(ROOT)/hardware/netlist/${TOPLEVEL}.v
#BINARY  = $(ROOT)/hardware/netlist/Ice40up5kbevn.v_toplevel_system_cpu_cpu_RegFilePlugin_regFile.bin
BINARY  = $(ROOT)/hardware/netlist/Ice40up5kbevn.v_toplevel_system_cpu_logic_cpu_RegFilePlugin_regFile.bin

#BINARY  = $(ROOT)/hardware/netlist/Ice40up5kbevnNoXip.v_toplevel_system_cpu_logic_cpu_RegFilePlugin_regFile.bin

DEVICE = up5k
PACKAGE = sg48
PCF = mapping.pcf
CONSTRAINT = constraint.py


include ../flow/icestorm.mk

generate :
	#(cd ../../..; sbt "runMain saxon.board.Ice40up5kbevnNoXip.Ice40up5kbevnNoXip")
	(cd ../../..; sbt "runMain saxon.board.Ice40up5kbevn.Ice40up5kbevn")
