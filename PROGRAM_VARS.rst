************************************
Program Variables (Virtual Machine)
************************************

From gplsrc/desc.h:

Every variable is represented by a DESCRIPTOR structure.  In some cases
this holds all the data associated with the variable.  In other cases it
contains pointers to other structures.
  
 UNASSIGNED
   
   All local variable start out in this state.
  
 ADDR
   
   A pointer to another descriptor.
  
 INTEGER
   
   A 32 bit integer value.
  
 FLOATNUM
   
   A floating point number.
  
 SUBR
   
   A pointer to a memory resident QMBasic program.
   Created by CALL transforming a character string holding the
   name of the subroutine.  This string is still available via
   pointer in the descriptor.
  
 STRING
 
   A character string.
   Strings are stored as a series of STRING_CHUNK structures. By
   holding strings in this way, some operations such as
   concatenation are much faster than using contiguous strings.
   Conversely, some operations are slower.
   The actual string represented by the string chunks may be
   referenced by more than one descriptor. This is controlled
   using a reference count in the first chunk.
   The rmv_saddr and n1 elements of the descriptor are only
   significant if the DF_REMOVE flag is set in which case they
   point to the string chunk and offset within that chunk of the
   current remove pointer position.
  
 FILE_REF
   
   A file variable.
   Points to a FILE_VAR structure.  This includes a reference
   count to allow multiple descriptors to share a FILE_VAR.
  
 ARRAY
 
   An array.
   Points to an ARRAY_HEADER structure.  Again, this has a
   reference count.  The array header contains information about
   the array dimensions and points to a list of ARRAY_CHUNK
   structures.  This two level approach significantly improves
   performance of redimensioning arrays.
  
 COMMON
 
   A common block.
   Common blocks are actually held as arrays, each element of
   which is an entry from the common declaration, perhaps also an
   array.  The common blocks are chained together through the
   next_common item in the array header. Element 0 of the common
   "array" is the name of the block, the remaining elements are
   the common itself. The chain has its head in named_common. The
   blank common is addressed by blank_common.
  
 IMAGE
 
   A screen image.
   Points to a SCREEN_IMAGE structure.  On a Windows QMConsole
   session, this structure contains the actual screen data. On a
   QMTerm session, the data is held locally by QMTerm and a
   unique reference id is stored in the SCREEN_IMAGE.
               
 BTREE
 
   A binary tree data item.
   Not accessible to user mode programs (because it's a bit
   awkward to use), these are used internally by the QMBasic
   compiler and the query processor for fast access tables.
  
 SELLIST
   
   A select list variable.
   Used by QMBasic operations that work with select list
   variables as distinct from numbered select lists.  Because this
   variable type was introduced late in the life of QM, the
   numbered lists are handled differently.
  
 PMATRIX 
   
   Pick style systems use a different form of matrix than 
   Information style systems. Firstly, Pick matrices have no zero
   element. Secondly, a Pick matrix in a common block is just a
   list of simple data items. Both styles have their advantages
   and disadvantages. QM supports both, though the Information
   style is the default. The PMATRIX descriptor defines a Pick
   style matrix in a common block.
  
 SOCK
   
   Socket.
  
 LOCALVARS
 
   A local variable pool.
   Local variables are held as arrays in exactly the same way as
   common. As far as the program using them is concerned, they
   are common variables.
   To allow for recursion, entry to an internal subroutine that
   declares local variables will stack any previous incarnation
   of the local variable pool by chaining it on to the next_common
   item in the array header.
  
 OBJ
   
   Object.
  
 OBJCD
 
   Object code pointer for property access.
  
 OBJCDX
   
   Undefined object routine reference (variant on OBJCD)
  
 PERSISTENT
 
   Persistent public and private variables of a CLASS module.
   For most purposes, this is the same as a COMMON block. The
   most significant difference is that element 0 is not
   reserved for the block name but is available for normal use.

