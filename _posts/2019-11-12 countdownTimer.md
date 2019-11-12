---
layout:     post
title:      Countdown Timer Design
subtitle:   Keyang | FPGA Digital System Design
date:       2019-11-12
author:     Keyang
header-img: img/post-bg-huawei-01.jpg
catalog: 	 true
tags:
    - FPGA
    - RTL Design
    - verilog
    
    
---
The aim of this assignment is to design an electronic countdown timer. The report will explain how it works why it is designed this way.

RTL design ( RTL diagram attached as appendix1)
When design the RTL diagram of “countdown Timer” ,I decide to divide it as basically two sub blocks. One of the block is to implement control logic and the other one is to process counter down function.(FSM diagram attached as appendix 2).
 I am in charge of the design and Verification. So here I am going to introduce the whole of design and verification flows.

##Controller module
 ![avatar](/img/controller.png)
 
Like the FSM diagram shown, I separate the entire flow into 4 stages: IDLE, loading, running, and alert state.
When the start button been pressed firstly, machine convert state from IDLE to loading state, the counter start to load the initial value and read inputs from switches.
As operator realise the button, the FPGA detect the negative edge of signal, and controller is turn into running stage. The counter start to counter down and reload while each digit overflow until the BCD outputs all approach to zero.
When all digitals display as zeros ,state machine goes to alert stage ,hold value and set alert to high active. By the way, the Rst signal can reset the state into idle from any states.

 '''module controller (clk,start,zero,load,run,alert,rst);//this clk  need aligin 
input start,zero,rst,clk;
output reg load,run,alert;
reg [3:0] nextstate,currentstate;


parameter  idle = 4'b0001,
		loading = 4'b0010,
		running = 4'b0100,
		alertstate = 4'b1000;
		
//descirbe the sequential state transition		 
always@(posedge clk or posedge rst)
	if(rst)
		currentstate <= idle;
	else
		currentstate <= nextstate;
		
		
always@(*)
	begin
	case(currentstate)'''
	
	'''idle:begin
		if(start==1'b1)
		nextstate <= loading;
		else
		nextstate<=idle;
		end
		
	loading:begin
		if(start==1'b0)
		nextstate<=running;
		else
		nextstate<=loading;
		end
		
	running:begin
	if(zero==1'b1)
	nextstate<=alertstate;
	else
	nextstate<=running;
	end	
	
	alertstate:begin
	if(zero==1'b0)
	nextstate <=idle;
	else
	nextstate<=alertstate;
	end
	
	default: nextstate <= 4'b0001;
	endcase
	end
	
	//third block describe the FSM ouput in differnt state
always@(currentstate or alert)
	begin
			case(currentstate)
				idle: begin
				    load <= 1'b0;
				    run <= 1'b0;
					alert<=1'b0;
				    end
				loading:begin
				        load <= 1'b1;
						run <= 1'b0;
						alert<=1'b0;
				        end
				running:begin 
				        load <= 1'b0;
						run <= 1'b1;
						alert<=1'b0;
				    end
				alertstate:begin
				       load <= 1'b0;
						run <= 1'b0;
						alert<=1'b1;
				    end
				
				default begin 
					load <= 1'b0;
				    run <= 1'b0;
					alert<=1'b0;
				        end
			endcase	
    end
endmodule'''

##Counter down module

When design the countdown module , firstly we generate the 1ms clock using a flip-flop.
 Then I found the rest of structures share a high similarity. So it would be simple to make a general block and instantiate when we need it in different forms.
 
![avatar](/img/countdown.png)





The block design show above ,which  use a flip-flop and a multiplexer to control the behaviours of the counter.
The multiplexer use 4 branches to realise the functions :
1.	Mins 1 when counter is counting down and receive the overflow signal from front flip-flop.
2.	Hold the value when counter is counting down but without the overflow form front flip-flop.
3.	Reload the number when the flip-flop decrease to zero and overflow.
4.	Load the value of “ sw” signal when load signal is set active high.

The overall block shown as below:
 
##Verification & Debug
  
When design the testbench, it is better to set a shorter interval of clock pulse ,so that we could  see the result in a proper time scale. Unlike other assignment , here we have multiple modules and better to have different testbench for different modules.
After go through all the testbench’s results ,there are some problems we met.

1.	The biggest one is that the initial issue with the overflow signal. 
Here we can’t just take the condition that output of each flip- flop equals zero as the judgement of overflow symbol. Because the initial output values of flip-flop after reset are all zero. So it need some combinational logic to avoid the unexpected overflow.
2.	Second problem we found from testbench result it that we may the first output period is one clock cycle less other.
  The reason is because ,in my design, we decide to rest the first flip-flop , to generate 1ms pulse, so that  we can start to counter when we release the start button. However the reset move is asynchronous. So there will always be 1 clock cycle less than other display period.
 Given that 1 clock cycle is relatively small compared with 500000 cycle ( = 1ms), so it is acceptable.

3.	 Here we use 9 second as se input just for example，and I have also tested others value like（9mins.）and convert of state machine which work successfully shown as below. 

Testbench diagram
 
 
##Integration
After successfully complete the main function blocks ,rest need to be done is integration the display module which was designed and verified last lab  with the top module of countdown timer .Then add the constraint file and test on hardware.
As we can see the picture below the CPLD resource we used main focus on X0Y1 andX1Y1 area. And we could also see the I/O resource we used. 
