**Problem Statement**
 To Find the Inverse of a matrix and if there exsist no inverse , output no Inverse possible.

**Development Method**:
	a. VS-Code:
	Visual studio code (VSCode) is a free and open-source code editor developed by Microsoft. It is a highly popular and versatile tool for software development. This VSCode has many extensions which is used to compile a program.	
                  Using VSCode with the Venus Simulator for RV32IMAF architectures means you have support for Floating point (F) instructions and these instruction follows IEEE-754 32-bit format for floating-point numbers.

	 b. RISC-V Venus Simulator embedded in  VS Code:
This Visual Studio Code extension embeds the popular Venus RISC-V simulator. It provides a standalone learning environment as no other tools are needed. It runs RISC-V assembly code with the standard debugging capabilities of VS Code.

	c.  Compiler Explorer:
	Used compiler explorer for figuring 	out the methodology and approach behind development I	n RISC V.


 **Steps:**
    • Basic Computer Architecture for learning the instructions
    
    • Googled pseudo codes and wrote C code for different parts like loop, ration , various row operations
    
    • Used the material provided 
    
    • Utilizing several google resources wrote code in RISC V in Venus using C code as framework.
    
    • Matching the solution at several BreakPoints eg. First Matrix, Augmented Matrix, Subroutines .
    
    • Debugging any error at all the breakpoints and kept on matching with register values.



**Logic Behind Finding Inverse.**

Representing the Matrix as an Augmented Matrix:
    • Step 1: To find the inverse of a matrix, we start by representing the given matrix as an augmented matrix. For a 3x3 matrix, this means augmenting the matrix with an identity matrix of the same size.
    
    • Example:
        ◦ Let's consider the following 3x3 matrix:
          Matrix A = 1  2  1
                             1  2  1
                             1  2  1
        ◦ We augment this matrix with the identity matrix of size 3x3:
          Matrix Augmented Matrix [A∣I ] =  1 2 1 | 1 0 0 
        ◦                                                            2 1 2 | 0 1 0
                                                                     1 2 1 | 0  0 1
Elimination Step:

    • Step 2: The next step is to perform elimination to transform the augmented matrix into row echelon form (REF).
    
    • Step 3: For each row below the current row, we subtract a multiple of the current row from that row to make the element below the pivot element zero. The multiple is chosen so that the element below the pivot becomes zero.
    
    • Example:
    
        ◦ Iteration 1:
            ▪ Use row operations to create zeros in the first column below the pivot element in the first row (pivot = 1):
              R2=R2−2×R1andR3=R3+R1
              
            ▪ The matrix now looks like this:
                          1   2   1   |   1   0   0
                          0  -3   0   |  -2   1   0
                          0   4   2   |   1   0   1
                          
        ◦ Iteration 2:
            ▪ Next, create a zero in the second column below the pivot in the second row (pivot = -3):
              R3 = R3−(−4/-3)×R2
              
            ▪ The matrix now looks like this:
            
        ◦ Iteration 3:
            ▪ Finally, normalize the pivot element in the third row:
              R3 =  1/2 (R3)
              
            ▪ The matrix now looks like this:
                          1   2   1   |   1      0     0
                          0  -3   0   |  -2     1     0
                          0   0   1   |  -1/2  -2/3   1/2
                          
Final Matrix:

    • Step 4: After performing all elimination steps, the matrix on the left side of the augmented matrix is reduced to the identity matrix, and the right side of the augmented matrix becomes the inverse of the original matrix.
    
    • Final Augmented Matrix:
                          1   0   0   |   1      2     -1
                          0   1   0   |  -2     -1     2
                          0   0   1   |  3      -4     1
                          
    • Result:
        ◦ The right side of the matrix now contains the inverse of the original matrix A. 
          Inverse of A =                                         1      2     -1
                                                                 -2     -1     2
                                                                  3      -4     1
                                                                  

**Architectural Strategies.**

        3.1 Gaussian Elimination Algorithm:
Gaussian Jordan Elimination is the efficient algorithm to find the inverse of a matrix. The algorithm we have used here is partial pivoting which reduces complexity for mathematical calculations.

        3.2 Modular Design:
Program should be easy to read for that I have used labels which make it readable. In order to reduce memory requirement, program is designed such that it reuses many registers.

        3.3 Data Types and Sizes:
I’m using 32-bit IEEE 754 single-precision floating-point numbers, with 1 bit for sign, 8 bits for the exponent, and 23 bits for the fraction. This format represents real numbers with a sign, magnitude, and precision suitable for various applications.




 **Pseudo Code.**

1. Start
   
3.  Order of Matrix (n).
   
5. Read Matrix (A):
   For i = 1 to n
     For j = 1 to n
       Read Ai,j
     Next j
   Next i

6. Augment Identity Matrix of Order n to Matrix A:
   For i = 1 to n
     For j = 1 to n
       If i = j
         Ai,j+n = 1
       Else
         Ai,j+n = 0
       End If
     Next j
   Next i

7. Apply Gauss Jordan Elimination on Augmented Matrix (A):
   For i = 1 to n
     If Ai,i = 0
       Do Partial Pivoting 
         If Ai,i still=0
          Print "Mathematical Error!"
          Stop
         End If
       End If
     
     For j = 1 to n
       If i ≠ j
         Ratio = Aj,i / Ai,i
         
         For k = 1 to 2*n
           Aj,k = Aj,k - Ratio * Ai,k
         Next k
       End If
     Next j
   Next i

8. Row Operation to Convert Principal Diagonal to 1:
   For i = 1 to n
     For j = n+1 to 2*n
       Ai,j = Ai,j / Ai,i
     Next j
   Next i

9. Display Inverse Matrix:
   For i = 1 to n
     For j = n+1 to 2*n
       Print Ai,j
     Next j
   Next i

10. Stop
