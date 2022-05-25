****************
evaluation stack
****************
A memory area of contiguous DESCRIPTOR strutures.

Every BASIC program variable is represented by a DESCRIPTOR structure.
In some cases this holds all the data associated with the variable.
In other cases it contains pointers to other structures. (see Program_Vars)

Variables are "passed" to other BASIC routines (CALL) and VM internal c functions via the
evaluation stack.

The BASIC pogram header includes an estimate of the stack depth needed to run the
program. (hdr->stack_depth)

Evaluation stack control variables::

 Evaluation stack control variables are define in kernel.h:

 Public DESCRIPTOR* e_stack_base;      Base of e-stack, apparently initialized via notes below:
 Public DESCRIPTOR* e_stack;           Ptr to next e-stack descriptor 
 Public int16_t e_stack_depth init(0); Depth of e-stack 

 Note: could not see how pointers were initialized until I found this:
  e_stack_base & e_stack are initialized to null:
  (C99 standard) section 6.7.8 clause 10:
  If an object that has static storage duration is not initialized explicitly, then:

    if it has pointer type, it is initialized to a null pointer;
    if it has arithmetic type, it is initialized to (positive or unsigned) zero;
    if it is an aggregate, every member is initialized (recursively) according to these rules;
    if it is a union, the ?rst named member is initialized (recursively) according to these rules.

The memory for the evaluation stack is allocated in function k_call of kernel.c. 
e_stack is then set to point to the first available descriptor on the stack.


Releasing Descriptor::

 k_dismiss() Release descriptor at top of stack, macro for k_release(--e_stack)
 k_release(p - pointer to descr)
 
 For descriptors that "contain" the data, set the descriptor to UNASSIGNED.
 For descriptors that "point" to other memory areas for the descriptors data (like strings),
 release the memory, then set the descriptor to unassigned.
 Note this does not actually release the memory for the passed descriptor!


Stack example::

 program test
 $define OS_EXISTS  2
 path = '/this/way/there'
 stat = ospath(path,OS_EXISTS)
 end

 step
 code    3    path = '/this/way/there'
 S1         0000A5:       LDSLCL   PATH hx: 19   00
 S2         0000A7:       LDSTR      hx: 12   F   "/this/way/there"
 S3         0000B8:       STOR       hx: 16
 code    4  stat = ospath(path,OS_EXISTS)
 S4         0000B9:       LDSLCL   STAT hx: 19   1
 S5         0000BB:       LDSLCL   PATH hx: 19   00
 S6         0000BD:       LDSINT   2
 S7         0000BF:       OSPATH     hx: 99
 S8         0000C0:       STOR       hx: 16
 code    5  end

 symbol table:
 0         PATH
 1         STAT

step S1::

       LDSLCL   PATH   <- (op_ldslcl() in op_loads.c) Load short local variable address 
                          for Variable PATH onto stack.
                          Rem Local variables have their own descriptor structure 
                          (start of which is pointed to by process.program.vars) 
         
       InitDescr(e_stack, ADDR)    <- this stack descriptor will hold an address 
                                   of the local variable's descriptor.
                                   
       (e_stack++)->data.d_addr = process.program.vars + *(pc++) <- the address is the address
                                                                    of the start of local program variables
                                                                    + variable number

         +=============================+
         +       STACK BEFORE          +
         +=============================+
 estack> +    next available descr     +
         +=============================+


         +=============================+
         +       STACK AFTER           +
         +=============================+
 estack> +    next available descr     +
         +=============================+
         + Addr to descriptor for PATH +
         +=============================+
         
         
step S2::

       LDSTR    - (op_ldslcl() in op_loads.c) Load constant string from program object (<= 255 characters?)
       
       InitDescr(e_stack, STRING)   <- this descriptor will hold a string
                                       strings are something we need to figure out!
       
         +=============================+
         +       STACK BEFORE          +
         +=============================+
 estack> +    next available descr     +
         +=============================+
         + Addr to descriptor for PATH +
         +=============================+

         +=============================+
         +       STACK AFTER           +
         +=============================+
 estack> +    next available descr     +
         +=============================+
         + descr for string            + 
         +=============================+
         + Addr to descriptor for PATH + 
         +=============================+
         
step S3::

         STOR    -  (op_stor() in op_loads.c)  -  Store value in variable
         variable = e_stack - 2;
         value = e_stack - 1;
          .
         Release(var_descr)  <-  For descriptors that "contain" the data, set the descriptor to UNASSIGNED.
                                 For descriptors that "point" to other memory areas for the descriptors data,
                                 release the memory, then set the descriptor to unassigned
          .
          .
         *var_descr = *value_descr  <- code moves the value into variable 
                                       (copies the data in value_descr descriptor structure 
                                        into var_descr descriptor structure)
          .
          .
         performs pop(2) so we are back to where we started at S1
         
         +=============================+
         +       STACK BEFORE          +
         +=============================+
 estack> +    next available descr     +
         +=============================+
         + descr for string            +
         +     ( value )               +
         +=============================+
         + Addr to descriptor for PATH + 
         +     ( variable )            +
         +=============================+
         
         +=============================+
         +       STACK AFTER           +
         +=============================+
 estack> +    next available descr     +
         +=============================+
         
step S4::

         LDSLCL   STAT   Load short local variable address

         +=============================+
         +       STACK BEFORE          +
         +=============================+
 estack> +    next available descr     +
         +=============================+

         +=============================+
         +       STACK AFTER           +
         +=============================+
 estack> +    next available descr     +
         +=============================+
         + Addr to descriptor for STAT +
         +=============================+
         
step S5::
         
         LDSLCL   PATH   Load short local variable address

         +=============================+
         +       STACK BEFORE          +
         +=============================+
 estack> +    next available descr     +
         +=============================+
         + Addr to descriptor for STAT +
         +=============================+
         
         +=============================+
         +       STACK AFTER           +
         +=============================+
 estack> +    next available descr     +
         +=============================+
         + Addr to descriptor for PATH +
         +=============================+
         + Addr to descriptor for STAT +
         +=============================+
         
step  S6::
       
         LDSINT   2  (op_ldsint() in op_loads.c)  -  Load short integer 
         InitDescr(e_stack, INTEGER)
         (e_stack++)->data.value = (signed char)(*(pc++))
                                  take the descriptor pointed to by e_stack and set its data value to the value
                                  in memory location (program object) pointed to by pc
           
         +=============================+
         +       STACK BEFORE          +
         +=============================+
 estack> +    next available descr     +
         +=============================+
         + Addr to descriptor for PATH +
         +=============================+
         + Addr to descriptor for STAT +
         +=============================+
         
         +=============================+
         +       STACK AFTER           +
         +=============================+
 estack> +    next available descr     +
         +=============================+
         +  descriptor w/ int 2 data   +
         +=============================+
         + Addr to descriptor for PATH +
         +=============================+
         + Addr to descriptor for STAT +
         +=============================+
         
step  S7::

         OSPATH -  op_ospath in op_dio2.c) OS file system actions
                   OSPATH code / psuedo code shown

          C varables of interest:
          int32_t status = 0;
          int16_t key;
          char path[MAX_PATHNAME_LEN + 1];
          int16_t path_len;
          char name[MAX_PATHNAME_LEN + 1];  
                   
         
         +=============================+
         +       STACK BEFORE          +
         +=============================+
 estack> +    next available descr     +
         +=============================+
         +  descriptor w/ int 2 data   +
         +            Key              +
         +=============================+
         + Addr to descriptor for PATH +
         +(which points to string data)+
         +=============================+
         + Addr to descriptor for STAT +
         +=============================+
         
           /* Get action key */
           descr = e_stack - 1;
           GetInt(descr);
           key = (int16_t)(descr->data.value);
           k_pop(1);
           
         +=============================+
         +   STACK AFTER KEY access    +
         +=============================+
 estack> +    next available descr     +
         +=============================+
         + Addr to descriptor for PATH +
         +(which points to string data)+
         +=============================+
         + Addr to descriptor for STAT +
         +=============================+
         
           /* Get pathname */
           descr = e_stack - 1;
           path_len = k_get_c_string(descr, path, MAX_PATHNAME_LEN)
           k_dismiss();     Rem k_dismiss() Release descriptor at top of stack,
                            macro for k_release(--e_stack)
         
         +=============================+
         + STACK AFTER pathname access +
         +=============================+
 estack> +    next available descr     +
         +=============================+
         + Addr to descriptor for STAT +
         +=============================+
         
          Code which eveluates Key for requested action and sets:
          depending on action
            c integer status  
               -OR-  
            c string name 
         
         For actions that return a status:                       For actions that return a string:
                                                                 
         +=============================+                         +=============================+
         + STACK AFTER Action Request  +                         + STACK AFTER Action Request  +
         + that reurns a status        +                         + that reurns a status        +
         +=============================+                         +=============================+
 estack> +    next available descr     +                 estack> +    next available descr     +
         +=============================+                         +=============================+
         +  descriptor w/ int value    +                         +  descriptor which points    +
         +   of status to return       +                         +   string data to return     +
         +=============================+                         +=============================+
         + Addr to descriptor for STAT +                         + Addr to descriptor for STAT +
         +=============================+                         +=============================+   
         
         
step S8::

         STOR    -  op_stor()  -  Store value in variable

         
         +=============================+
         +       STACK BEFORE          +
         +=============================+
 estack> +    next available descr     +
         +=============================+
         +  descriptor w/ int value    +
         +   of status to return       +
         +     - OR -                  +
         +  descr for string           +
         +     ( value )               +
         +=============================+
         + Addr to descriptor for STAT + 
         +     ( variable )            +
         +=============================+
         
         +=============================+
         +       STACK AFTER           +
         +=============================+
 estack> +    next available descr     +
         +=============================+
         