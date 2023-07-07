#########################################################################################
# pre-process OCELOT into a single blackbox file
#########################################################################################
base_dir=$(abspath ../../../../../..)
vsrc_dir=$(abspath .)
ocelot_dir=$(vsrc_dir)/rvv
hardfloat_dir=$(vsrc_dir)/HardFloat/source

# name of output pre-processed verilog file
PREPROC_VERILOG = OcelotCoreBlackbox.preprocessed.sv

.PHONY: default $(PREPROC_VERILOG)
default: $(PREPROC_VERILOG)

#########################################################################################
# includes and vsrcs
#########################################################################################
OCELOT_PKGS =  \
  	$(ocelot_dir)/autogen_riscv_imabfv.v
#  	$(ocelot_dir)/briscv_defines.h \
#  	$(ocelot_dir)/tt_briscv_pkg.vh \
#  	$(ocelot_dir)/autogen_defines.h 

OCELOT_VSRCS = \
  	$(ocelot_dir)/vfp_pipeline.sv \
  	$(ocelot_dir)/tt_id.sv \
  	$(ocelot_dir)/tt_ex.sv \
  	$(ocelot_dir)/tt_lq.sv \
  	$(ocelot_dir)/tt_mem.sv \
  	$(ocelot_dir)/tt_vec.sv \
  	$(ocelot_dir)/tt_vec_iadd.sv \
  	$(ocelot_dir)/tt_vec_idp.sv \
  	$(ocelot_dir)/tt_vec_imul.sv \
  	$(ocelot_dir)/tt_vec_mul_dp.sv \
  	$(ocelot_dir)/tt_vec_regfile.sv \
  	$(ocelot_dir)/tt_vfp_unit.sv \
  	$(ocelot_dir)/tt_vfp_ex_unit.sv \
  	$(ocelot_dir)/tt_vfp_lane.sv \
  	$(ocelot_dir)/tt_vfp_encoder.sv \
  	$(ocelot_dir)/tt_vfp_encoder_lane.sv \
  	$(ocelot_dir)/tt_vfp_fma.sv \
  	$(ocelot_dir)/tt_vfp_red.sv \
  	$(ocelot_dir)/tt_popcnt.sv \
  	$(ocelot_dir)/tt_pipe_stage.sv \
  	$(ocelot_dir)/tt_rts_rtr_pipe_stage.sv \
  	$(ocelot_dir)/tt_cam_buffer.sv \
  	$(ocelot_dir)/tt_skid_buffer.sv \
  	$(ocelot_dir)/tt_store_fifo.sv \
  	$(ocelot_dir)/tt_ffs.sv \
  	$(ocelot_dir)/tt_ascii_instrn_decode.sv \
  	$(ocelot_dir)/tt_compare.sv \
  	$(ocelot_dir)/tt_decoded_mux.sv \
  	$(ocelot_dir)/tt_decoder.sv \
  	$(ocelot_dir)/tt_reshape.sv 

HARDFLOAT_VSRCS = \
	$(hardfloat_dir)/RISCV/HardFloat_specialize.v  \
  	$(hardfloat_dir)/HardFloat_primitives.v \
  	$(hardfloat_dir)/HardFloat_rawFN.v \
  	$(hardfloat_dir)/addRecFN.v \
  	$(hardfloat_dir)/compareRecFN.v \
  	$(hardfloat_dir)/divSqrtRecFN_small.v \
  	$(hardfloat_dir)/fNToRecFN.v \
  	$(hardfloat_dir)/iNToRecFN.v \
  	$(hardfloat_dir)/isSigNaNRecFN.v \
  	$(hardfloat_dir)/mulAddRecFN.v \
  	$(hardfloat_dir)/mulRecFN.v \
  	$(hardfloat_dir)/recFNToFN.v \
  	$(hardfloat_dir)/recFNToIN.v \
  	$(hardfloat_dir)/recFNToRecFN.v                

ALL_VSRCS = $(OCELOT_PKGS) $(OCELOT_VSRCS) $(OCELOT_WRAPPER) $(HARDFLOAT_VSRCS)

#########################################################################################
# pre-process using verilator
#########################################################################################

#lookup_dirs = $(shell find -L $(ocelot_dir) -name target -prune -o -type d -print 2> /dev/null | grep '.*/\($(1)\)$$')
#INC_DIR_NAMES ?= include inc
#INC_DIRS ?= $(foreach dir_name,$(INC_DIR_NAMES),$(call lookup_dirs,$(dir_name)))
INC_DIRS ?= $(ocelot_dir) $(hardfloat_dir) $(hardfloat_dir)/RISCV

# these flags are specific to Chipyard
EXTRA_PREPROC_DEFINES ?=
PREPROC_DEFINES ?= \
	WT_DCACHE \
	DISABLE_TRACER \
	SRAM_NO_INIT \
	VERILATOR \
	$(EXTRA_PREPROC_DEFINES)

PREPROC_SCRIPT = $(base_dir)/scripts/insert-includes.py

$(PREPROC_VERILOG): $(ALL_VSRCS)
	mkdir -p $(dir $(PREPROC_VERILOG))
	$(foreach def,$(PREPROC_DEFINES),echo "\`define $(def)" >> def.v; )
	$(foreach def,$(PREPROC_DEFINES),echo "\`undef $(def)" >> undef.v; )
	cat def.v $(ALL_VSRCS) undef.v > combined.sv
	sed -i '/l15.tmp.h/d' combined.sv
	sed -i '/define.tmp.h/d' combined.sv
	$(PREPROC_SCRIPT) combined.sv $@ $(INC_DIRS)
	rm -rf combined.sv def.v undef.v

clean:
	rm -rf $(PREPROC_VERILOG)
