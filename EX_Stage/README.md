# EX Stage (Execute)

Design the execute stage: ALU operand selection, the ALU, and the branch / jump target and "taken" logic. It contains the **ALU** and the **ALU Control**.

### Module Specification

```SystemVerilog
module ex_stage (
    input  logic [31:0] rs1_data,
    input  logic [31:0] rs2_data,
    input  logic [31:0] imm_out,
    input  logic [31:0] address,
    input  logic        branch,
    input  logic        alu_src,
    input  logic        fn7_5,
    input  logic        mux_inp,
    input  logic [2:0]  fn3,
    input  logic [6:0]  imm11_5,
    input  logic [2:0]  aluop,
    input  logic [6:0]  opcode,
    output logic        and_out_ex,
    output logic [31:0] alu_out,
    output logic [31:0] pc_ex_out
);
    logic [4:0]  alu_ctrl;
    logic [31:0] operand_b;
    logic [31:0] pc_signed_offset;
    logic        zero;

    assign operand_b        = alu_src ? imm_out : rs2_data;
    assign pc_signed_offset = address + imm_out;
    assign pc_ex_out        = mux_inp ? {alu_out[31:1], 1'b0} : pc_signed_offset;
    assign and_out_ex       = (branch && zero) || (opcode == 7'h6F) || (opcode == 7'h67);

    alu_control u_alu_control (
        .alu_op      (aluop),
        .fn3         (fn3),
        .imm11_5     (imm11_5),
        .fn7_5       (fn7_5),
        .control_out (alu_ctrl)
    );

    alu #(.DATA_WIDTH(32)) u_alu (
        .alu_control (alu_ctrl),
        .rs1_data    (rs1_data),
        .rs2_data    (operand_b),
        .alu_out     (alu_out),
        .zero        (zero)
    );
endmodule

```

### Working

- ALU operand B is `imm_out` when `alu_src` is 1, else `rs2_data`; operand A is `rs1_data`.
- The ALU control comes from **ALU Control** (`aluop`, `fn3`, `imm11_5`, `fn7_5`); the ALU's `zero` output is the branch condition.
- `pc_signed_offset = address + imm_out`.
- `pc_ex_out` = `{alu_out[31:1], 1'b0}` when `mux_inp` is 1 (JALR), otherwise `pc_signed_offset`.
- `and_out_ex = (branch && zero) || opcode == 7'h6F (JAL) || opcode == 7'h67 (JALR)`.

### Interface

> [!IMPORTANT]
> Your module, port and parameter names are up to you. Your design must have the same number of ports, with the same directions and widths, as shown above, and must behave as specified. Map your names to this spec in `rnr-stage.yaml` (see the assignment handout).
