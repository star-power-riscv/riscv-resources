# Simple 5-stage Pipelined RISC-V Core Hierarchy 

## Soc

```verilog
module soc (
  input logic clk,
  input logic reset,

  // 实例化core
  
  // 实例化Instruction Memory
  
  // 实例化Data Memory
);
endmodule
```

### Memory

```verilog
module memory
  input logic en,
  input logic [4:0] address,
  output logic [31:0] out_data,
);
  // 单周期读取数据
  // 可以初始化数据
endmodule
```



### Core

```verilog
module core (
  input logic clk,
  input logic reset,

  // Instrcution Memory IO
  input logic [31:0] imem_data, // 从IMem读取的指令
  input logic imem_valid, // Imem读使能

  // Data Memory IO
  input logic [31:0] dmem_data, // 从DMem读取的数据
  input logic dmem_valid, // Dmem读使能
);
  // 实例化以下5个stage
  // 实例化register file，写法跟上面memory类似，可读可写
endmodule
```

#### Fetch Stage

```verilog
module if_stage (
  input logic clk,
  input logic reset,
  input logic stall, // stall时所有寄存器保持旧值

  input logic [31:0] pc_address_in, // 如果要跳转，这个值告诉PC下一条指令的地址
  input logic pc_branch_valid, // 跳转使能
  
  // 输出给Instruction Momery
  output logic [31:0] pc_address_out, // 这个信号也要输出给Decode阶段，jump时要用
  output logic imem_valid, // 读使能
  
  // 来自Instruction Memory的输入，也就是fetch到的指令
  input logic [31:0] imem_data,

  // 输出给Decode阶段
  output logic [31:0] inst_out, // Fetch到的指令
);
endmodule
```

这里的pc可以先做成组合逻辑，也就是0个cycle完成，这样的话fetch需要一个cycle（从imem读指令的latency）。

#### Decode Stage

```verilog
module id_stage (
  input logic clk,
  input logic reset,
  input logic stall,
  
  input logic [31:0] inst_in, // 指令
  input logic [31:0] pc_address_in, // PC地址

  output logic stall_request,   // Decode需要检查是否存在data hazard，如果存在，这个信号输出给fetch
  
  output logic [31:0] rs1_data,
  output logic [31:0] rs2_data,
  output logic [31:0] imm,
  output logic [4:0] rd_address,
  
  output logic reg_write_en, // 这条指令是否要写回某个寄存器
  output logic load_en, // 是否要load
  output logic store_en, // 是否要store
  
  output logic [6:0] opcode,
  output logic [2:0] funct3,
  output logic [6:0] funct7,
  
  output logic [31:0]  pc_address_out,
);
endmodule
```

#### Execute Stage

```verilog
module ex_stage (
  input logic [31:0] pc_address_in, // PC地址
  
  input logic [31:0] rs1_data,
  input logic [31:0] rs2_data,
  input logic [31:0] imm,
  input logic [4:0] rd_address,
  
  input logic reg_write_en, // 这条指令是否要写回某个寄存器
  input logic load_en, // 是否要load
  input logic store_en, // 是否要store
  
  input logic [6:0] opcode,
  input logic [2:0] funct3,
  input logic [6:0] funct7,

  output logic [31:0] alu_result, // ALU执行结果
  output logic [31:0] rs2_data_out, // 保留rs2值，用于store时写入数据
  output logic [4:0] rd_address_out, // 目的寄存器
  output logic reg_write_en_out,
  output logic load_en_out,
  output logic store_en_out,
  output logic [2:0] func3_out,
);
endmodule
```

#### Memory Stage

```verilog
module mem_stage (
  input logic [31:0] alu_result, // 地址或计算结果
  input logic [31:0] rs2_data, // store 写数据
  input logic [4:0] rd_address, // 目的寄存器
  
  input logic reg_write_en, // 这条指令是否要写回某个寄存器
  input logic load_en, // 是否要load
  input logic store_en, // 是否要store
  
  input logic [2:0] funct3,

  // 与data memory通信
  input logic [31:0] dmem_rdata, // 从dmem读回的数据
  output logic [31:0] dmem_addr, // 发给dmem的地址
  output logic [31:0] dmem_wdata, // 写给dmem的数据
  output logic dmem_we, // 写使能
  output logic [3:0] dmem_be, // 字节使能，有些指令只需要对dmem写入目标字节

  // 输出给WB stage
  output logic [31:0] mem_result, // 输出到WB的数据（可能是来自ALU或DMEM）
  output logic [4:0] rd_address_out,
  output logic reg_write_en_out,
);
endmodule
```

#### Write Back Stage

```verilog
module wb_stage (
  input logic [31:0] mem_result,
  input logic [4:0] rd_address,
  input logic reg_write_en,
);
endmodule
```

