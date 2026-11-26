# Autora: Isabela Alvarez Ramos
# Cobertura Tabla 3:
# - add, sub: operandos positivos y negativos
# - and, or: operandos con unos en los 16 bits altos
# - slt, slti: resultados 1 y 0; casos negativos donde signo y sin signo difieren
# - addi: inmediato negativo
# - addiu, addu: mediante pseudoinstrucciones li (-32768, 32767) y move
# - $zero: escritura en $zero comprobando que mantiene 0
# - syscall: servicios 11 y 10

        .text
main:
        # ADD
        la   $a0, msg_add
        jal  print_str
        addi $s0, $zero, 5
        addi $s1, $zero, 7
        addi $s2, $zero, -5
        addi $s3, $zero, 3
        add  $a0, $s0, $s1          # 5 + 7 = 0000000c
        jal  ph_sp
        add  $a0, $s2, $s3          # -5 + 3 = fffffffe
        jal  ph_sp
        add  $a0, $s2, $s2          # -5 + -5 = fffffff6
        jal  ph_nl

        # SUB
        la   $a0, msg_sub
        jal  print_str
        sub  $a0, $s1, $s0          # 7 - 5 = 00000002
        jal  ph_sp
        sub  $a0, $s3, $s2          # 3 - (-5) = 00000008
        jal  ph_sp
        sub  $a0, $s2, $s3          # -5 - 3 = fffffff8
        jal  ph_nl

        # AND / OR con unos en 16 bits altos
        la   $a0, msg_ao
        jal  print_str
        lui  $t0, 0xffff
        ori  $t0, $t0, 0x00f0       # 0xffff00f0
        lui  $t1, 0x0f0f
        ori  $t1, $t1, 0xffff       # 0x0f0fffff
        and  $a0, $t0, $t1          # 0f0f00f0
        jal  ph_sp
        lui  $t2, 0xf0f0
        ori  $t3, $zero, 0xff00
        or   $a0, $t2, $t3          # f0f0ff00
        jal  ph_sp
        or   $a0, $t0, $t1          # ffffffff
        jal  ph_nl

        # SLT / SLTI
        la   $a0, msg_slt
        jal  print_str
        addi $t4, $zero, -1
        addi $t5, $zero, 1
        slt  $a0, $t4, $t5          # -1 < 1 -> 00000001
        jal  ph_sp
        slt  $a0, $t5, $t4          # 1 < -1 -> 00000000
        jal  ph_sp
        slti $a0, $t4, 1            # -1 < 1 -> 00000001
        jal  ph_sp
        slti $a0, $t5, -1           # 1 < -1 -> 00000000
        jal  ph_nl

        # ADDI / LI (addiu) / MOVE (addu)
        la   $a0, msg_ai
        jal  print_str
        addi $t6, $zero, 10
        addi $a0, $t6, -30          # ffffffec
        jal  ph_sp
        li   $t7, -32768            # ffff8000
        move $a0, $t7
        jal  ph_sp
        li   $t7, 32767             # 00007fff
        move $a0, $t7
        jal  ph_nl

        # $ZERO inmutable
        la   $a0, msg_zr
        jal  print_str
        add  $zero, $s0, $s1
        addi $zero, $zero, 99
        move $zero, $s2
        add  $a0, $zero, $zero      # 00000000
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
msg_add: .asciiz "ADD "
msg_sub: .asciiz "SUB "
msg_ao:  .asciiz "AND/OR "
msg_slt: .asciiz "SLT "
msg_ai:  .asciiz "ADDI/LI "
msg_zr:  .asciiz "ZERO "