# OUTPUT
![image](https://github.com/shreyasingh2302vl10/IFFT_using_IP_Core/blob/b9d0e9ff10fc74155dc4b500f452710cf62cf76a/Screenshot%202026-07-06%20022951.png)
# Comparision
![image](https://github.com/shreyasingh2302vl10/IFFT_using_IP_Core/blob/d22ab5701b1175a5c2c9b166e40337f4fe5c8dc2/Screenshot%202026-07-06%20025513.png)
# CODE
```verilog
`timescale 1ns / 1ps

module Input(
    input clk,
    input [15:0] a,
    output [15:0] b
);

    
    reg [2:0] count = 0;

    
    wire s_axis_config_tvalid;
    wire s_axis_config_tready;
    
    wire s_axis_data_tvalid;
    wire s_axis_data_tready; // FFT core batayega kab ready hai
    wire s_axis_data_tlast;
    
    wire m_axis_data_tvalid;
    wire m_axis_data_tready;

    
    reg config_done = 1'b0;
    always @(posedge clk) begin
        if (s_axis_config_tvalid && s_axis_config_tready) begin
            config_done <= 1'b1;
        end
    end
    assign s_axis_config_tvalid = !config_done;

   
    assign s_axis_data_tvalid = 1'b1; 
    
    
    assign s_axis_data_tlast = (count == 3'd7);

   
    assign m_axis_data_tready = 1'b1;

   
    always @(posedge clk) begin
        
        if (s_axis_data_tvalid && s_axis_data_tready) begin
            if (count == 3'd7) begin 
                count <= 3'd0;
            end else begin 
                count <= count + 1;
            end
        end
    end

    // --- Xilinx FFT Core Instantiation ---
    xfft_0 fft (
        .aclk(clk),

        // Config Channel
        .s_axis_config_tdata(8'd1), 
        .s_axis_config_tvalid(s_axis_config_tvalid),
        .s_axis_config_tready(s_axis_config_tready),

        // Slave Input Data Channel
        .s_axis_data_tdata(a),
        .s_axis_data_tvalid(s_axis_data_tvalid),
        .s_axis_data_tready(s_axis_data_tready), // Linked properly
        .s_axis_data_tlast(s_axis_data_tlast),

        // Master Output Data Channel
        .m_axis_data_tdata(b),
        .m_axis_data_tvalid(m_axis_data_tvalid),
        .m_axis_data_tready(m_axis_data_tready)
    );

endmodule
```
# TESTBENCH
```verilog
`timescale 1ns / 1ps

module tb();
    reg  [15:0] a;
    reg         clk;
    wire [15:0] b;

    // Instantiating the corrected design module
    Input dut (
        .a(a),
        .b(b),
        .clk(clk)
    );

    // Clock Generation
    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end 

    // Stimulus
    initial begin 
        a = 16'h0001;
        #10 a = 16'h0002;
        #10 a = 16'h0003;
        #10 a = 16'h0004;
        #10 a = 16'h0005;
        #10 a = 16'h0006;
        #10 a = 16'h0007;
        #10 a = 16'h0008;
        #40;
        $finish; 
    end 

    initial begin
        $monitor("Time=%0t clk=%b Input_A=%h Output_B=%h", $time, clk, a, b);
    end
endmodule
```
