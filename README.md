# riscv-kuznechik

The proposed instructions are:

This helps to speed up S-transformation.
1. ld.simd.b/h/w(?)[reg, imm], src -> dst
At the offset of M[reg+imm] there is a substitution table. src register contains bytes/halves/words to be substituted.
for (i = 0; i < XLEN / sizeof(b/h/w); i = i + 1) {
    dst[i-th byte/half/word] = M[reg + imm + src[i-th byte/half/word]]
}

This helps to speed up L-transformation.
2. g.mul.b src1, src2 -> dst
Each byte of both sources get multiplied in the Galois field with specified primitive polynom(stored in the system register for example).
for (i = 0; i < XLEN / sizeof(byte); i = i + 1) {
    dst[i-th byte] = galois_mul_8(src1[i-th byte], src2[i-th byte], polynom);
}

3. xor.collapse.b/h/w src1, src2 -> dst
Collapse all data from given sources into the byte/half/word by xoring it to the dest low part.
for (i = 0; i < XLEN / sizeof(b/h/w); i = i + 1) {
    dst[low b/h/w] ^= src1[i-th byte/half/word];
    dst[low b/h/w] ^= src2[i-th byte/half/word];
}
dst[high] = 0;

4. shift_left.insert.b/h/w src1, src2 -> dst, accum
This instruction takes the low part of the src2 and puts it into the dst register, while the equal size high part of src1 is stored into the accum low part.
dst = {src1[XLEN - sizeof(b/h/w) : 0], src2[sizeof(b/h/w) : 0]};
accum = src1[XLEN : XLEN - sizeof(b/h/w)]

5. move.to.acc
put the value of the register to the accumulator register
