# 8x3_Priority_encoder
## Method 1 
### [RTL]
```bash
module priority_encoder(y0,y1,y2,IDLE,d0,d1,d2,d3,d4,d5,d6,d7,en);
input d0,d1,d2,d3,d4,d5,d6,d7,en;
output y0,y1,y2,IDLE;
wire h0,h1,h2,h3,h4,h5,h6,h7y_0,y_1,Y_2;
priority_ckt c1(.IDLE(IDLE),.h0(h0),.h1(h1),.h2(h2),.h3(h3),.h4(h4),.h5(h5),.h6(h6),.h7(h7),.d0(d0),.d1(d1),.d2(d2),.d3(d3),.d4(d4),.d5(d5),.d6(d6),.d7(d7));
binary_decoder b1(.y0(y_0),.y1(y_1),.y2(y_2),.d0(h0),.d1(h1),.d2(h2),.d3(h3),.d4(h4),.d5(h5),.d6(h6),.d7(h7));
assign y0   = en ? y_0 : 1'b0;
assign y1   = en ? y_1 : 1'b0;
assign y2   = en ? y_2 : 1'b0;
endmodule

module priority_ckt(IDLE,h0,h1,h2,h3,h4,h5,h6,h7,d0,d1,d2,d3,d4,d5,d6,d7);
input d0,d1,d2,d3,d4,d5,d6,d7;
//output y0,y1,y2,y3,y4,y5,y6,y7;
output IDLE,h0,h1,h2,h3,h4,h5,h6,h7;
assign h7 = d7;
assign h6 = ~d7 & d6;
assign h5 = ~d7 & ~d6 & d5;
assign h4 = ~d7 & ~d6 & ~d5 & d4;
assign h3 = ~d7 & ~d6 & ~d5 & ~d4 & d3;
assign h2 = ~d7 & ~d6 & ~d5 & ~d4 & ~d3 & d2;
assign h1 = ~d7 & ~d6 & ~d5 & ~d4 & ~d3 & ~d2 & d1;
assign h0 = ~d7 & ~d6 & ~d5 & ~d4 & ~d3 & ~d2 & ~d1 & d0;
assign IDLE =  ~d7 & ~d6 & ~d5 & ~d4 & ~d3 & ~d2 & ~d1 & ~d0;

endmodule
module binary_decoder (y0,y1,y2,d0,d1,d2,d3,d4,d5,d6,d7);
input d0,d1,d2,d3,d4,d5,d6,d7;
wire h0,h1,h2,h3,h4,h5,h6,h7;
output y0,y1,y2;
assign y0=(d7|d5|d3|d1);
assign y1=(d7|d6|d3|d2);
assign y2=(d7|d6|d5|d4);
//assign valid=(d7|d6|d5|d4|d3|d2|d1|d0);
endmodule
```
### [Test bench]
```bash
module priority_encoder_tb;
	// Inputs
	reg d0;
	reg d1;
	reg d2;
	reg d3;
	reg d4;
	reg d5;
	reg d6;
	reg d7;
	reg en;
	// Outputs
	wire y0;
	wire y1;
	wire y2;
	wire IDLE;
	//wire valid;
   integer i,j;
	// Instantiate the Unit Under Test (UUT)
	priority_encoder uut (
		.y0(y0), 
		.y1(y1), 
		.y2(y2), 
      .IDLE(IDLE),		
		.d0(d0), 
		.d1(d1), 
		.d2(d2), 
		.d3(d3), 
		.d4(d4), 
		.d5(d5), 
		.d6(d6), 
		.d7(d7),
		.en(en)
	);
	initial begin
		d0 = 0;
		d1 = 0;
		d2 = 0;
		d3 = 0;
		d4 = 0;
		d5 = 0;
		d6 = 0;
		d7 = 0;
		en = 0;
		#100; 
		  for(i=0;i<256;i=i+1)
		  begin 
		  {d7,d6,d5,d4,d3,d2,d1,d0}=i;
		  for(j=0;j<2;j=j+1)
		  begin 
		  en=j;
		  #10;// Add stimulus here
        end
		  end
	end
	initial begin 
	#10000 $finish;
	end
	initial begin 
	$monitor("|d7=%b,d6=%b,d5=%b,d4=%b,d3=%b,d2=%b,d1=%b,d0=%b,en=%b,y0=%b y1=%b,y2=%b,IDLE=%b",d7,d6,d5,d4,d3,d2,d1,d0,en,y0,y1,y2,IDLE);
	end
endmodule
```
## Method 2
### [RTL]
```bash
//behavioural model
module priority_encoder(
    input  d0, d1, d2, d3, d4, d5, d6, d7,en,
    output reg y0, y1, y2,
    output reg IDLE);
always @(*) begin
    IDLE = 0;
    y0 = 0;
    y1 = 0;
    y2 = 0;
	 if(en==0)
	 begin
	  {y2,y1,y0}=3'b111;
	 end
	  else
	     begin
    //  from highest to lowest priority
    casez ({d7,d6,d5,d4,d3,d2,d1,d0}) //casez
        8'b1???????: begin y2=1; y1=1; y0=1; IDLE=0; 
		  end 
        8'b01??????: begin y2=1; y1=1; y0=0; IDLE=0; 
		  end
        8'b001?????: begin y2=1; y1=0; y0=1; IDLE=0; 
		  end 
        8'b0001????: begin y2=1; y1=0; y0=0; IDLE=0; 
		  end 
        8'b00001???: begin y2=0; y1=1; y0=1; IDLE=0; 
		  end 
        8'b000001??: begin y2=0; y1=1; y0=0; IDLE=0; 
		  end 
        8'b0000001?: begin y2=0; y1=0; y0=1; IDLE=0; 
		  end 
        8'b00000001: begin y2=0; y1=0; y0=0; IDLE=0; 
		  end 
        default: begin
            y2=0; y1=0; y0=0; IDLE=1;
		 end
    endcase
	 end
end
endmodule
```
### [Test bench]
```bash
//structural model
module priority_encoder_tb;
	// Inputs
	reg d0;
	reg d1;
	reg d2;
	reg d3;
	reg d4;
	reg d5;
	reg d6;
	reg d7;
	reg en;
	// Outputs
	wire y0;
	wire y1;
	wire y2;
	wire y_0;
	wire y_1;
	wire y_2;
	wire IDLE;
	//wire valid;
   integer i,j;
	// Instantiate the Unit Under Test (UUT)
	priority_encoder uut (
		.y0(y_0), 
		.y1(y_1), 
		.y2(y_2), 
    .IDLE(IDLE),		
		.d0(d0), 
		.d1(d1), 
		.d2(d2), 
		.d3(d3), 
		.d4(d4), 
		.d5(d5), 
		.d6(d6), 
		.d7(d7),
		.en(en)
	);

	initial begin
		d0 = 0;
		d1 = 0;
		d2 = 0;
		d3 = 0;
		d4 = 0;
		d5 = 0;
		d6 = 0;
		d7 = 0;
		en = 0;
		#100;
		  for(i=0;i<256;i=i+1)
		  begin 
		  {d7,d6,d5,d4,d3,d2,d1,d0}=i;
		  for(j=0;j<2;j=j+1)
		  begin 
		  en=j;
		  #10;// Add stimulus here
        end
		  end  
	end
	assign y0   = en ? y_0 : 1'b0;
   assign y1   = en ? y_1 : 1'b0;
   assign y2   = en ? y_2 : 1'b0;
	initial begin 
	#10000 $finish;
	end
	initial begin 
	$monitor("|d7=%b,d6=%b,d5=%b,d4=%b,d3=%b,d2=%b,d1=%b,d0=%b,en=%b,y0=%b y1=%b,y2=%b,IDLE=%b",d7,d6,d5,d4,d3,d2,d1,d0,en,y0,y1,y2,IDLE);
	end 
endmodule
```
## Method 3
### [RTL]
```bash
//behavioural model
module priority_encoder(
    input  d0, d1, d2, d3, d4, d5, d6, d7,en,
    output reg y0, y1, y2,
    output reg IDLE);
always @(*) begin
    IDLE = 0;
    y0 = 0;
    y1 = 0;
    y2 = 0;
	 if(en==0)
	 begin
	  {y2,y1,y0}=3'b111;
	 end
	  else
	     begin
    //  from highest to lowest priority
    casex ({d7,d6,d5,d4,d3,d2,d1,d0}) //casex
        8'b1xxxxxxx: begin y2=1; y1=1; y0=1; IDLE=0; 
		  end 
        8'b01xxxxxx: begin y2=1; y1=1; y0=0; IDLE=0; 
		  end
        8'b001xxxxx: begin y2=1; y1=0; y0=1; IDLE=0; 
		  end 
        8'b0001xxxx: begin y2=1; y1=0; y0=0; IDLE=0; 
		  end 
        8'b00001xxx: begin y2=0; y1=1; y0=1; IDLE=0; 
		  end 
        8'b000001xx: begin y2=0; y1=1; y0=0; IDLE=0; 
		  end 
        8'b0000001x: begin y2=0; y1=0; y0=1; IDLE=0; 
		  end 
        8'b00000001: begin y2=0; y1=0; y0=0; IDLE=0; 
		  end 
        default: begin
            y2=0; y1=0; y0=0; IDLE=1;
		 end
    endcase
	 end
end
endmodule
```
## Method 4
### [RTL]
```bash
module pri_enc_8to3 (
    input  [7:0] din,
    output reg [2:0] y,
    output reg v
);
always @(*) begin
    case (1'b1)
        din[7]: begin y = 3'b111; v = 1; end
        din[6]: begin y = 3'b110; v = 1; end
        din[5]: begin y = 3'b101; v = 1; end
        din[4]: begin y = 3'b100; v = 1; end
        din[3]: begin y = 3'b011; v = 1; end
        din[2]: begin y = 3'b010; v = 1; end
        din[1]: begin y = 3'b001; v = 1; end
        din[0]: begin y = 3'b000; v = 1; end
        default: begin y = 3'b000; v = 0; end
    endcase
end
endmodule
```

