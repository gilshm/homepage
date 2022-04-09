---
title: VCD-Based RTL Power Estimation
subtitle: How to evaluate power using Cadence Innovus given VCD files

# Summary for listings and search engines
summary: I'm done with the RTL, I synthesize it with Synopsys Design Compiler (DC), and now I need some power estimations. The thing is that DC estimates power with some sort of global net activity factor, which may not be enough. So, a couple of hours later, I managed to setup a static VCD-based power estimation in Cadence Innovus.


# Link this post with a project
projects: []

# Date published
date: "2020-05-17T00:00:00Z"

# Date updated
lastmod: "2020-05-17T00:00:00Z"

# Is this an unpublished draft?
draft: false

# Show this page in the Featured widget?
featured: false

# Featured image
# Place an image named `featured.jpg/png` in this page's folder and customize its options here.
image:
  caption: 'Image credit: [**Unsplash**](https://unsplash.com/photos/EUsVwEOsblE)'
  focal_point: ""
  placement: 1
  preview_only: false

authors:
- admin

tags:
- Synopsys Design Compiler
- Cadence Innovus
- Power Estimation

categories:
- Tutorial
---

I'm done with the RTL, I synthesize it with Synopsys Design Compiler (DC), and now I need some power estimations.
The thing is that DC estimates power with some sort of global net activity factor, which may not be enough.
So, a couple of hours later, I managed to setup a static VCD-based power estimation in Cadence Innovus.

I will demonstrate the procedure with a simple processing engine (PE) which comprises a multiply-and-accumulate (MAC) unit and some registers.
Basically, an array of such PEs working in tandem is called a systolic array; however, it is beyond the scope of this post.

## SystemVerilog: PE Implementation

The PE is a simple module that multiplies two input values, `a_in` and `b_in`, and accumulates the multiplication result into a partial sum register, `psum`. Inputs also propagate directly to the PE output registers, `a_out` and `b_out`, for other PEs to consume these inputs in the systolic array configuration. For simplicity’s sake, I implemented an unsigned INT8 PE.

```verilog
module pe
   (
      input clk, rst,
      input logic [7:0] a_in,
      input logic [7:0] b_in,
      output logic [7:0] a_out,
      output logic [7:0] b_out,
      output logic [31:0] psum
   );
   always_ff @ (posedge clk) begin
      if (rst) begin
         psum <= '0;
         a_out <= '0;
         b_out <= '0;
      end
      else begin
         psum <= a_in * b_in + psum;
         a_out <= a_in;
         b_out <= b_in;
      end
   end
endmodule
```

Now, the **testbench** (TB).
Not only I will use the TB to validate my implementation, but I will also use it to create the value change dump (VCD) file, which I will use for power estimation.
To make things interesting, the TB is capable of zeroing out the `a_in` and `b_in` inputs in a given probability, so later on we will explore the PE power as a function of its input sparsity.

```verilog
`timescale 1ns/100ps

program automatic pe_tb
  (
    input wire clk, output logic rst,
    output logic [7:0] a_in, output logic [7:0] b_in,
    input logic [7:0] a_out, input logic [7:0] b_out,
    input logic [31:0] psum
  );
  logic [31:0] ref_psum;
  int a_zeros, b_zeros;
    
  initial begin
    if (!$value$plusargs("a+zeros+%d", a_zeros))
      a_zeros = 0; 
    if (!$value$plusargs("b+zeros+%d", b_zeros))
      b_zeros = 0;
            
    rst = 1'b1;
    a_in = '0;
    b_in = '0;
    ref_psum = '0;
        
    @(posedge clk);    // Wait for the positive edge
    rst = 0;        // Disable reset
    #0.2;        // Wait additional 0.2ns to simulate input delay
        
    repeat(1000) begin
      if ($test$plusargs("verbose"))
        $display("@%0dns: a=%0d \t b=%0d \t psum=%0d \t ref_psum[1]=%0d",
          $time, a_in, b_in, psum, ref_psum);
      assert (psum == ref_psum);
            
      a_in = a_zeros < $urandom_range(0, 100) ? $urandom_range(255) : 0;
      b_in = b_zeros < $urandom_range(0, 100) ? $urandom_range(255) : 0;
            
      ref_psum = a_in * b_in + ref_psum;
            
      #2;
    end
  end
endprogram

module pe_tb_top();
  logic clk = 0, rst;
  logic [7:0] a_in;
  logic [7:0] b_in;
  logic [7:0] a_out;
  logic [7:0] b_out;
  logic [31:0] psum;
  always #1 clk++;
    
  pe dut(clk, rst, a_in, b_in, a_out, b_out, psum);
  pe_tb testbench(clk, rst, a_in, b_in, a_out, b_out, psum);
endmodule
```

Let's execute everything with
```
vcs -R -full64 -debug_all -sverilog pe_tb.sv pe.sv \
    +a+zeros+0 +b+zeros+0 +verbose
```
`-R` runs the executable file immediately after VCS links together the executable file,
`-full64` compiles the design in 64-bit mode,
`-sverilog` enables the use of the Verilog language extensions,
and `-debug_all` allows to run the interactive DVE tool and use steps for debug.
The rest of the arguments control the input sparsity and testbench verbosity and are testbench specific.

## Design Compiler: Some Notes
I will not go over the whole process of synthesizing with DC, since many other sources cover that topic — for example, I found the tutorial from [Cornell ECE 5745](https://cornell-ece5745.github.io/ece5745-tut5-asic-tools/) course useful.

One thing though is creating the SDF file.
The standard delay format (SDF) file consists of the "physical" interconnect delays and cell delays of the synthesized design.
We want to consider these delays in the VCD file for the power estimation.
First, I change the naming rules to conform with the SDF file, I found [these](http://www.ee.ncu.edu.tw/~jfli/vlsi21/lecture/dc.pdf) helpful:
```
set bus_inference_style { %s[%d] }
set bus_naming_style { %s[%d] }
set hdlout_internal_busses true
change_names -hierarchy -rule verilog
define_name_rules name_rule -allowed "A-Z a-z 0-9 _" -max_length 255 -type cell
define_name_rules name_rule -allowed "A-Z a-z 0-9 _[]" -max_length 255 -type net
define_name_rules name_rule -map { {"\\*cell\\*""cell"} }
define_name_rules name_rule -case_insensitive
change_names -hierarchy -rules name_rule
```

Then, I extract the SDF and the Verilog netlist from DC using the following commands:
```
write_sdf [PATH]/pe.sdf
write -format verilog -hierarchy -output [PATH]/pe.v
```
Finally, I use VCS again to simulate the design, but this time with the synthesized design and the corresponding SDF file:
```
vcs -R -full64 -debug_all -sverilog -sdf max:pe:pe.sdf \
    pe_tb.sv pe.v ~/pkg-freepdk45-nangate/stdcells.v \
    +a+zeros+0 +b+zeros+0 +verbose +vcs+dumpvars+dump.vcd
```

Notice that: (1) pe.v is loaded and not pe.sv;
(2) the standard cells behavioral Verilog file is loaded;
(3) the SDF file is loaded with `-sdf (min/typ/max):scope_name:sdf_filename`;
and (4) we dump everything into a VCD. Be aware that you might need to write a wrapper to the Verilog module.

> **Important:** NanGate 45nm open cell library has a bug in its `stdcells.v` file.
> Both `SDFF_X` and `SDFF_X2` modules have `ng_xbuf(SE, SEx, 1'b1);` defined.
> However, `SE` is an input port and not an output port.
> The correct statement is, therefore, `ng_xbuf(SE_d, SEx, 1'b1);` [[link]](https://projects.si2.org/openeda.si2.org/tracker/index.php?func=detail&aid=1451&group_id=63&atid=284).

## Innovus: Static VCD-BASED Power Estimation
Fire up Cadence Innovus and reach the point where you have your design “on silicon”.
![vcd-power_innovus_pe](https://raw.githubusercontent.com/gilshm/gilshm.github.io/master/images/vcd-power_innovus_pe.png)

To setup power analysis with a VCD file, run the following commands:
```
read_activity_file -reset
read_activity_file -format VCD -scope pe_tb_top/dut \
                   -start {} -end {} -block {} ./dump.vcd
set_power -reset
set_powerup_analysis -reset
set_dynamic_power_simulation -reset
report_power -outfile ./pe_power.rpt
```

> **Important:** During execution something like this will pop up:
> <br/>`Names in file that matched to design: 2180/2884`
> <br/> this is somewhat expected, since the post-synthesis model is not a one-to-one match with the post-P&R model.
> However, we still have 75% matches, and registers are guaranteed to be in both versions.
> If you get 0 matches, check that `-scope` points to your DUT instance correctly.
>
> In addition, notice that your design clock is the same as the one used in your testbench, otherwise you will see this warning:
> <br/> `** WARN: (VOLTUS_POWR-1784): Existing clock frequency 250MHz is being overwritten with 500.004MHz on clock rooted on net 'clk' from VCD file.`

### Results
I wrote a script that sweeps over both input sparsities.
The results are, therefore, a 3D power surface as a function of the input sparsities.
Run the script through Innovus terminal.
```tcl
set fd [open "results" w+]

for {set a 0} {$a <= 100} {set a [expr {$a + 10}]} {
  for {set b 0} {$b <= 100} {set b [expr {$b + 10}]} {
    vcs -R -full64 -debug_all -sverilog -sdf max:pe:[PATH]/pe.sdf \
    [PATH]/pe_tb.sv [PATH]/pe.v [PATH]/stdcells.v \
    +a+zeros+$a +b+zeros+$b +verbose +vcs+dumpvars+./dump.vcd
    
    read_activity_file -reset
    read_activity_file -format VCD -scope pe_tb_top/dut \
                       -start {} -end {} -block {} ./dump.vcd
    set_power -reset
    set_powerup_analysis -reset
    set_dynamic_power_simulation -reset
    report_power -outfile ./pe_power.rpt
    set pwr [exec cat ./pe_power.rpt | grep "Total Power:" | cut -d: -f2 | xargs]
    puts $fd "$a $b $pwr"
  }
}

close $fd
```

Some post-processing, and voilà!

![](https://raw.githubusercontent.com/gilshm/gilshm.github.io/master/images/vcd-power_results.png)
