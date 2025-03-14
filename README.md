# DAY-44_100_DAYS_OF_RTL
//HAMMING CODE ENCODER AND DECODER

 // Hamming Code Encoder (7,4) in Verilog
module HammingEncoder (
    input  [3:0] data_in,  // 4-bit input data
    output [6:0] code_out  // 7-bit encoded Hamming code
);
    
    // Parity bit calculations
    assign code_out[0] = data_in[0] ^ data_in[1] ^ data_in[3]; // P1
    assign code_out[1] = data_in[0] ^ data_in[2] ^ data_in[3]; // P2
    assign code_out[2] = data_in[0]; // D1
    assign code_out[3] = data_in[1] ^ data_in[2] ^ data_in[3]; // P4
    assign code_out[4] = data_in[1]; // D2
    assign code_out[5] = data_in[2]; // D3
    assign code_out[6] = data_in[3]; // D4
    
endmodule

// Hamming Code Decoder (7,4) in Verilog
module HammingDecoder (
    input  [6:0] code_in,  // 7-bit received code
    output [3:0] data_out, // 4-bit corrected output
    output reg   error     // Error detection flag
);
    wire p1, p2, p4;
    wire [2:0] parity;
    
    // Parity check bits
    assign p1 = code_in[0] ^ code_in[2] ^ code_in[4] ^ code_in[6];
    assign p2 = code_in[1] ^ code_in[2] ^ code_in[5] ^ code_in[6];
    assign p4 = code_in[3] ^ code_in[4] ^ code_in[5] ^ code_in[6];
    
    assign parity = {p4, p2, p1};
    
    always @(*) begin
        error = (parity != 3'b000); // If parity is non-zero, error exists
    end
    
    // Correct the received code if error exists
    reg [6:0] corrected_code;
    always @(*) begin
        if (error)
            corrected_code = code_in ^ (1 << (parity - 1));
        else
            corrected_code = code_in;
    end
    
    // Extract original data
    assign data_out = {corrected_code[6], corrected_code[5], corrected_code[4], corrected_code[2]};
    
endmodule
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

// Testbench for Hamming Encoder and Decoder
module tb_Hamming74;
    reg [3:0] data_in;
    wire [6:0] code_out;
    wire [3:0] data_out;
    wire error;
    
    // Instantiate modules
    HammingEncoder enc (.data_in(data_in), .code_out(code_out));
    HammingDecoder dec (.code_in(code_out), .data_out(data_out), .error(error));
    
    initial begin
        // Test Cases
        data_in = 4'b1011; #10;
        $display("Data: %b, Encoded: %b, Decoded: %b, Error: %b", data_in, code_out, data_out, error);
        
        data_in = 4'b1100; #10;
        $display("Data: %b, Encoded: %b, Decoded: %b, Error: %b", data_in, code_out, data_out, error);
        
        data_in = 4'b0110; #10;
        $display("Data: %b, Encoded: %b, Decoded: %b, Error: %b", data_in, code_out, data_out, error);
        
        data_in = 4'b0001; #10;
        $display("Data: %b, Encoded: %b, Decoded: %b, Error: %b", data_in, code_out, data_out, error);
        
        $stop;
    end
endmodule
