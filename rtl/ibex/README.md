# Ibex RTL

**Processor:** Ibex (RV32IMC), lowRISC — https://github.com/lowRISC/ibex

| Folder | Contents |
|---|---|
| `original/` | Unmodified Ibex RTL. All files are kept because JasperGold needs the package and sub-module files to build each security module. Never edit these. |
| `trojaned_rtl/` | The 39-Trojan benchmark: one complete trojaned copy of a module per TrustHub category. |
| `generated_trojans/` | The Trojan code snippets used to build `trojaned_rtl/`, kept to show how the benchmark was made. |

Only the 7 RTL files behind the 9 security modules are in the benchmark:

| RTL file | Security module(s) | Trojans |
|---|---|---|
| `ibex_pmp.sv` | PMP | 5 (no Availability) |
| `ibex_cs_registers.sv` | CSR | 6 |
| `ibex_controller.sv` | Debug (DO), Exceptions/Traps (ETI), Control Flow (CF), Mode Transition (MT) | 6 |
| `ibex_load_store_unit.sv` | Memory Access (MA) | 5 (no Privilege) |
| `ibex_id_stage.sv` | Instruction Execution (IE) | 6 |
| `ibex_ex_block.sv` | Instruction Execution (IE) | 6 |
| `ibex_wb_stage.sv` | Register Update (RU) | 5 (no Privilege) |

File naming: `<module>/<module>_trojan_<Category>.sv`.
