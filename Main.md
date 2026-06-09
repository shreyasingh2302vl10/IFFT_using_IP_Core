```verilog
`timescale 1ns / 1ps

// =======================================================
// 1. MAIN TOP SYSTEM WRAPPER
// =======================================================
module top_ifft_system (
    input wire clk,
    input wire rst,
    output wire [15:0] M_AXIS_DATA_0_tdata;
    output wire m_axis_data_tvalid_0       
);

    // Internal wires to connect Driver and IFFT Block Design
    wire [15:0] connection_data;
    wire connection_valid;
    wire connection_ready;

    // Custom Input Driver Instance
    basic_driver your_driver_inst (
        .clk(clk),
        .rst(rst),
        .ready_from_ifft(connection_ready), 
        .data_to_ifft(connection_data),     
        .valid_to_ifft(connection_valid)    
    );

  // Ek naya wire banaiye jo batayega ki hum aakhiri sample par hain
    wire connection_last;
    
    // Kyunki N=8 hai, toh jab count 7 par pahunchega (0 se 7 = 8 samples), tab tlast high hoga
    assign connection_last = (count == 3'd7) ? 1'b1 : 1'b0;

    // Ab module instantiation mein isko connect kijiye:
    ifft_1_wrapper uut (
        .aclk_0(clk),                                    
        .S_AXIS_CONFIG_0_tdata(8'h00),  
        .S_AXIS_CONFIG_0_tvalid(1'b1), 
        
        .S_AXIS_DATA_0_tdata(connection_data),   
        .S_AXIS_DATA_0_tvalid(connection_valid), 
        .S_AXIS_DATA_0_tready(connection_ready), 
        
        .S_AXIS_DATA_0_tlast(connection_last),     // <-- IS LINE KO 1'b0 SE BADAL KAR YEH KIJIYE!
        
        .M_AXIS_DATA_0_tdata(m_axis_data_tdata),
        .M_AXIS_DATA_0_tvalid(m_axis_data_tvalid),
        .M_AXIS_DATA_0_tready(1'b1)     
    );
endmodule 


// =======================================================
// 2. BASIC INPUT DRIVER MODULE
// =======================================================
module basic_driver (
    input wire clk,
    input wire rst,
    input wire ready_from_ifft,          
    output reg [15:0] data_to_ifft,      
    output reg valid_to_ifft             
);

    reg [2:0] count; 

    // Valid signal generation
    always @(posedge clk or posedge rst) begin
        if (rst) begin
            valid_to_ifft <= 1'b0;
        end else begin
            valid_to_ifft <= 1'b1; // Reset ke baad driver hamesha active rahega
        end
    end

    // Sequential Data Counter Loop (8 points logic)
    always @(posedge clk or posedge rst) begin
        if (rst) begin
            count        <= 3'b000;
            data_to_ifft <= 16'h0000;
        end else begin
            if (valid_to_ifft && ready_from_ifft) begin
                count <= count + 1'b1;
                
                case (count)
                    3'd0: data_to_ifft <= 16'h000A; // Real = 10, Imag = 0
                    3'd1: data_to_ifft <= 16'h0005; // Real = 5,  Imag = 0
                    3'd2: data_to_ifft <= 16'h0000; // Real = 0,  Imag = 0
                    3'd3: data_to_ifft <= 16'h0002; // Real = 2,  Imag = 0
                    3'd4: data_to_ifft <= 16'h0000; 
                    3'd5: data_to_ifft <= 16'h0101; // Real = 1,  Imag = 1
                    3'd6: data_to_ifft <= 16'h0000; 
                    3'd7: data_to_ifft <= 16'h0008; // Real = 8,  Imag = 0
                    default: data_to_ifft <= 16'h0000;
                endcase
            end
        end
    end

endmodule
```
