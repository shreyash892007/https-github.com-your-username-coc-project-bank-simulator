 Project Description
A C program that simulates a realistic customer queue in a bank over an 8-hour day (480 minutes).
 The program models customer arrivals using a Poisson distribution, manages a dynamic queue of
 customers with linked lists (pointers, structs, malloc/free), and simulates teller service times.
The simulation outputs a report including average, median, mode, standard deviation, and maximum wait times of customers.

Concepts Used
C Concepts:
For loops
If/else logic
Functions (customer_arrives(), serve_customer())
Structs
Pointers
Dynamic memory allocation: malloc(), free()
Math Concepts:
Poisson Distribution (for customer arrivals)
Central tendencies: Mean, Median, Mode
Standard deviation
How to Compile
gcc coc-project-bank-simulator.c

How to Run
After compiling, run: "./a.exe"
