.data
array1: .float 2, 1, 3, 4, 5,1,0,0,0,0        # First row
        .float 1, 0, 4, 3, 2,0,1,0,0,0       # Second row
        .float 3, 4, 0, 1, 5,0,0,1,0,0       # Third row
        .float 4, 5, 1, 0, 3,0,0,0,1,0       # Fourth row
        .float 5, 2, 1, 4, 0,0,0,0,0,1       # Fifth row

array:  .space 100
checking_identity: .space 100
inverse:  .space 100 

SIZE:           .word 5


inverse_not: .asciiz "Inverse not possible\n"      
hel: .asciiz "Ahmad Siraj Hashmi 2024EET2376 \n"  # Use .asciiz to ensure a null terminator
newline: .asciiz "  "
anotherLine: .asciiz "\n"
last: .asciiz " "
tired: .asciiz "Matrix Inverse \n"
dot: .asciiz "."
minus: .asciiz "-"

.text
main:

    li a0, 4           # Load the syscall code for print_string
    la a1, hel         # Load the address of the string into a1
    ecall              # Make the syscall to print the string

    j making_first_array_from_augumented_array
    
    li a0, 1
    li a1, 10   
    ecall             
    


making_first_array_from_augumented_array:
    li t0, 0                  # Row index           
    li t1, 0                  # Column index
    
    la t2, SIZE                 
    lw t2, 0(t2) 
    slli t2,t2,1
    la t3, array1            
    la t6, array
    li t5,0
    
first_matrix:                     
    la t2, SIZE                   
    lw t2, 0(t2) 
    slli t2,t2,1

    mul t4, t0, t2            # t4 = row index * number of columns
    add t4, t4, t1            # t4 = t4 + column index (offset)
    slli t4, t4, 2            # Multiply index by 4 (size of word)
    add t4, t4, t3            # t4 = base address + offset
    flw ft0, 0(t4)              
    
    la t2, SIZE                # Load address of SIZE
    lw t2, 0(t2) 
    mul t4, t0, t2            # t4 = row index * number of columns
    add t4, t4, t5            # t4 = t4 + column index (offset)
    slli t4, t4, 2            # Multiply index by 4 (size of word)
    add t4, t4, t6            # t4 = t6 is the base address + offset
    fsw ft0, 0(t4)              

    addi t1, t1, 1            # j = j + 1
    
    la s0, SIZE                     # Load address of SIZE
    lw s0, 0(s0) 
    
    addi t5,t5,1                 
    beq t1, s0, matrix_first       # If j == 3, go to new_row

    j first_matrix              # Continue loop


matrix_first:
    addi t0, t0, 1            # i = i + 1  , this is to make sure that we have traversed all the rows                  
    li t1,0
    li t5,0
    
    la s1, SIZE                     
    lw s1, 0(s1)
    beq t0, s1, rgt_first    # If i == 3, exit loop

    j first_matrix              # Continue loop
 
rgt_first: 
    li a0, 4           
    la a1, last         
    ecall

#NOW WE HAVE SUCCESFULLY COPIED THE SQUARE MATRIX FROM THE AUGUMENTED ARRAY ABOVE


real_loop:            
    li t0, 0                  # Row index (i)
    li t1, 0                  # Column index (j)
    la t2, SIZE               # Number of columns (width of the array)
    lw t2, 0(t2) 
    slli t2,t2,1                 
    la t3, array1              # Load address of the augumented matrix into t3 

inner_loop:
    mul t4, t0, t2            # t4 = row index * number of columns
    add t4, t4, t0            # t4 = t4 + column index (offset)
    slli t4, t4, 2            # Multiply index by 4 (size of word)
    add t4, t4, t3            # t4 = base address + offset 
    #LOADING a[i][i] to check 
    flw ft5, 0(t4)
    fcvt.w.s t6 ,ft5

    beq t6,zero,swaptherow  

    mul t4, t0, t2            # t4 = row index * number of columns
    add t4, t4, t1            # t4 = t4 + column index (offset)
    slli t4, t4, 2            # Multiply index by 4 (size of word)
    add t4, t4, t3            # t4 = base address + offset
    lw a1, 0(t4)              # Load word from array into a1
    
    beq t0, t1 ,checking      # if (i!=j)
    
    # Ratio calculation: ratio = a[j][i] / a[i][i]
    mul t4, t0, t2            # Calculate index for a[i][i]
    add t4 , t4 ,t0           # COLUMN INDEX ADDED TO IT
    slli t4, t4, 2            # Multiply by 4 (size of word)
    add t4, t4, t3            # Base address + offset for a[i][i]
    flw ft1, 0(t4)             # Load a[i][i] 

    mul t4, t1, t2            # Calculate index for a[j][i]
    add t4 ,t4, t0
    slli t4, t4, 2            # Multiply by 4 (size of word)
    add t4, t4, t3            # Base address + offset for a[j][i] 
    flw ft0, 0(t4)             # Load a[j][i]
    
    fdiv.s ft2, ft0, ft1      # Perform ft2 = ft0 / ft1  || ft2 = a[j][i] / a[i][i]  (ratio)
     
    li t5, 0                 # k = 0, Initialize loop index
    mv s0,t2


#THIS INSIDE SUBROUTINE IS FOR a[j][k] = a[j][k] - ratio * a[i][k]
    loop_k:
    beq t5, s0, end_loop_k   # If k = 2*n, exit the loop ,starting from o indexing
    
    la t3 , array1
                             # Calculate ratio * a[i][k] and subtract from a[j][k]
    mul t4, t0, s0           # t4 = (row index * number of columns)
    add t4, t4, t5           # t4 = +(column index k)
    slli t4, t4, 2           # t4 = (i * n + k) * 4 (since word size is 4 bytes)
    add t4, t4, t3           # t4 = base address + offset for a[i][k]
    
    flw ft1, 0(t4)             # Load a[i][k]

    fmul.s ft1, ft2, ft1     # ft1 = ratio * a[i][k] (floating-point multiplication)
    
    mul t4, t1, s0           # t4 = j * 2n (row index * number of columns)
    add t4, t4, t5           # t4 = j * 2n + k (column index k)
    slli t4, t4, 2           # t4 = (j *2n + k) * 4 (since word size is 4 bytes)
    add t4, t4, t3           # t4 = base address + offset for a[j][k]

    flw ft3, 0(t4)             # Load a[j][k] into in ft3       
   
    fsub.s ft3 ,ft3,ft1
    fsw ft3, 0(t4)

    # Increment k
    addi t5, t5, 1           # k = k + 1
    j loop_k                 # Repeat the loop

    end_loop_k:
    addi t1, t1, 1            # j = j + 1
    la t5, SIZE                     # Load address of SIZE
    lw t5, 0(t5) 
    #li t5, 3                  # Number of columns
    beq t1, t5, trial         # If j == 3, go to new_row
    
    j inner_loop


trial:
    addi t0, t0, 1            # i = i + 1
    li t1, 0                  # j = 0 (reset column index)
    la t5, SIZE                     # Load address of SIZE
    lw t5, 0(t5) 
    #li t5, 3                  # Number of rows
    beq t0, t5, second            # If i == 3, exit loop

    j inner_loop 


checking:
    addi t1, t1, 1            # j = j + 1
    la t5, SIZE                     # Load address of SIZE
    lw t5, 0(t5) 
    #li t5, 3                  # Number of columns
    beq t1, t5, trial         # If j == 3, go to new_row

    j inner_loop


tochek:
 j second

 swaptherow:
    mul t4, t0, t2            # t4 = row index * number of columns
    add t4, t4, t0            # t4 = t4 + column index (offset)
    slli t4, t4, 2            # Multiply index by 4 (size of word)
    add t4, t4, t3            # t4 = base address + offset
    flw ft5 ,0(t4)
    fabs.s ft5,ft5

    mv s1,t0         #ye Yaha pr [maxrow] stored
    mv s7,t0

    addi t5,t0,1     #this t5 is K one index more than the initial vala
    la s2,SIZE
    lw s2,0(s2)

    pivot_inner_loop:
                beq t5, s2, pivot_done    # if k = n, exit the loop
                mul t4, s1, t2            # t4 = row index * number of columns
                add t4, t4, t0            # t4 = t4 + column index (offset)
                slli t4, t4, 2            # Multiply index by 4 (size of word)
                add t4, t4, t3            # t4 = base address + offset
                flw ft5 ,0(t4)
                fabs.s ft5,ft5
                
                
               mul t4, t5, t2            # t4 = row index * number of columns
               add t4, t4, t0            # t4 = t4 + column index (offset)
               slli t4, t4, 2            # Multiply index by 4 (size of word)
               add t4, t4, t3            # t4 = base address + offset
               flw ft6 ,0(t4)
               fabs.s ft6,ft6

                flt.s s4,ft5,ft6 #ft5 (which contains fabs(a[maxRow][i])
                beqz s4 ,pivot_next 
                mv s1,t5

      
    pivot_next:
      addi t5,t5,1
      j pivot_inner_loop

    pivot_done:
         beq s1,s7, No_in
         j pre_swapper

pre_swapper:
            la t2, SIZE                     # Load address of SIZE
            lw t2, 0(t2) 
            slli t2,t2,1
            li s6, 0                  # Column index (j)
            la t3, array1 
            
swapper:
                mul t4,t0, t2            # t4 = row index * number of columns
                add t4, t4, s6            # t4 = t4 + column index (offset)
                slli t4, t4, 2            # Multiply index by 4 (size of word)
                add t4, t4, t3            # t4 = base address + offset
                flw ft5 ,0(t4)            #0 value
                mv s4,t4    #i jisse swap krna hai maxrow ko uska base adress 
                # jaha pr maxrow ka element store hoga

                mul t4,s1, t2            # t4 = row index * number of columns
                add t4, t4, s6            # t4 = t4 + column index (offset)
                slli t4, t4, 2            # Multiply index by 4 (size of word)
                add t4, t4, t3            # t4 = base address + offset
                flw ft6 ,0(t4)            # 2 value
                mv s5,t4

                fsw ft6,0(s4)
                fsw ft5,0(s5)

                flw ft0, 0(s4)
                flw ft1,0(s5)

                addi s6,s6,1
                beq s6,t2 , swap_out
                j swapper

swap_out:
    j inner_loop   

#THIS LOOP IS TO MAKE THE DIAGNOL ELEMENT 1
second:
    li t0, 0                  # Row index (i)
    la t1, SIZE                     # Load address of SIZE
    lw t1, 0(t1) 
    
    la t2, SIZE                     # Number of columns (width of the array)
    lw t2, 0(t2) 
    slli t2,t2,1                  
    la t3, array1              # Load address of the array into t3

   
sec_last:                     #a[i][i]
    mul t4, t0, t2            # t4 = row index * number of columns
    add t4, t4, t0            # t4 = t4 + column index (offset)
    slli t4, t4, 2            # Multiply index by 4 (size of word)
    add t4, t4, t3            # t4 = base address + offset
    flw ft1, 0(t4)            

    mul t4, t0, t2            # t4 = row index * number of columns
    add t4, t4, t1            # t4 = t4 + column index (offset)
    slli t4, t4, 2            # Multiply index by 4 (size of word)
    add t4, t4, t3            # t4 = base address + offset
    flw ft0, 0(t4)            

    fdiv.s ft2, ft0, ft1      # Perform ft2 = ft0 / ft1
    fsw ft2, 0(t4)

    addi t1, t1, 1            # j = j + 1
    la t5, SIZE                     # Load address of SIZE
    lw t5, 0(t5) 
    slli t5,t5,1
    beq t1, t5, new_sec       # If j == 3, go to new_row

    j sec_last              # Continue loop


new_sec:
    addi t0, t0, 1            # i = i + 1  , this is to make sure that we have traversed all the rows 
    la t1, SIZE               # Load address of SIZE
    lw t1, 0(t1)              # j = 0 (reset column index) DOING THIS BCZ IN AUGUMENTED ARRAY WE ARE COPYING FROM THIRD INDEX

    la t5, SIZE                
    lw t5, 0(t5)
    beq t0, t5, rg             # If i == 3, exit loop

    j sec_last              # Continue loop
 
rg: 
    li a0, 4           # Load the syscall code for print_string
    la a1, last         # Load the address of the string into a1
    ecall


#THIS SUBROUTINE IS TO COPY THE INVERSE FROM THE AUGUMENTED ARRAY INTO THE INVERSE MATRIX I HAVE DECLARED IN DATA PART
Inverse_copy_start_subroutine:
    li t0, 0                  # Row index (i)          
    la t1, SIZE                # Load address of SIZE # Column index (j)
    lw t1, 0(t1) 
    
    la t2, SIZE                     # Load address of SIZE
    lw t2, 0(t2) 
    slli t2,t2,1
    la t3, array1            
    la t6, inverse
    li t5,0
    
Inverse_copy:                     #a[i][i]
    #li t2, 6
    la t2, SIZE                     # Load address of SIZE
    lw t2, 0(t2) 
    slli t2,t2,1

    mul t4, t0, t2            # t4 = row index * number of columns
    add t4, t4, t1            # t4 = t4 + column index (offset)
    slli t4, t4, 2            # Multiply index by 4 (size of word)
    add t4, t4, t3            # t4 = base address + offset
    flw ft0, 0(t4)              
    
    #li t2, 3
    la t2, SIZE                     # Load address of SIZE
    lw t2, 0(t2) 
    mul t4, t0, t2            # t4 = row index * number of columns
    add t4, t4, t5            # t4 = t4 + column index (offset)
    slli t4, t4, 2            # Multiply index by 4 (size of word)
    add t4, t4, t6            # t4 = t6 is the base address + offset
    fsw ft0, 0(t4)              

    addi t1, t1, 1            # j = j + 1
    
    la s0, SIZE                     # Load address of SIZE
    lw s0, 0(s0) 
    slli s0,s0,1
    
    addi t5,t5,1                 
    beq t1, s0, Inverse_loop_next       # If j == 3, go to new_row

    j Inverse_copy              # Continue loop


Inverse_loop_next:
    addi t0, t0, 1            # i = i + 1  , this is to make sure that we have traversed all the rows                 
    la t1, SIZE                     # Load address of SIZE
    lw t1, 0(t1) 
    li t5,0
    
    la s1, SIZE                     
    lw s1, 0(s1)
    beq t0, s1, Inverse_Exit_Part  # If i == 3, exit loop

    j Inverse_copy              # Continue loop
 
Inverse_Exit_Part: 
    li a0, 4           
    la a1, last         
    ecall


#THIS LOOP IS TO MULTIPLY INVERSE OF A MATRIX AND MATRIX 
#STORE THE RESULT IN CHECKING IDENTITY MATRIX
start_check:
    li t0, 0                        # i = 0

outer_i_loop:
    li t1, 0                        # j = 0

outer_j_loop:
    la t2, checking_identity          # Load base address of resultMatrix
    la t3, SIZE                     # Load address of SIZE
    lw t3, 0(t3)                    # Load SIZE
    mul t4, t0, t3                  # t4 = i * SIZE
    add t4, t4, t1                  # t4 = i * SIZE + j
    slli t4, t4, 2                  # t4 = (i * SIZE + j) * 4 (index to byte offset)
    add t2, t2, t4                  # Address of resultMatrix[i][j]
    sw zero, 0(t2)                  # resultMatrix[i][j] = 0

    li t5, 0                        # k = 0

inner_k_loop:
    bge t5, t3, end_inner_k_loop    # if (k >= SIZE) break

    la t6, array              # Load base address of firstMatrix
    mul t4, t0, t3                  # t4 = i * SIZE
    add t4, t4, t5                  # t4 = i * SIZE + k
    slli t4, t4, 2                  # t4 = (i * SIZE + k) * 4 (index to byte offset)
    add t6, t6, t4                  # Address of firstMatrix[i][k]
    flw f0, 0(t6)                   # Load firstMatrix[i][k] into f0

    la t6, inverse                 # Load base address of secondMatrix
    mul t4, t5, t3                  # t4 = k * SIZE
    add t4, t4, t1                  # t4 = k * SIZE + j
    slli t4, t4, 2                  # t4 = (k * SIZE + j) * 4 (index to byte offset)
    add t6, t6, t4                  # Address of secondMatrix[k][j]
    flw f1, 0(t6)                   # Load secondMatrix[k][j] into f1

    fmul.s f2, f0, f1               # f2 = firstMatrix[i][k] * secondMatrix[k][j]

    flw f3, 0(t2)                   # Load current resultMatrix[i][j] into f3
    fadd.s f3, f3, f2               # f3 = resultMatrix[i][j] + (firstMatrix[i][k] * secondMatrix[k][j])
    fsw f3, 0(t2)                   # Store the result back to resultMatrix[i][j]

    addi t5, t5, 1                  # k++

    j inner_k_loop

end_inner_k_loop:
    addi t1, t1, 1                  # j++
    blt t1, t3, outer_j_loop        # if (j < SIZE) continue

    addi t0, t0, 1                  # i++
    blt t0, t3, outer_i_loop        # if (i < SIZE) continue

#END OF THE LOOP 

chkkk:
    li a0, 4           # Load the syscall code for print_string
    la a1, tired         # Load the address of the string into a1
    ecall


#LOOP TO PRINT THE FLOATING POINT VALUE

start_print_loop_of_float:            
    li t0, 0                  # Row index (i)
    li t1, 0                  # Column index (j)
    #li t2, 3                 # Number of columns (width of the array)
    la t2, SIZE                
    lw t2, 0(t2)
    #WANT TO PRINT ANOTHER MATRIX JUST CHANGE NAME HERE 
    la t3, inverse              


mulmat:
    mul t4, t0, t2            # t4 = row index * number of columns
    add t4, t4, t1            # t4 = t4 + column index (offset)
    slli t4, t4, 2            # Multiply index by 4 (size of word)
    add t4, t4, t3            # t4 = base address + offset
    flw ft0, 0(t4)      
    
    addi t4, zero, 0    # Set t4 to 0
    
    loop_a:
       addi t6,zero,0
       fcvt.s.w ft4,t6
       flt.s t6, ft0, ft4 # t6 = 1 if ft0 < 0, t6 = 0 otherwise
       li s0,1
       beq t6,s0 ,negat

       fcvt.w.s t5 ,ft0
       #addi t5,t5,-1
       fcvt.s.w ft3,t5
       addi t6,zero,0
       flt.s t6, ft0, ft3  #if t6 =1 ft0 < ft3
       
       sub t5,t5,t6
       mv a1, t5
       li a0,1
       ecall
       j subop
    
    negat:
        li a0, 4                  # Print newline for the new row
        la a1, minus
        ecall

        li t6,-1
        fcvt.s.w ft5,t6
        fmul.s ft0,ft0,ft5
        j loop_a
       
    subop:
       fcvt.s.w ft2 ,t5
       fsub.s ft0,ft0,ft2
       #la t6,float_value
       addi t6 ,zero, 10
       fcvt.s.w ft1,t6
       #flw ft1 ,0(t6)
       fmul.s ft0,ft0,ft1

       addi t4,t4,1
       li t5 ,1
       beq t4,t5 dot_loop
       li t5,4 
       beq t4,t5 ,end_loop_a

       j loop_a
       
     
    dot_loop:    
     li a0 ,4
     la a1,dot
     ecall
     j loop_a
      
    end_loop_a:
     li a0 ,4
     la a1,newline
     ecall

    addi t1, t1, 1            # j = j + 1
    la t5, SIZE                     # Load address of SIZE
    lw t5, 0(t5)
    beq t1, t5, last_fg       # If j == 3, go to new_row

    j mulmat              # Continue loop

last_fg:
    addi t0, t0, 1            # i = i + 1  , this is to make sure that we have traversed all the rows 
    li t1, 0                  # j = 0 (reset column index) bcz row major order se store hua hai
    la t5, SIZE                     # Load address of SIZE
    lw t5, 0(t5)
    beq t0, t5, karma  # If i == 3, exit loop

    li a0, 4                  # Print newline for the new row
    la a1, anotherLine
    ecall

    j mulmat              # Continue loop
 
karma: 
    j end

No_in:
   li a0,4
   la a1 ,inverse_not
   ecall

end:
    li a0, 10                 # Syscall number for exit (if a0 is used for syscall number)
    li a1, 0                  # Exit status 0
    ecall
    