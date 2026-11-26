# Autora: Isabela Alvarez Ramos
# Cobertura Tabla 3:
# - beq, bne: tomados y no tomados; salto hacia atras en bucle
# - j: salto incondicional
# - jal, jr: procedimiento recursivo no hoja guardando $ra en pila
# - syscall: servicios 11 y 10

        .text
main:
        # BEQ / BNE tomados y no tomados
        la   $a0, msg_br
        jal  print_str
        add  $s0, $zero, $zero
        addi $t0, $zero, 4
        addi $t1, $zero, 4
        addi $t2, $zero, 9

        beq  $t0, $t1, br1          # tomado
        ori  $s0, $s0, 0x10
br1:    ori  $s0, $s0, 0x01
        beq  $t0, $t2, br_err       # no tomado
        ori  $s0, $s0, 0x02
        bne  $t0, $t2, br2          # tomado
        ori  $s0, $s0, 0x20
br2:    ori  $s0, $s0, 0x04
        bne  $t0, $t1, br_err       # no tomado
        ori  $s0, $s0, 0x08
        j    br_ok
br_err:
        ori  $s0, $s0, 0x80
br_ok:
        add  $a0, $s0, $zero        # 0000000f
        jal  ph_nl

        # Bucle hacia atras con BNE
        la   $a0, msg_lp
        jal  print_str
        add  $t3, $zero, $zero      # acumulador
        addi $t4, $zero, 1          # i = 1
        addi $t5, $zero, 11         # limite
loop_b:
        add  $t3, $t3, $t4
        addi $t4, $t4, 1
        bne  $t4, $t5, loop_b       # salto hacia atras
        add  $a0, $t3, $zero        # 55 = 00000037
        jal  ph_sp
        addi $a0, $zero, 9          # 00000009
        jal  ph_nl

        # J directo
        la   $a0, msg_j
        jal  print_str
        addi $s1, $zero, 0x100
        j    salto_dir
        addi $s1, $s1, 0x0ee        # saltada
salto_dir:
        ori  $a0, $s1, 0x011        # 00000111
        jal  ph_nl

        # JAL / JR recursivo no hoja con pila ($ra y $a0)
        la   $a0, msg_jal
        jal  print_str
        addi $a0, $zero, 10
        jal  sum_rec                # sum(10) = 55 = 00000037
        add  $a0, $v0, $zero
        jal  ph_sp
        add  $a0, $sp, $zero        # $sp restaurado = 00003ffc
        jal  ph_nl

        addi $v0, $zero, 10
        syscall

# sum_rec(n): procedimiento recursivo no hoja
sum_rec:
        addi $sp, $sp, -8
        sw   $ra, 4($sp)
        sw   $a0, 0($sp)
        bne  $a0, $zero, sr_paso
        add  $v0, $zero, $zero
        addi $sp, $sp, 8
        jr   $ra
sr_paso:
        addi $a0, $a0, -1
        jal  sum_rec
        lw   $a0, 0($sp)
        lw   $ra, 4($sp)
        addi $sp, $sp, 8
        add  $v0, $v0, $a0
        jr   $ra

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
msg_br:  .asciiz "BR "
msg_lp:  .asciiz "LOOP "
msg_j:   .asciiz "J "
msg_jal: .asciiz "JAL "