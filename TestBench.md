# TESTBENCH
```verilog
`timescale 1ns / 1ps

module tb_top_system();

    // ==========================================
    // 1. SIGNALS DECLARATION (Wires aur Registers)
    // ==========================================
    reg clk;
    reg rst;
    
    // Input Signals (Hamara Data)
    reg [15:0] connection_data;
    reg        connection_valid;
    wire       connection_ready;
    wire       connection_last;      // Frame ka aakhiri sample batane ke liye
    
    // Configuration Signals (IFFT Mode Set Karne Ke Liye)
    reg [7:0]  config_data;
    reg        config_valid;
    wire       config_ready;
    
    // Output Signals (IFFT ka Result)
    wire [15:0] m_axis_data_tdata;  
    wire        m_axis_data_tvalid;
    
    // Internal Counters
    reg [2:0] sample_counter;       // 0 se 7 tak samples count karne ke liye
    reg       is_streaming;         // Data bhejte waqt '1' rahega

    // TLAST LOGIC: Jab sample_counter 7 par hoga, tab yeh automatic 1 ho jayega
    assign connection_last = (sample_counter == 3'd7 && connection_valid) ? 1'b1 : 1'b0;


    // ==========================================
    // 2. IP CORE INSTANTIATION (UUT Connection)
    // ==========================================
    ifft_1_wrapper uut (
        .aclk_0(clk),                                    
        
        // CONFIG CHANNEL (IP ko jagane aur mode set karne ke liye)
        .S_AXIS_CONFIG_0_tdata(config_data),  
        .S_AXIS_CONFIG_0_tvalid(config_valid), 
        .S_AXIS_CONFIG_0_tready(config_ready),
        
        // INPUT DATA CHANNEL (Hamare 8 numbers bhejne ke liye)
        .S_AXIS_DATA_0_tdata(connection_data),   
        .S_AXIS_DATA_0_tvalid(connection_valid), 
        .S_AXIS_DATA_0_tready(connection_ready), 
        .S_AXIS_DATA_0_tlast(connection_last),     
        
        // OUTPUT DATA CHANNEL (Jahan se 450ns par data mila)
        .M_AXIS_DATA_0_tdata(m_axis_data_tdata),
        .M_AXIS_DATA_0_tvalid(m_axis_data_tvalid),
        .M_AXIS_DATA_0_tready(1'b1)     
    );


    // ==========================================
    // 3. CLOCK GENERATOR (50 MHz)
    // ==========================================
    always #10 clk = ~clk; // Har 10ns mein clock ulti hogi (Total Period = 20ns)


    // ==========================================
    // 4. DATA DRIVER PROCESS (State Machine)
    // ==========================================
    always @(posedge clk) begin
        if (rst) begin
            // Reset ke dauran sab zero rahega
            sample_counter   <= 3'd0;
            connection_data  <= 16'h0000;
            connection_valid <= 1'b0;
        end 
        else if (is_streaming) begin
            connection_valid <= 1'b1; // IP ko batao ki data valid hai
            
            // Jab IP ready hoga, tabhi naya sample bhejenge
            if (connection_ready) begin
                sample_counter <= sample_counter + 1'b1;
                
                case (sample_counter)
                    // Format: {Imaginary[7:0], Real[7:0]}
                    3'd0: connection_data <= {8'h00, 8'd100}; // Sample 0: Real=100
                    3'd1: connection_data <= {8'h00, 8'd50};  // Sample 1: Real=50
                    3'd2: connection_data <= {8'h00, 8'd0};   // Sample 2: Real=0
                    3'd3: connection_data <= {8'h00, 8'd20};  // Sample 3: Real=20
                    3'd4: connection_data <= {8'h00, 8'd0};   // Sample 4: Real=0
                    3'd5: connection_data <= {8'h00, 8'd10};  // Sample 5: Real=10
                    3'd6: connection_data <= {8'h00, 8'd0};   // Sample 6: Real=0
                    3'd7: connection_data <= {8'h00, 8'd80};  // Sample 7: Real=80 (Yahan TLAST='1' hoga)
                endcase
                
                // 8 samples poore hone ke baad stream rok do
                if (sample_counter == 3'd7) begin
                    is_streaming <= 1'b0;
                end
            end
        end 
        else begin
            // Stream khatam hone ke baad wires ko clean kar do
            connection_valid <= 1'b0;
            connection_data  <= 16'h0000;
        end
    end


    // ==========================================
    // 5. MAIN TIMELINE SEQUENCE (Initial Block)
    // ==========================================
    initial begin
        // --- Phase 1: System Boot & Reset ---
        clk = 0; 
        rst = 1; 
        config_data = 8'h00; 
        config_valid = 0; 
        is_streaming = 0;
        #200; // 200ns tak reset daba ke rakha
        
        @(posedge clk); 
        rst = 0; // Reset hata diya
        #100;
        
        // --- Phase 2: Configuration (IP Mode Setup) ---
        @(posedge clk); 
        config_data = 8'h00;  // LSB = 0 matlab IFFT mode active
        config_valid = 1'b1;  // IP ko bola "config data pakdo"
        
        @(posedge clk); 
        while (!config_ready) @(posedge clk); // Jab tak IP accept na kare, ruko
        config_valid = 1'b0;  // Accept hote hi config valid off
        #100;
        
        // --- Phase 3: Data Transmission ---
        @(posedge clk); 
        is_streaming = 1;     // Yahan se upar waala `case` block chalu ho jayega
        
        // --- Phase 4: Wait for Output (450ns to 610ns window) ---
        #10000; // 10 micro-seconds tak simulation ko chalne diya
        $finish; // Simulation Khatam!
    end

endmodule
```
