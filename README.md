// Part 2 skeleton

module Lab_6b_VGA_Square
	(
		CLOCK_50,						//	On Board 50 MHz
		// Your inputs and outputs here
        KEY,
        SW,
		// The ports below are for the VGA output.  Do not change.
		VGA_CLK,   						//	VGA Clock
		VGA_HS,							//	VGA H_SYNC
		VGA_VS,							//	VGA V_SYNC
		VGA_BLANK_N,						//	VGA BLANK
		VGA_SYNC_N,						//	VGA SYNC
		VGA_R,   						//	VGA Red[9:0]
		VGA_G,	 						//	VGA Green[9:0]
		VGA_B,   						//	VGA Blue[9:0]
		LEDR
	);

	input			CLOCK_50;				//	50 MHz
	input   [9:0]   SW;
	input   [3:0]   KEY;

	// Declare your inputs and outputs here
	// Do not change the following outputs
	output			VGA_CLK;   				//	VGA Clock
	output			VGA_HS;					//	VGA H_SYNC
	output			VGA_VS;					//	VGA V_SYNC
	output			VGA_BLANK_N;				//	VGA BLANK
	output			VGA_SYNC_N;				//	VGA SYNC
	output	[9:0]	VGA_R;   				//	VGA Red[9:0]
	output	[9:0]	VGA_G;	 				//	VGA Green[9:0]
	output	[9:0]	VGA_B;   				//	VGA Blue[9:0]
	output [17:0]LEDR;
	wire resetn;
	assign resetn = KEY[0];
	
	// Create the colour, x, y and writeEn wires that are inputs to the controller.
	wire [2:0] colour;
	wire [7:0] x;
	wire [6:0] y;
	assign LEDR[7:0] = X[7:0];
	assign LEDR[15:8] = Y[6:0];
	wire writeEn;

	// Create an Instance of a VGA controller - there can be only one!
	// Define the number of colours as well as the initial background
	// image file (.MIF) for the controller.
	vga_adapter VGA(
			.resetn(resetn),
			.clock(CLOCK_50),
			.colour(colour[2:0]),
			.x(x[7:0]),
			.y(y[6:0]),
			.plot(writeEn),
			/* Signals for the DAC to drive the monitor. */
			.VGA_R(VGA_R),
			.VGA_G(VGA_G),
			.VGA_B(VGA_B),
			.VGA_HS(VGA_HS),
			.VGA_VS(VGA_VS),
			.VGA_BLANK(VGA_BLANK_N),
			.VGA_SYNC(VGA_SYNC_N),
			.VGA_CLK(VGA_CLK));
		defparam VGA.RESOLUTION = "160x120";
		defparam VGA.MONOCHROME = "FALSE";
		defparam VGA.BITS_PER_COLOUR_CHANNEL = 1;
		defparam VGA.BACKGROUND_IMAGE = "black.mif";
			
	// Put your code here. Your code should produce signals x,y,colour and writeEn/plot
	// for the VGA controller, in addition to any other functionality your design may require.

	assign colour = SW[9:7];
	
	wire [6:0] datain;
	assign datain = SW[6:0];
	
	wire loadreg;
	assign loadreg = ~KEY[3];
	
	wire go;
	assign go = ~KEY[1];
	
	wire [1:0] change;
	wire plot;
	wire [7:0]X;
	wire [6:0]Y;
	// Load the initial values for X and Y 
	 enterXY xy(.data_in(datain),
					.go(loadreg),
					.X(X[7:0]),
					.Y(Y[6:0]),
					.clk(CLOCK_50));
	
	VGA_control vga(	.go(go),
							.resetn(resetn),
							.clock(CLOCK_50), 
							.direction_signal(change[1:0]), 
							.plot(plot));
	assign writeEn = plot;
	
	// recieve drawclock and change value from the FSM
	
	
	reg [2:0] xChange;	
	reg [2:0] yChange;
	
	always@(posedge CLOCK_50)
	begin
		if	(~resetn || change == 2'b10)
			begin
				xChange <= 3'b000;
				yChange <= 3'b000;
			end
		else if ( change == 2'b00)
				xChange <= xChange + 1;
			
		else if(change == 2'b01)
				yChange <= yChange + 1;
		else if(change == 2'b11)
				yChange <= yChange - 1;
						
	end	

		 assign x = X + xChange;
		 assign y = Y - yChange;
	
    // Instansiate datapath
	// datapath d0(...);
	
    // Instansiate FSM control
    // control c0(...);
    
endmodule




module enterXY (data_in, go, X, Y, clk);
	input [7:0] data_in;
	input go;
	input clk;
	output reg [7:0]  X;
	output reg[6:0]  Y;
	 
	reg [5:0] current_state, next_state; 
	
	localparam S_LOAD_A        = 5'd0,
               S_LOAD_A_WAIT   = 5'd1,
               S_LOAD_B        = 5'd2,
               S_LOAD_B_WAIT   = 5'd3;
					//S_LOAD_C			= 5'd4,
				   //S_LOAD_C_WAIT   = 5'd5;

					
	initial X = 8'b00000000;
	initial Y = 7'b0000000;
	
   always@(*)
    begin: state_table 
            case (current_state)
                S_LOAD_A: next_state = go ? S_LOAD_A_WAIT : S_LOAD_A; // Loop in current state until value is input
                S_LOAD_A_WAIT: next_state = go ? S_LOAD_A_WAIT : S_LOAD_B; // Loop in current state until go signal goes low
                S_LOAD_B: next_state = go ? S_LOAD_B_WAIT : S_LOAD_B; // Loop in current state until value is input
                S_LOAD_B_WAIT: next_state = go ? S_LOAD_B_WAIT : S_LOAD_A; // Loop in current state until go signal goes low
                //S_LOAD_C: next_state = go ? S_LOAD_C_WAIT : S_LOAD_C; // Loop in current state until value is input
                //S_LOAD_C_WAIT: next_state = go ? S_LOAD_C_WAIT : S_LOAD_A; // Loop in current state until go signal goes low
				endcase
	 end

   always @(*)
   begin: stuff

		case(current_state)
			S_LOAD_A: begin
					X <= data_in;
					end
			S_LOAD_B: begin
					Y <= data_in;
					end
		endcase
	end

   always@(posedge clk)
    begin: state_FFs
            current_state <= next_state;
    end // state_FFS


endmodule 


module VGA_control(
	input go,
	input resetn,
	input clock,
	output reg [1:0]direction_signal,
	output reg plot
	);

	reg [4:0] current_state;
	reg [4:0] next_state;
	initial current_state = 5'b10000;

	localparam  S0 	= 5'b00000,
				S1 	= 5'b00001,
				S2 	= 5'b00010,
				S3 	= 5'b00011,
				S4 	= 5'b00100,
				S5 	= 5'b00101,
				S6 	= 5'b00110,
				S7 	= 5'b00111,
				S8 	= 5'b01000,
				S9 	= 5'b01001,
				S10	= 5'b01010,
				S11	= 5'b01011,
				S12	= 5'b01100,
				S13	= 5'b01101,
				S14	= 5'b01110,
				S15	= 5'b01111,
				S16	= 5'b10000;

	always@(*)
    begin: state_table 
         case (current_state)
			S0 : next_state = S1;
			S1 : next_state = S2;
			S2 : next_state = S3;
			S3 : next_state = S4;
			S4 : next_state = S5;
			S5 : next_state = S6;
			S6 : next_state = S7;
			S7 : next_state = S8;
			S8 : next_state = S9;
			S9 : next_state = S10;
			S10 : next_state = S11;
			S11 : next_state = S12;
			S12 : next_state = S13;
			S13 : next_state = S14;
			S14 : next_state = S15;
			S15 : next_state = S16;
			S16 : next_state = go ? S0 : S16;
			default: next_state = S16;
		endcase
	end // end state_table

	always@(posedge clock)
	begin: enable_signals
		case(current_state)
			S0:
			begin
			direction_signal <= 2'b01;
			plot <= 1'b1;
			end
			S1:
			begin
			direction_signal <= 2'b01;
			end
			S2:
			begin
			direction_signal <= 2'b01;
			end
			S3:
			begin
			direction_signal <= 2'b00;
			end
			S4:
			begin
			direction_signal <= 2'b11;
			end
			S5:
			begin
			direction_signal <= 2'b11;
			end
			S6:
			begin
			direction_signal <= 2'b11;
			end
			S7:
			begin
			direction_signal <= 2'b00;
			end
			S8:
			begin
			direction_signal <= 2'b01;
			end
			S9:
			begin
			direction_signal <= 2'b01;
			end
			S10:
			begin
			direction_signal <= 2'b01;
			end
			S11:
			begin
			direction_signal <= 2'b00;
			end
			S12:
			begin
			direction_signal <= 2'b11;
			end
			S13:
			begin
			direction_signal <= 2'b11;
			end
			S14:
			begin
			direction_signal <= 2'b11;
			end
			S15:
			begin
			direction_signal <= 2'b00;
			end
			S16:
			begin
			direction_signal <= 2'b10;
			plot <= 1'b0;
			end
		endcase
	end

	always@(posedge clock)
    begin: state_FFs
        if(~resetn)
            current_state <= S16;
        else
            current_state <= next_state;
    end

endmodule 
