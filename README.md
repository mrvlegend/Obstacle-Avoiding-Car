#FULL ADDER WITHOUT STRUCTURE
Verilog:
module prladder(a,b,c0,s,c4);
input [3:0]a,b;
input c0;
output c4;
output [3:0]s;
wire c1,c2,c3;

assign {c1,s[0]} =a[0]+b[0]+c0;
assign {c2,s[1]} =a[1]+b[1]+c1;
assign {c3,s[2]} =a[2]+b[2]+c2;
assign {c4,s[3]} =a[3]+b[3]+c3;
endmodule

Testbench:
module prladder_test;
reg [3:0]a;
reg [3:0]b;
reg c0;
wire [3:0]s;
wire c4;
prladder uut(.a(a),.b(b),.c0(c0),.s(s),.c4(c4));
initial begin
a=4'b0011; b=4'b0011; c0=1'b0;#100;
a=4'b1011; b=4'b0011; c0=1'b1;#200;
a=4'b1111; b=4'b1111; c0=1'b1;#400;
end 
endmodule

constraints:
set_input_delay -max 0.8 [get_port "a"]
set_input_delay -max 0.8 [get_port "b"]
set_input_delay -max 0.8 [get_port "c0"]
set_output_delay -max 0.8 [get_port "s"]
set_output_delay -max 0.8 [get_port "c4"]

RCscript:
set_db init_lib_search_path /home/install/FOUNDRY/digital/90nm/dig/lib
set_db hdl_search_path /root/Desktop/vlsi/program1a
set_db library slow.lib
read_hdl adder.v
elaborate
read_sdc constraints_sdc.sdc
set_db syn_generic_effort medium
syn_generic
set_db syn_map_effort medium
syn_map
set_db syn_opt_effort medium
syn_opt
write_hdl > adder_netlist.v
write_sdc > adder_block.sdc
report_area > adder_area.rep
report_gates > adder_gate.rep
report_power > adder_power.rep
report_timing -unconstrained > adder_timing.rep
gui_show

#FULL ADDER WITH STRUCTURE
Verilog:
module four_bit_adder(a,b,c0,s,c4);
input [3:0]a,b;
input c0;
output c4;
output [3:0]s;
wire c1,c2,c3;
full_adder fa0 (a[0],b[0],c0,s[0],c1);
full_adder fa1 (a[1],b[1],c1,s[1],c2);
full_adder fa2 (a[2],b[2],c2,s[2],c3);
full_adder fa3 (a[3],b[3],c3,s[3],c4);
endmodule
module full_adder(a,b,cin,s,cout);
input a,b,cin;
output s,cout;
assign s=a^b^cin;
assign cout=((a&b)|(cin&(a^b)));
endmodule

Testbench:
module struct_addertest;
reg [3:0]a;
reg [3:0]b;
reg c0;
wire [3:0]s;
wire c4;
four_bit_adder uut(.a(a),.b(b),.c0(c0),.s(s),.c4(c4));
initial begin
a=4'b0011; b=4'b0011; c0=1'b0;#100;
a=4'b1011; b=4'b0011; c0=1'b1;#200;
a=4'b1111; b=4'b1111; c0=1'b1;#400;
end 
endmodule

Constraints:
set_input_delay -max 0.8 [get_port "a"]
set_input_delay -max 0.8 [get_port "b"]
set_input_delay -max 0.8 [get_port "c0"]
set_output_delay -max 0.8 [get_port "s"]
set_output_delay -max 0.8 [get_port "c4"]

RCscript:
set_db init_lib_search_path /home/install/FOUNDRY/digital/90nm/dig/lib
set_db hdl_search_path /root/Desktop/vlsi/program1b
set_db library slow.lib
read_hdl structadder.v
elaborate
read_sdc constraints_sdc.sdc
set_db syn_generic_effort medium
syn_generic
set_db syn_map_effort medium
syn_map
set_db syn_opt_effort medium
syn_opt
write_hdl > structadder_netlist.v
write_sdc > structadder_block.sdc
report_area > structadder_area.rep
report_gates > structadder_gate.rep
report_power > structadder_power.rep
report_timing -unconstrained > structadder_timing.rep
gui_show

#MULTIPLIER:
Verilog:
module shift_add(
input clk,
input load,
input reset,
input [3:0]A,
input [3:0]B,
output reg [7:0] P);
 reg [3:0] AC=4'b0, Q=4'b0, M=4'b0, Count=3'd4;
always @ (posedge clk)
begin
if (reset==1)
begin
 AC = 4'b0000;
 Q = 4'b0000;
 M = 4'b0000;
 Count = 3'd4;
 P= 8'b00000000;
end
else if (load==1)
begin
 M = A; 
 Q = B; 
end
else if((Q[0]==1) && (Count >3'd0))
begin
 AC=AC+M;
 P ={AC, Q} >> 1;
 AC =P[7:4];
 Q =P[3:0];
 Count=Count-1;
end
else if((Q[0]==0) && (Count>3'd0))
begin
P={AC,Q};
P = P>>1;
AC=P[7:4];
Q=P[3:0];
Count=Count-1;
end
else
begin
Count=3'b0;
end
P={AC,Q};
end
endmodule

testbench:
module tb_shift_add_multiplier;
 reg clk, reset, load;
  reg [3:0] A,B;
 wire [7:0] P;
 shift_add uut (.clk(clk),.reset(reset),.load(load),.A(A),.B(B),.P(P));
always
#10
clk=~clk;
initial
 begin
clk=0;
load=0;
reset=1'b1;
A=4'b1001;
B=4'b1100;
#20; load=1;
reset=1'b0;
#40; load =0;
#150
 $finish;
end
endmodule

Constraints:
create_clock -name clk -period 10 -waveform {0 0.5} [get_ports "clock"]
set_clock_transition -rise 0.1 [get_clocks "clk"]
set_clock_transition -fall 0.1 [get_clocks "clk"]
set_clock_uncertainty 0.01 [get_ports "clk"]
set_input_delay -max 1.0 -clock clk [all_inputs]
set_output_delay -max 1.0 -clock clk [all_outputs]

RCscript:
set_db init_lib_search_path /home/install/FOUNDRY/digital/90nm/dig/lib
set_db hdl_search_path /root/Desktop/vlsi/Multiplier
set_db library slow.lib
read_hdl multiplier.v
elaborate
read_sdc constraints.sdc
set_db syn_generic_effort medium
syn_generic
set_db syn_map_effort medium
syn_map
set_db syn_opt_effort medium
syn_opt
write_hdl > mul_netlist.v
write_sdc > mul_block.sdc
report_area > mul_area.rep
report_gates > mul_gate.rep
report_power > mul_power.rep
report_timing -unconstrained > mulr_timing.rep
report_qor> mul_qor.rep
gui_show

#32 BIT ALU
Verilog:(case)
module alu_32bit_case(y, a, b, f);
input[31:0]a;
input [31:0]b;
input [2:0]f;
output reg[31:0]y;
always@(*)
begin
case(f)
3'b000:y=a&b;
3'b001:y=a|b;
3'b010:y=~(a&b);
3'b011:y=~(a|b);
3'b100:y=a+b;
3'b101:y=a-b;
3'b110:y=a*b;
3'b111:y=~a;
default: 
 y=32'bx;
endcase
end
endmodule

Verilog:(if case)
module alu_32bit_case(y, a, b, f);
input[31:0]a;
input [31:0]b;
input [2:0]f;
output reg[31:0]y;
always@(*)
begin
if(f==3'b000)
  y=a&b;
else if(f==3'b001)
  y=a|b;
else if(f==3'b010)
  y=~(a&b);
else if(f==3'b011)
  y=~(a|b);
else if(f==3'b100)
  y=a+b;
else if(f==3'b101)
  y=a-b;
else if(f==3'b110)
  y=a*b;
else if(f==3'b111)
  y=~a;
else 
  y=32'bx;
end
endmodule

Testbench:
module alu_tb_case;
reg[31:0]a;
reg [31:0]b;
reg [2:0] f;
wire[31:0]y;
alu_32bit_case test2(.y(y), .a(a), .b(b), .f(f));
initial
begin
a=32'h00000000;
b=32'h0000003F;
#10 f=3'b000;
#10 f=3'b001;
#10 f=3'b010;
#10 f=3'b011;
#10 f=3'b100;
#10 f=3'b101;
#10 f=3'b110;
#10 f=3'b111;
end
initial
#100
$finish;
endmodule

Constraints:
set_input_delay -max 0.8 [get_port "a"]
set_input_delay -max 0.8 [get_port "b"]
set_input_delay -max 0.8 [get_port "f"]
set_output_delay -max 0.8 [get_port "y"]

RCscript:
set_db init_lib_search_path /home/install/FOUNDRY/digital/90nm/dig/lib
set_db hdl_search_path /root/Desktop/vlsi/alu1
set_db library slow.lib
read_hdl alu.v
elaborate
read_sdc /root/Desktop/vlsi/alu1/constraints.sdc
set_db syn_generic_effort medium
syn_generic
set_db syn_map_effort medium
syn_map
set_db syn_opt_effort medium
syn_opt
write_hdl > alu_netlist.v
write_sdc > alu_block.sdc
report_area > alu_area.rep
report_gates > alu_gate.rep
report_power > alu_power.rep
report_timing -unconstrained > alu_timing.rep
gui_show

#SR FLIPFLOP
Verilog:
module srff(clk,sr,q,qb); 
input clk;
input[1:0]sr;
 output q,qb;
 reg q,qb;
initial 
    begin
      q=1'b1; 
      qb=~q; 
    end
       always@ (posedge clk)
       begin
         if(clk==1)
         begin
        case(sr) 
           2'b00:q=q;
           2'b01:q=1'b0; 
           2'b10:q=1'b1;
           2'b11:q=1'bz;
        endcase 
          qb=~q; 
      end 
end
endmodule

Testbench:
module rsff_v;
reg clk;
reg[1:0]sr;
srff uut(.clk(clk),.sr(sr),.q(q),.qb(qb));
initial
  begin
   clk = 0; sr=00;#100;
   clk = 1; sr=00;#100;
   clk = 0; sr=01;#100;
   clk = 1; sr=01;#100;
   clk = 0; sr=10;#100;
   clk = 1; sr=10;#100;
   clk = 0; sr=11;#100;
   clk = 1; sr=11;#100;
end
endmodule

#JK FLIPFLOP
verilog:
module jkff(Clk,jk,q,qb);
input Clk;
input[1:0]jk;
output q,qb;
reg q,qb;
initial
    begin
       q=1'b1;
       qb=~q;
    end
       always@ (posedge Clk)
       begin
         if(Clk==1)
         begin
        case(jk)
           2'b00:q=q;
           2'b01:q=1'b0;
           2'b10:q=1'b1;
           2'b11:q=~q;
        endcase
          qb=~q;
       end
end
endmodule

Testbench:
module jkff_v;
reg Clk ;
reg[1:0]jk;
jkff uut(.Clk(Clk),.jk(jk),.q(q),.qb(qb) );
initial
   begin
    Clk = 0; jk=00;#100;
    Clk = 1; jk=00;#100;
    Clk = 0; jk=01;#100;
    Clk = 1; jk=01;#100;
    Clk = 0; jk=10;#100;
    Clk = 1; jk=10;#100;
    Clk = 0; jk=11;#100;
    Clk = 1; jk=11;#100;
end
endmodule

#D FLIPFLOP
Verilog:
module d_ff(d,clk,q,qb);   #for T use t instead of d
input d,clk;
output q,qb;
reg q,qb;
initial
   begin
     q=1'b1;
   end
     always@(posedge clk)
      begin
        if (d==1'b1)
         q=d;               #for T ff q=~t
        else if(d==1'b0)
         q=d;
         qb= ~q;
      end
endmodule

Testbench:
module dflip;
reg d, clk; // inputs
wire q, qb; // outputs
d_ff uut(.d(d), .clk(clk), .q(q), .qb(qb) );
initial 
   begin
    d = 0; clk = 0; #50;// initialization ofinputs
    d = 0; clk = 1; #50;
    d = 1; clk = 0; #50;
    d = 1; clk = 1; #50;
   end
endmodule

Constraints:
create_clock -name Clk -period 2-waveform {0 1} [get_ports "Clk"]
set_clock_transition -rise 0.1[get_clocks "Clk"]
set_clock_transition fall 0.1 [get_clocks"Clk"]
set_clock_uncertainty 0.01 [get_ports"Clk"]
set_input_delay -max 1.0 [get_ports"D"] -clock [get_clocks "Clk"]#for jk  "jk",for sr "sr"
set_output_delay -max 1.0 [get_ports"q"] -clock [get_clocks "Clk"]
set_output_delay -max 1.0 [get_ports"qb"] -clock [get_clocks Clk"]

RCscript:
set_db init_lib_search_path /home/install/FOUNDRY/digital/90nm/dig/lib
set_db hdl_search_path /root/Desktop/vlsi/dff
set_db library slow.lib
read_hdl df.v    #verilog filename
elaborate
read_sdc /root/Desktop/vlsi/dff/constraints.sdc
set_db syn_generic_effort medium
syn_generic
set_db syn_map_effort medium
syn_map
set_db syn_opt_effort medium
syn_opt

write_hdl > D_netlist.v   #D or JK, SR
write_sdc > D_block.sdc
report_area > D_area.rep
report_gates > D_gate.rep
report_power > D_power.rep
report_timing -unconstrained > D_timing.rep
gui_show

#COUNTER
Verilog:
module modn_ctr
 # (parameter N = 6,parameter WIDTH = 4)
 ( input clk,
 input rstn,
 output reg[WIDTH-1:0] out);
 always @ (posedge clk or posedge rstn) begin
 if (rstn) begin
 out <= 0;
 end
 else begin
 if (out == N-1)
 out <= 0;
 else
 out <= out + 1;
 end
 end
endmodule 

Testbench:
module tb;
 parameter N = 6;
 parameter WIDTH = 4;
 reg clk;
 reg rstn;
 wire [WIDTH-1:0] out;
 modn_ctr uut ( .clk(clk), .rstn(rstn), .out(out));
 always #10 
clk = ~clk;
 initial begin
   clk=0;
   rstn=0;
 #10;
rstn=1;
#35;
rstn=0;
#260;
rstn=1;
#40;
$finish;
end
endmodule 

Constraints:
create_clock -name Clk -period 1 -waveform {0 0.5} [get_ports "Clk"]
set_clock_transition -rise 0.1 [get_clocks "Clk"]
set_clock_transition fall 0.1 [get_clocks "Clk"]
set_clock_uncertainty 0.01 [get_ports "Clk"]
set_input_delay -max 1.0 [get_ports "rstn"]-clock [get_clocks "Clk"]
set_output_delay -max 1.0 [get_ports "out"]-clock [get_clocks "Clk"] 

RCscript:
set_db init_lib_search_path /home/install/FOUNDRY/digital/90nm/dig/lib
set_db hdl_search_path /root/Desktop/vlsi/counter
set_db library slow.lib
read_hdl counter.v
elaborate
read_sdc /root/Desktop/vlsi/counter/constraints.sdc
set_db syn_generic_effort medium 
syn_generic
set_db syn_map_effort medium 
syn_map
set_db syn_opt_effort medium 
syn_opt

write_hdl > modn_netlist.v
write_sdc > modn_block.sdc
report_area > modn_area.rep
report_gates > modn_gate.rep
report_power > modn_power.rep
report_timing -unconstrained > modn_timing.rep
gui_show