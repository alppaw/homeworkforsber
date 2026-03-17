# Обзор системы инструкций RISC-V RV32I

---

# 1. R-Type (Register - операции регистр-регистр)

Формат R-типа используется для арифметических и логических операций, где все операнды находятся в регистрах.

```systemverilog
R : begin
    opcode = cmd[6:0];
    rd = cmd[11:7];
    rs1 = cmd[19:15];
    rs2 = cmd[24:20];
    funct3 = cmd[14:12];
    funct7 = cmd[31:25];

    // sltu
    rd_r = (rs1 < rs2) ? 1 : 0;
end
```

---

# 2. I-Type (Immediate - операции с немедленными значениями)

I-тип используется для операций, где один операнд является немедленным значением (константой).

```systemverilog
I : begin
    opcode = cmd[6:0];
    rd = cmd[11:7];
    rs1 = cmd[19:15];
    funct3 = cmd[14:12];
    imm = cmd[31:20];

    // Пример: загрузка байта (lb)
    rd_r = mmem[rs1 + imm][0:7];
end
```

---

# 3. S-Type (Store - инструкции сохранения)

S-тип используется для инструкций сохранения данных в память.

```systemverilog
S : begin
    opcode = cmd[6:0];
    rs1 = cmd[19:15];
    rs2 = cmd[24:20];
    funct3 = cmd[14:12];
    imm = {cmd[31:25], cmd[11:7]};

    // sb
    mmem[rs1 + imm][0:7] = rs2[0:7];
end
```

---

# 4. B-Type (Branch - условные переходы)

B-тип используется для условных переходов (ветвлений) в коде.

```systemverilog
B : begin
    opcode = cmd[6:0];
    rs1 = cmd[19:15];
    rs2 = cmd[24:20];
    funct3 = cmd[14:12];
    imm = {cmd[12], cmd[7], cmd[30:25], cmd[11:8]};

    // bge
    PC = (rs1 >= rs2) ? PC + imm : PC;
end
```

---

# 5. U-Type (Upper-immediate - верхние немедленные)

U-тип используется для загрузки 20-битных констант в верхние разряды регистра.

```systemverilog
U : begin
    opcode = cmd[6:0];
    rd = cmd[11:7];
    u_imm = cmd[31:12];

    // lui
    rd_r = imm << 12;
end
```

---

# 6. J-Type (Jump - безусловные переходы)

J-тип используется для безусловных переходов с сохранением адреса возврата.

```systemverilog
J : begin
    opcode = cmd[6:0];
    rd = cmd[11:7];
    u_imm = {cmd[31], cmd[19:12], cmd[20], cmd[30:21]};

    // jal
    rd = PC + 4;      а
    PC = PC + imm;    
end
```

---

# 7. Полный код
*код не имеет ничего общего с реальным модулем rv32i, написан только для демонстрации*
*выбран язык systemverilog, так как можно было выбрать любой удобный, ну и он по сути си-образный*

```systemverilog
module rv_instr(
    input clk,
    input rst_n,
    input [2:0] type,
    input [31:0] cmd,
    output [5:0] rd
);

    localparam R = 3'b000;
    localparam I = 3'b001;
    localparam S = 3'b010;
    localparam B = 3'b011;
    localparam U = 3'b100;
    localparam J = 3'b101;
    localparam INIT = 3'b111;

    logic [31:0] mmem [0:256];
    // mem init
    // .....
    //

    logic [31:0] PC = 32'b0;
    logic [6:0] opcode;
    logic [4:0] rd_r,rs1,rs2;
    logic [2:0] funct3;
    logic [6:0] funct7;
    logic [11:0] imm;
    logic [19:0] u_imm;
    always_comb begin
        case(type_r)
            INIT : begin
                opcode = '0;
                rd = '0;
                rs1 = '0;
                rs2 = '0;
                funct3 = '0;
                funct7 = '0;
                imm = '0;
                u_imm = '0;
            end
            R : begin
                opcode = cmd[6:0];
                rd = cmd[11:7];
                rs1 = cmd[19:15];
                rs2 = cmd[24:20];
                funct3 = cmd[14:12];
                funct7 = cmd[31:25];

                // sltu
                rd_r = (rs1 < rs2) ? 1 : 0;
            end

            I : begin
                opcode = cmd[6:0];
                rd = cmd[11:7];
                rs1 = cmd[19:15];
                funct3 = cmd[14:12];
                imm = cmd[31:20];

                //lb
                rd_r = mmem[rs1 + imm][0:7]
            end

            S : begin
                opcode = cmd[6:0];
                rs1 = cmd[19:15];
                rs2 = cmd[24:20];
                funct3 = cmd[14:12];
                imm = {cmd[31:25],cmd[11:7]};


                //sb
                mmem[rs1 + imm][0:7] = rs2[0:7];
            end

            B : begin
                opcode = cmd[6:0];
                rs1 = cmd[19:15];
                rs2 = cmd[24:20];
                funct3 = cmd[14:12];
                imm = {cmd[12],cmd[7],cmd[30:25],cmd[11:8]};

                //bge
                PC = (rs1 >= rs2) ? PC = PC + imm : PC;
            end

            U : begin
                opcode = cmd[6:0];
                rd = cmd[11:7];
                u_imm = cmd[31:12];

                //lui
                rd_r = imm << 12;
            end

            J : begin
                opcode = cmd[6:0];
                rd = cmd[11:7];
                u_imm = {cmd[31],cmd[19:12],cmd[20],cmd[30:21]};

                //jal
                rd = PC + 4;
                PC = PC + imm;
            end
        endcase
end

    assign rd = rd_r;
endmodule
```
