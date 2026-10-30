# ALU Control

Design the ALU control decoder. It turns the operation class from Main Control, plus the instruction's `fn3` / `fn7_5` / `imm11_5` bits, into the ALU's 5-bit `alu_control` code.

### Module Specification

```SystemVerilog
module alu_control (
    input  logic [2:0] alu_op,
    input  logic [2:0] fn3,
    input  logic [6:0] imm11_5,
    input  logic       fn7_5,
    output logic [4:0] control_out
);
    localparam [4:0] ADD = 5'b00000, SUB = 5'b00001, XOR_ = 5'b00010, OR_ = 5'b00011,
                     AND_ = 5'b00100, SLL = 5'b00101, SRL = 5'b00110, SRA = 5'b00111,
                     SLT = 5'b01000, SLTU = 5'b01001, BEQ = 5'b01010, BNE = 5'b01011,
                     BLT = 5'b01100, BGE = 5'b01101, BLTU = 5'b01110, BGEU = 5'b01111;

    always_comb begin
        case (alu_op)
            3'b000: begin // R-type
                case (fn3)
                    3'b000:  control_out = fn7_5 ? SUB : ADD;
                    3'b001:  control_out = SLL;
                    3'b010:  control_out = SLT;
                    3'b011:  control_out = SLTU;
                    3'b100:  control_out = XOR_;
                    3'b101:  control_out = fn7_5 ? SRA : SRL;
                    3'b110:  control_out = OR_;
                    default: control_out = AND_;
                endcase
            end
            3'b001: begin // I-type ALU
                case (fn3)
                    3'b000:  control_out = ADD;
                    3'b001:  control_out = SLL;
                    3'b010:  control_out = SLT;
                    3'b011:  control_out = SLTU;
                    3'b100:  control_out = XOR_;
                    3'b101:  control_out = (imm11_5 == 7'h20) ? SRA : SRL;
                    3'b110:  control_out = OR_;
                    default: control_out = AND_;
                endcase
            end
            3'b100: begin // Branch
                case (fn3)
                    3'b000:  control_out = BEQ;
                    3'b001:  control_out = BNE;
                    3'b100:  control_out = BLT;
                    3'b101:  control_out = BGE;
                    3'b110:  control_out = BLTU;
                    3'b111:  control_out = BGEU;
                    default: control_out = ADD;
                endcase
            end
            default: control_out = ADD; // load, store, JAL/LUI, other
        endcase
    end
endmodule

```

### Working

| alu_op | Class | control_out |
|---|---|---|
| `000` | R-type | by `fn3`: `000` ADD (SUB if `fn7_5`), `001` SLL, `010` SLT, `011` SLTU, `100` XOR, `101` SRL (SRA if `fn7_5`), `110` OR, `111` AND |
| `001` | I-type ALU | same `fn3` mapping, no SUB; `fn3 = 101` is SRAI if `imm11_5 == 7'h20`, else SRLI |
| `010` | Load | ADD |
| `011` | Store | ADD |
| `100` | Branch | by `fn3`: `000` BEQ, `001` BNE, `100` BLT, `101` BGE, `110` BLTU, `111` BGEU (other `fn3`: ADD) |
| `110` | JAL / LUI | ADD |
| other | - | ADD |

The output codes are the ALU's control table (`00000` ADD, `00001` SUB, `00010` XOR, `00011` OR, `00100` AND, `00101` SLL, `00110` SRL, `00111` SRA, `01000` SLT, `01001` SLTU, `01010`-`01111` BEQ BNE BLT BGE BLTU BGEU).

### Interface

> [!IMPORTANT]
> Your module, port and parameter names are up to you. Your design must have the same number of ports, with the same directions and widths, as shown above, and must behave as specified. Map your names to this spec in `rnr-stage.yaml` (see the assignment handout).
