# Smart Adaptive Streetlight System Using Vehicle Speed–Based Brightness Control


## Abstract
This project presents a smart streetlight system that automatically adjusts LED brightness based on vehicle speed to improve energy efficiency and road safety. A speed detector, FSM controller, and PWM generator work together to control illumination in real time. Implemented in Verilog HDL, the system provides brighter light for fast-moving vehicles and dim lighting during low or no traffic, ensuring optimal performance with reduced power consumption.

## Introduction
Smart streetlighting systems are essential for modern intelligent transportation and highway infrastructure. Traditional streetlights operate at fixed brightness levels without responding to real-time traffic flow, resulting in unnecessary energy consumption and reduced efficiency. This project designs a digital Smart Adaptive Streetlight Controller using Verilog HDL that:

  1.Detects vehicle movement and estimates speed using sensor pulse inputs.
     
  2.Automatically adjusts LED brightness using PWM based on the detected speed.
     
  3.Uses an FSM to switch between Idle, Active, and Bright lighting modes.
     
  4.Provides energy-efficient illumination by dimming lights during low or no traffic.
     
  5.Integrates an optional timer for auto-shutdown during long inactivity.
     
  6.Offers complete simulation-based verification of speed detection, PWM output, and mode transitions.

## Objectives

->Detect vehicle movement and speed.

->Adjust LED brightness based on speed.

->Implement FSM for lighting modes.

->Reduce energy usage during low traffic.

->Enable auto-shutdown after inactivity.

->Verify all functions through simulation.


## Block Diagram


<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/5200da31-4268-43b6-88e5-4312fff250f3" />


## System Description

## Inputs

1.Speed Sensor Input

  ->Simulated digital input representing vehicle speed levels (Low / Medium / High).
    
  ->Determines the required brightness mode.
           
2.Clock Signal
 
   ->Provides timing reference for PWM generation, FSM transitions, and timers.
           
3.Reset Signal
     
  ->Resets the entire system to the Idle mode and default brightness levels.
           
4.Enable/Trigger Signal
     
  ->Indicates vehicle presence and activates the streetlight from idle to active mode.

## Outputs

1.PWM Output Signal
     
  ->Controls LED brightness proportionally (Dim / Medium / Bright) based on vehicle speed.
           
2.Mode Indicator Output
     
  ->Shows FSM states (Idle, Active, Bright) for monitoring or debugging.
           
3.Timer Status Output
     
  ->Indicates auto-shutdown timing when no vehicle is detected.
           
4.Streetlight Control Output
     
  ->Final drive signal to the LED/lighting hardware, based on processed speed and mode.


 ## Verilog Implementation

 ## Verilog code
 ```
 `timescale 1ns / 1ps
module speed_detector(
    input clk,
    input sensor_pulse,
    output reg [7:0] speed
);

    reg [15:0] counter;
    always @(posedge clk) begin
        if(sensor_pulse) begin
            speed <= counter[15:8]; 
            counter <= 0;
        end else begin
            counter <= counter + 1;
        end
    end
endmodule
module pwm_generator(
    input clk,
    input [7:0] duty_cycle,
    output reg pwm_out
);

    reg [7:0] counter = 0;
    always @(posedge clk) begin
        counter <= counter + 1;
        pwm_out <= (counter < duty_cycle) ? 1 : 0;
    end
endmodule
module streetlight_fsm(
    input clk,
    input [7:0] speed,
    output reg [7:0] duty_cycle
);

    reg [1:0] state = 0;

    always @(posedge clk) begin
        case(state)

            2'b00: begin
                duty_cycle <= 20;       
                if(speed > 10)
                    state <= 2'b01;
            end

            2'b01: begin
                duty_cycle <= 100;    
                if(speed > 50)
                    state <= 2'b10;
                else if(speed < 5)
                    state <= 2'b00;
            end

            2'b10: begin
                duty_cycle <= 200;     
                if(speed < 40)
                    state <= 2'b01;
            end

        endcase
    end
endmodule
module timer(
    input clk,
    input reset,
    output reg timeout
);
    reg [31:0] count;

    always @(posedge clk or posedge reset) begin
        if(reset) begin
            count <= 0;
            timeout <= 0;
        end
        else begin
            if(count > 32'd50_000_000)  
                timeout <= 1;
            else begin
                timeout <= 0;
                count <= count + 1;
            end
        end
    end
endmodule
module top_module(
    input clk,
    input sensor_pulse,
    output pwm_led
);

wire [7:0] speed;
wire [7:0] duty_cycle;
speed_detector U1(clk, sensor_pulse, speed);
streetlight_fsm U2(clk, speed, duty_cycle);
pwm_generator U3(clk, duty_cycle, pwm_led);
endmodule

```
## TestBench Code
```
module tb_top_module();
reg clk = 0;
reg sensor_pulse = 0;
wire pwm_led;
top_module DUT(clk, sensor_pulse, pwm_led);
always #5 clk = ~clk;
initial begin
    #50 sensor_pulse = 1; #10
    sensor_pulse = 0;
    repeat(3) begin
        #20 sensor_pulse = 1;
        #10 sensor_pulse = 0;
    end
    #200;

    $finish;
end
endmodule
```

## Output Waveform

 <img width="1918" height="1073" alt="image" src="https://github.com/user-attachments/assets/bd6834d0-331e-430e-9ae8-b01ee7bbef69" />

## Conclusion
The Smart Adaptive Streetlight System successfully demonstrates how vehicle-based sensing and PWM control can optimize road illumination. By adjusting brightness according to traffic speed, the design ensures both energy efficiency and improved safety. The FSM-based control enables smooth and reliable mode transitions under varying conditions. Through intelligent dimming during low-traffic periods, the system significantly reduces unnecessary power usage. Overall, the project provides a practical and scalable approach for modern smart-city lighting applications.
