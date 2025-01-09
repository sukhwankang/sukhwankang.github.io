---
layout: post
title: UVM Practice (1)":""/"/:/ Simple 8-bit Adder Verification Using UVM
date: 2025-01-09
description: UVM Practice
tags: UVM SystemVerilog Verification
categories: sample-posts
featured: true
---

In this implementation, I created a simple UVM testbench for verifying an 8-bit Adder. The goal is to demonstrate the process of building a simple UVM-based verification environment. The simulation was performed using the popular online platform [EDA Playground](https://edaplayground.com).


#### Simple Implementation

- 8-bit Adder and Interface
- Class env
- Testbench

### Block diagram

<div class="row" style="text-align: center;">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" max-width="300px" max-height="auto" path="assets/blogimg/uvmpractice/1/uvmpractice1_bd.PNG" class="img-fluid rounded z-depth-1" zoomable=true caption="Block Diagram" %}
    </div>
</div>

### 8-bit Adder and Interface

```verilog
module ADDER(
  input bit              clk,
  input logic [7:0]      a0,
  input logic [7:0]      b0,
  input logic           enable,
  output reg [8:0] sum0
);

  always @ (posedge clk)
    begin
      if (enable)
        sum0 <= a0 + b0;
      else
        sum0 <= 8'hx;
    end
endmodule: ADDER

interface dut_if(
  input bit clk,
  input [7:0] a,
  input [7:0] b,
  input       en,
  input [8:0] sum
);

  clocking cb @(posedge clk);
    output    a;
    output    b;
    output    en;
    input     sum;
  endclocking
endinterface: dut_if

bind ADDER dut_if dut_if1(
  .clk(clk),
  .a(a0),
  .b(b0),
  .en(enable),
  .sum(sum0)
);
```
### Class env

```verilog
class env extends uvm_env;
  virtual dut_if dut_vif;

  function new(string name, uvm_component parent = null);
    super.new(name, parent);
  endfunction
  
  function void connect_phase(uvm_phase phase);
    `uvm_info("INFO", "Started connect phase.", UVM_HIGH);
    uvm_config_db#(virtual dut_if)::get(this, "", "dut_if", dut_vif);
    `uvm_info("INFO", "Finished connect phase.", UVM_HIGH);
  endfunction: connect_phase

  task run_phase(uvm_phase phase);
    phase.raise_objection(this);
    `uvm_info("INFO", "Started run phase.", UVM_HIGH);    
    begin
      int a = 1, b = 2;
      parameter int LOOP_CNT = 7;
      `uvm_info("INFO", "Start ADDER Check", UVM_HIGH);
      #200;
      for(int i=0;i<LOOP_CNT;i++) begin
        @(dut_vif.cb);
        dut_vif.cb.a <= a;
        dut_vif.cb.b <= b;
        dut_vif.cb.en <= 1'b1;
        repeat(2) @(dut_vif.cb);
        `uvm_info("RESULT", $sformatf("#%0d a=%0d b=%0d Sum=%0d", i, a, b, dut_vif.cb.sum), UVM_LOW);
        assert(dut_vif.cb.sum == (a + b))
         else `uvm_error("ERROR", $sformatf("Error! Result=%0d, not %0d", dut_vif.cb.sum, a+b));
        a = a*2; b = b*2;
      end
    end
    dut_vif.cb.en <= 1'b0;
    #200;
    `uvm_info("INFO", "Finished ADDER Check", UVM_HIGH);      
    phase.drop_objection(this);
  endtask: run_phase
endclass
```


### Testbench

```verilog
import uvm_pkg::*;
`include "uvm_macros.svh"

module top;
  bit clk;
  env env_h;
  ADDER dut(.clk (clk));

  initial begin
    env_h = new("env");
    uvm_config_db#(virtual dut_if)::set(null, env_h.get_full_name(), "dut_if", dut.dut_if1);
    run_test();
  end
  
  initial begin
    $dumpvars(0, top);
    clk = 0;
    forever #50 clk = ~clk;
  end
endmodule
```

### Option
+UVM_VERBOSITY=UVM_HIGH +access+r

### Result Waveform

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/blogimg/uvmpractice/1/uvmpractice1_1.PNG" class="img-fluid rounded z-depth-1" zoomable=true caption="Result waveform" %}
    </div>
</div>

### UVM Log

<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/blogimg/uvmpractice/1/uvmpractice1_2.PNG" class="img-fluid rounded z-depth-1" zoomable=true caption="UVM log"%}
    </div>
</div>

### Future Work
- Enhance the testbench by first incorporating UVM sequences, drivers, and monitors to achieve a more structured verification.
- Extend it for additional functionality and scalability.
