# Tutorial for Graduate-Level IC Contest
**Git:** https://github.com/jayhua267/HackMD/blob/Graduate/TestingProcedure.md

**HackMD:**
https://hackmd.io/Tt053rm0RDenzGcEb27rpg
###### tags: `CAID` `Synthesis`
# Preliminary Contest

## Prework
`cd grad_prelim/ICC_2018/2018_TPA`

## Presim
* Go to the location where place `TPA.v` and `testbench.v`  

![](https://i.imgur.com/Ras0RA6.png)

* Make a new `.f` file using instruction `vim simulate.f`
* In `.f` file write:
```
testbench.v
TPA.v
+nc64bit
+access+r
+define+FSDB        
```
* Input these instruction to initial environment
    *    **`tcsh`**
    *    **`menu`** then **`2`**
    *    **`menu`** then **`3`** 
    *    ![](https://i.imgur.com/2iwwo4N.png)
    *    **`ncverilog -f simulate.f`**
   

### Result: 
![](https://i.imgur.com/XqfyYWJ.png)

## Postsim (note: make sure search_path is correct)

*    **Design compiler** : preparation before synthesis
        1. **Set up file**: `synopsys_dc.setup` (Remember to change the search path)
        
        ![](https://i.imgur.com/7YUzVJA.png)
        ```
        set company "CIC"
        set designer "Student"
        ******************* !!!! Change here !!!!***********************
        set search_path      "/home/nfs_cad/lib/CBDK_IC_Contest_v2.1/SynopsysDC/db/  ../sram_512x16/ $search_path"
        **********************************************************
        set target_library   "slow.db"
        set link_library     "* $target_library dw_foundation.sldb"
        set symbol_library   "generic.sdb"
        set synthetic_library "dw_foundation.sldb"

        set hdlin_translate_off_skip_text "TRUE"
        set edifout_netlist_only "TRUE"
        set verilogout_no_tri true

        set hdlin_enable_presto_for_vhdl "TRUE"
        set sh_enable_line_editing true
        set sh_line_editing_mode emacs
        history keep 100
        alias h history

        set bus_inference_style {%s[%d]}
        set bus_naming_style {%s[%d]}
        set hdlout_internal_busses true
        define_name_rules name_rule -allowed {a-z A-Z 0-9 _} -max_length 255 -type cell
        define_name_rules name_rule -allowed {a-z A-Z 0-9 _[]} -max_length 255 -type net
        define_name_rules name_rule -map {{"\\*cell\\*" "cell"}}
        ```

        2. **Constraint File**: `TPA.sdc`
        ```
        set cycle  10.0        ;#clock period defined by designer
        create_clock -period $cycle [get_ports  clk]
        set_dont_touch_network      [get_clocks clk]
        set_clock_uncertainty  0.1  [get_clocks clk]
        set_clock_latency      0.5  [get_clocks clk]
        set_input_delay  5      -clock clk [remove_from_collection [all_inputs] [get_ports clk]]
        set_output_delay 0.5    -clock clk [all_outputs] 
        set_load         0.1     [all_outputs]
        set_drive        1     [all_inputs]
        set_operating_conditions  -max slow  
        set_wire_load_model -name tsmc13_wl10 -library slow               
        set_max_fanout 20 [all_inputs]
        ```
        3. **Synthesis Script**: `dc_syn.tcl`
        ```
        #Read All Files
        set TOP TPA 
        read_file -format verilog  ${TOP}.v
        current_design ${TOP}
        link

        #Setting Clock Constraints
        source -echo -verbose ${TOP}.sdc
        check_design
        set high_fanout_net_threshold 0
        uniquify
        set_fix_multiple_port_nets -all -buffer_constants [get_designs *]

        check_design
        #Synthesis all design
        #compile -map_effort high -area_effort high
        #compile -map_effort high -area_effort high -inc
        compile



        remove_unconnected_ports -blast_buses [get_cells -hierarchical *]

        set bus_inference_style {%s[%d]}
        set bus_naming_style {%s[%d]}
        set hdlout_internal_busses true
        change_names -hierarchy -rule verilog
        define_name_rules name_rule -allowed {a-z A-Z 0-9 _}   -max_length 255 -type cell
        define_name_rules name_rule -allowed {a-z A-Z 0-9 _[]} -max_length 255 -type net
        define_name_rules name_rule -map {{"\\*cell\\*" "cell"}}
        define_name_rules name_rule -case_insensitive
        change_names -hierarchy -rules name_rule

        write -format ddc     -hierarchy -output "${TOP}_syn.ddc"
        write_sdf -version 1.0  ${TOP}_syn.sdf
        write -format verilog -hierarchy -output ${TOP}_syn.v
        report_area > area.log
        report_timing > timing.log
        report_qor   >  ${TOP}_syn.qor
        ```
        
*  Input these instruction to execute postsim file
    *  Rename to synopsys_dc.setup to .synopsys_dc.setup
        * `mv synopsys_dc.setup .synopsys_dc.setup`
    *  **`menu`** then  **`6`** 
    *  **`dc_shell -f dc_syn.tcl`**
    *  ![](https://i.imgur.com/S6JAcL8.png)
    *  Making a new file .f ( `post.f` ) 
        ```
        testbench.v
        TPA_syn.v
        -v tsmc13_neg.v
        +nc64bit
        +access+r
        +define+SDF
        ```
    *    **`menu`** then **`2`**
    *    **`menu`** then **`3`** 
    *    ![](https://i.imgur.com/2iwwo4N.png)
    *    **`ncverilog -f post.f`**



### Error when running `ncverilog -f postsim.f`
*    Remember to change file name into **`.synopsys_dc.setup`** 
*    Maybe the clock period too short in **`*.sdc`** and **`*_tb.v`**. We can increase the cycle period 
        *    ***.sdc** 
        
        ![](https://i.imgur.com/rnhd485.png)
        *    ***_tb.v** 
        
        ![](https://i.imgur.com/Mo9xsar.png)

After synthesis, using `report_timing` and `report_constraint` to watch some infomation


Result: 

![](https://i.imgur.com/ggPzl4g.png)


---
## FINAL

## 2019 IC Design Contest Cell-Based IC Design Category for Graduate Level
###  Prework
*    `cd grad_final/ICC_2019/icc2019cb/`
*    In `testfixture.v` file, change 
```
define SDFFILE     "./IOTDF_syn.sdf"
```
into
```
`ifdef SYN
`define SDFFILE     "./IOTDF_syn.sdf"     //Modify your sdf file name
`elsif PR
`define SDFFILE     "./IOTDF_pr.sdf"     //Modify your sdf file name
`endif
```
###  Presim (RTL level)
*    In `runall_rtl` file, add `+nc64bit` at the end of each line:
```
ncverilog testfixture.v IOTDF.v +access+r -clean +define+F1 +nc64bit
ncverilog testfixture.v IOTDF.v +access+r -clean +define+F2 +nc64bit
ncverilog testfixture.v IOTDF.v +access+r -clean +define+F3 +nc64bit
ncverilog testfixture.v IOTDF.v +access+r -clean +define+F4 +nc64bit
ncverilog testfixture.v IOTDF.v +access+r -clean +define+F5 +nc64bit
ncverilog testfixture.v IOTDF.v +access+r -clean +define+F6 +nc64bit
ncverilog testfixture.v IOTDF.v +access+r -clean +define+F7 +nc64bit
```
Executing that file 
*    **`sh runall_rtl`**

==Result after each test==

![](https://i.imgur.com/PPVFoJg.png)
### Postsim (Synthesis and Gate level)
#### Synthesis (Refer to Synthesis step in Preliminary)
#### Gate level(Postsim)
*    In `runall_syn` file, add `+nc64bit +SYN` at each line:
```
ncverilog testfixture.v IOTDF_syn.v -v tsmc13_neg.v +access+r +nc64bit -clean +define+SDF+F1+SYN
ncverilog testfixture.v IOTDF_syn.v -v tsmc13_neg.v +access+r +nc64bit -clean +define+SDF+F2+SYN
ncverilog testfixture.v IOTDF_syn.v -v tsmc13_neg.v +access+r +nc64bit -clean +define+SDF+F3+SYN
ncverilog testfixture.v IOTDF_syn.v -v tsmc13_neg.v +access+r +nc64bit -clean +define+SDF+F4+SYN
ncverilog testfixture.v IOTDF_syn.v -v tsmc13_neg.v +access+r +nc64bit -clean +define+SDF+F5+SYN
ncverilog testfixture.v IOTDF_syn.v -v tsmc13_neg.v +access+r +nc64bit -clean +define+SDF+F6+SYN
ncverilog testfixture.v IOTDF_syn.v -v tsmc13_neg.v +access+r +nc64bit -clean +define+SDF+F7+SYN

```
Executing that file 
*    `sh runall_syn`

==Result after each test==
![](https://i.imgur.com/CLauvjl.png)

### Place and Route 
* Before P&R, you need to prepare the following files using Cadence Innovus
    *    `IOTDF.io`
    *    `IOTDF_pr.gds`
    *    `IOTDF_pr.sdf`
    *    `IOTDF_pr.spef`
    *    `IOTDF_pr.v`
* Put these files into your directory
* In `runall_pr` file, add `+nc64bit +PR` at each line:
```
ncverilog  +ncmaxdelays  testfixture.v IOTDF_pr.v -v tsmc13_neg.v +nc64bit +access+r -clean +define+SDF+F1+PR
ncverilog  +ncmaxdelays  testfixture.v IOTDF_pr.v -v tsmc13_neg.v +nc64bit +access+r -clean +define+SDF+F2+PR
ncverilog  +ncmaxdelays  testfixture.v IOTDF_pr.v -v tsmc13_neg.v +nc64bit +access+r -clean +define+SDF+F3+PR
ncverilog  +ncmaxdelays  testfixture.v IOTDF_pr.v -v tsmc13_neg.v +nc64bit +access+r -clean +define+SDF+F4+PR
ncverilog  +ncmaxdelays  testfixture.v IOTDF_pr.v -v tsmc13_neg.v +nc64bit +access+r -clean +define+SDF+F5+PR
ncverilog  +ncmaxdelays  testfixture.v IOTDF_pr.v -v tsmc13_neg.v +nc64bit +access+r -clean +define+SDF+F6+PR
ncverilog  +ncmaxdelays  testfixture.v IOTDF_pr.v -v tsmc13_neg.v +nc64bit +access+r -clean +define+SDF+F7+PR

```
Executing that file 
*    `sh runall_pr`

==Result after each test==
![](https://i.imgur.com/Ct3ymjT.png)

### Primetime
#### Prework
Preparation before executing primetime
*    `pt_script.tcl`
```
#PrimeTime Script
set power_enable_analysis TRUE
set power_analysis_mode time_based

read_file -format verilog  ./IOTDF_pr.v
current_design IOTDF
link

source ./IOTDF_APR.sdc
read_parasitics -format SPEF -verbose  ./IOTDF_pr.spef


## Measure  power
#report_switching_activity -list_not_annotated -show_pin

read_vcd  -strip_path test/u_IOTDF  ./IOTDF_F1.fsdb
update_power
report_power 
report_power > F1_7.power

read_vcd  -strip_path test/u_IOTDF  ./IOTDF_F2.fsdb
update_power
report_power
report_power >> F1_7.power

read_vcd  -strip_path test/u_IOTDF  ./IOTDF_F3.fsdb
update_power
report_power
report_power >> F1_7.power

read_vcd  -strip_path test/u_IOTDF  ./IOTDF_F4.fsdb
update_power
report_power
report_power >> F1_7.power

read_vcd  -strip_path test/u_IOTDF  ./IOTDF_F5.fsdb
update_power
report_power
report_power >> F1_7.power

read_vcd  -strip_path test/u_IOTDF  ./IOTDF_F6.fsdb
update_power
report_power
report_power >> F1_7.power

read_vcd  -strip_path test/u_IOTDF  ./IOTDF_F7.fsdb
update_power
report_power
report_power >> F1_7.power

```
*    `.synopsys_pt.setup`
```
set company "CIC"
set designer "Student"
set search_path       "./  /usr/cad/lib/CBDK_IC_Contest_v2.1/SynopsysDC/db/  $search_path"
set target_library    "slow.db                    \
                      "
set link_library      "* $target_library"

set hdlin_translate_off_skip_text "TRUE"
set edifout_netlist_only "TRUE"
set verilogout_no_tri true

set hdlin_enable_presto_for_vhdl "TRUE"
set sh_enable_line_editing true
set sh_line_editing_mode emacs
history keep 100
alias h history


```
#### Executing
Executing instruction 
*    `pt_shell -f pt_sctipt.tcl`

==Result after each test==
![](https://i.imgur.com/FDm4ZSW.png)



---

### Makefile Version
*    RTL level
        *    `make rtl tb=1` 
        *    `make rtl tb=2`
        *    `make rtl tb=3`
        *    `make rtl tb=4`
        *    `make rtl tb=5`
        *    `make rtl tb=6`
        *    `make rtl tb=7`
*    Synthesis `make synthesize`
*    Gate level
        *    `make gate tb=1`
        *    `make gate tb=2`
        *    `make gate tb=3`
        *    `make gate tb=4`
        *    `make gate tb=5`
        *    `make gate tb=6`
        *    `make gate tb=7`
*    Place and Route
        *    `make pr tb=1`
        *    `make pr tb=2`
        *    `make pr tb=3`
        *    `make pr tb=4`
        *    `make pr tb=5`
        *    `make pr tb=6`
        *    `make pr tb=7`
*    primetime `make pt_time`


