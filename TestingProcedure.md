# Verilog code testing procedure (Graduate)
###### tags: `CAID` `Synthesis`
:::info
## Presim
* Go to the location where place `TPA.v` and `testbench.v`  
    * ![](https://i.imgur.com/Ras0RA6.png)

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
   
:::
:::success
### Result: 
![](https://i.imgur.com/XqfyYWJ.png)
:::
:::info
## Postsim (note: make sure search_path is correct)

*    **Design compiler** : preparing those files
        1. synopsys_dc.setup (Remember to change the Library's path)
        ```
            set company "CIC"
            set designer "Student"
            set search_path      "/home/nfs_cad/lib/CBDK_IC_Contest_v2.1/SynopsysDC/db/  ../sram_512x16/ $search_path"
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

        2. TPA.sdc(constraint file)
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
        3. dc_syn.tcl(synthesis script)
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
:::


:::danger
### Error when running `ncverilog -f postsim.f`
*    Remember to change file name into **`.synopsys_dc.setup`** 
*    Maybe the clock period too short in **`*.sdc`** and **`*_tb.v`**. We can increase the cycle period 
        *    ***.sdc** 
        ![](https://i.imgur.com/rnhd485.png)
        *    ***_tb.v** ![](https://i.imgur.com/Mo9xsar.png)

:::
:::info
*    After synthesis, using `report_timing` and `report_constraint` to watch some infomation
:::
:::success
* Result: 
![](https://i.imgur.com/ggPzl4g.png)
:::
