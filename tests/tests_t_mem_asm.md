# Autora: Isabela Alvarez Ramos
# Cobertura Tabla 3:
# - lw, sw: desplazamiento positivo y desplazamiento negativo
# - lb, lbu: byte con bit 7 = 1 en distintas posiciones (d[1:0] distinto)
# - sb: en las cuatro posiciones (d[1:0] = 0, 1, 2, 3) comprobando vecinos con lw
# - syscall: servicios 11 y 10

        .text
main:
        # LW / SW con desplazamiento positivo y negativo
        la   $a0, msg_lw
        jal  print_str
        la   $s0, buf
        addi $s1, $s0, 12
        lui  $t0, 0xcafe
        ori  $t0, $t0, 0xbabe       # 0xcafebabe
        addi $t1, $zero, -2         # 0xfffffffe
        sw   $t0, 8($s0)            # buf[2] = cafebabe (offset +)
        sw   $t1, -12($s1)          # buf[0] = fffffffe (offset -)
        sw   $zero, 4($s0)          # buf[1] = 00000000
        lw   $a0, -4($s1)           # cafebabe (offset -)
        jal  ph_sp
        lw   $a0, 0($s0)            # fffffffe
        jal  ph_sp
        lw   $a0, 4($s0)            # 00000000 (offset +)
        jal  ph_nl

        # LB (extension con signo en d[1:0] = 0, 1, 2, 3)
        la   $a0, msg_lb
        jal  print_str
        la   $s2, patron            # 0x80f17f83 -> bytes: 83, 7f, f1, 80
        lb   $a0, 0($s2)            # ffffff83
        jal  ph_sp
        lb   $a0, 1($s2)            # 0000007f
        jal  ph_sp
        lb   $a0, 2($s2)            # fffffff1
        jal  ph_sp
        lb   $a0, 3($s2)            # ffffff80
        jal  ph_nl

        # LBU (extension con ceros en d[1:0] = 0, 2, 3 con offset -)
        la   $a0, msg_lbu
        jal  print_str
        addi $s3, $s2, 4
        lbu  $a0, 0($s2)            # 00000083
        jal  ph_sp
        lbu  $a0, -2($s3)           # 000000f1
        jal  ph_sp
        lbu  $a0, -1($s3)           # 00000080
        jal  ph_nl

        # SB en los 4 carriles (0, 1, 2, 3) sobre 0x11223344
        la   $a0, msg_sb
        jal  print_str
        lui  $t2, 0x1122
        ori  $t2, $t2, 0x3344
        sw   $t2, 0($s0)
        ori  $t3, $zero, 0xaa
        sb   $t3, 0($s0)
        lw   $a0, 0($s0)            # 112233aa
        jal  ph_sp
        ori  $t3, $zero, 0xbb
        sb   $t3, 1($s0)
        lw   $a0, 0($s0)            # 1122bbaa
        jal  ph_sp
        ori  $t3, $zero, 0xcc
        sb   $t3, 2($s0)
        lw   $a0, 0($s0)            # 11ccbbaa
        jal  ph_sp
        ori  $t3, $zero, 0xdd
        sb   $t3, 3($s0)
        lw   $a0, 0($s0)            # ddccbbaa
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
patron:  .word 0x80f17f83
buf:     .space 16
msg_lw:  .asciiz "LW/SW "
msg_lb:  .asciiz "LB "
msg_lbu: .asciiz "LBU "
msg_sb:  .asciiz "SB "