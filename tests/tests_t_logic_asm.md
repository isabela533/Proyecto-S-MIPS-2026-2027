# Autora: Isabela Alvarez Ramos
# Cobertura Tabla 3:
# - andi, ori: inmediatos con bit 15 = 1 (extension con ceros)
# - lui: inmediatos con bit 15 = 1
# - sll, srl: shamt = 0 y shamt != 0; srl sobre valor negativo (entran ceros)
# - syscall: servicios 11 y 10

        .text
main:
        # ANDI / ORI (bit 15 = 1 -> extension con ceros)
        la   $a0, msg_ao
        jal  print_str
        addi $s0, $zero, -1         # 0xffffffff
        andi $a0, $s0, 0x8001       # 00008001
        jal  ph_sp
        ori  $a0, $zero, 0x8000     # 00008000
        jal  ph_sp
        lui  $s1, 0x1234
        ori  $a0, $s1, 0xf00f       # 1234f00f
        jal  ph_nl

        # LUI (bit 15 = 1)
        la   $a0, msg_lui
        jal  print_str
        lui  $a0, 0x8001            # 80010000
        jal  ph_sp
        lui  $a0, 0xffff            # ffff0000
        jal  ph_nl

        # SLL (shamt = 0 y shamt != 0)
        la   $a0, msg_sll
        jal  print_str
        ori  $s2, $zero, 0x1234
        sll  $a0, $s2, 0            # 00001234
        jal  ph_sp
        sll  $a0, $s2, 4            # 00012340
        jal  ph_sp
        sll  $a0, $s2, 31           # 00000000
        jal  ph_sp
        sll  $a0, $s2, 30           # 00000000
        jal  ph_nl

        # SRL (shamt = 0, shamt != 0, valor negativo)
        la   $a0, msg_srl
        jal  print_str
        addi $s3, $zero, -16        # 0xfffffff0
        srl  $a0, $s3, 0            # fffffff0
        jal  ph_sp
        srl  $a0, $s3, 4            # 0fffffff (entran ceros)
        jal  ph_sp
        srl  $a0, $s3, 31           # 00000001
        jal  ph_sp
        srl  $a0, $s2, 8            # 00000012
        jal  ph_nl

        addi $v0, $zero, 10
        syscall

ph_sp:
        addi $sp, $sp, -4
        sw   $ra, 0($sp)
        jal  print_hex
        addi $a0, $zero, 32
        addi $v0, $zero, 11
        syscall
        lw   $ra, 0($sp)
        addi $sp, $sp, 4
        jr   $ra

ph_nl:
        addi $sp, $sp, -4
        sw   $ra, 0($sp)
        jal  print_hex
        addi $a0, $zero, 10
        addi $v0, $zero, 11
        syscall
        lw   $ra, 0($sp)
        addi $sp, $sp, 4
        jr   $ra

print_hex:
        add  $t0, $a0, $zero
        addi $t1, $zero, 8
ph_loop:
        srl  $t2, $t0, 28
        slti $t3, $t2, 10
        beq  $t3, $zero, ph_let
        addi $a0, $t2, 48
        j    ph_out
ph_let:
        addi $a0, $t2, 87
ph_out:
        addi $v0, $zero, 11
        syscall
        sll  $t0, $t0, 4
        addi $t1, $t1, -1
        bne  $t1, $zero, ph_loop
        jr   $ra

print_str:
        add  $t0, $a0, $zero
ps_loop:
        lbu  $a0, 0($t0)
        beq  $a0, $zero, ps_end
        addi $v0, $zero, 11
        syscall
        addi $t0, $t0, 1
        j    ps_loop
ps_end:
        jr   $ra

        .data
msg_ao:  .asciiz "ANDI/ORI "
msg_lui: .asciiz "LUI "
msg_sll: .asciiz "SLL "
msg_srl: .asciiz "SRL "