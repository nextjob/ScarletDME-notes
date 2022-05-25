*********************************
BASIC compile of internal modules
*********************************

Programs that live in GPL.BP are for the most part "internal" programs. As such, they need to be compiled in a special way.
You must start Scarlet with the internal flag:

/usr/qmsys/bin/qm -internal

You should then be able to compile programs that use the $internal compiler directive as well as the extended version of the $catalogue directive. 

To create a Op-code listing of your program, you add the $EXPLIST compiler directive to your program source. Use of this directive also requires starting Scarlet  with -internal flag.

Note: compiler directive names are hardcoded in the compiler source (BCOMP), not all of which are defined in the documentation, see internal subroutine proc.directive.

Example::

 sudo qm -AQMTEST -INTERNAL

 PROGRAM HELLO
 $EXPLIST
 PRINT "Hello QMUSER, What is your name?: "
 INPUT UNAME
 PRINT UNAME: " Have a great day!" 
 END


 BASIC BP HELLO

HELLO.LIS::

         0000A5:       LD0
         0000A6:       LDSTR    "Hello QMUSER, What is your name?: "
         0000CA:       PRNL
      4  INPUT UNAME
         0000CB:       LDSLCL   UNAME
         0000CD:       LD0
      5  PRINT UNAME: " Have a great day!"
         0000CE:       LD0
         0000CF:       INPUT
         0000D1:       LD0
         0000D2:       LDSLCL   UNAME
         0000D4:       LDSTR    " Have a great day!"
         0000E8:       CAT
         0000E9:       PRNL
      6  END
         0000EA:       RETURN

INTERNAL Compiler Directives
============================

BCOMP compiler directives hardcoded into BCOMP routine PROC.DIRECTIVE:

EXPLIST - create OP Code Listing 

FLAGS - ALLOW.BREAK   sets object header flag hdr.allow.break

   CPROC           sets object header flag hdr.is.cproc
   
   DEBUGGER        sets object header flag hdr.is.debugger
   
   NETFILES        sets object header flag hdr.netfiles
   
   TRUSTED         sets object header flag hdr.is.trusted
   
INTERNAL - sets object header flag hdr.internal

RECURSIVE - sets object header flag hdr.recursive

  Note:  this seems to have somethinig to do with the generation of the OP Code objects stored in /usr/qmsys/bin/pcode

Adding Op Codes
===============
 Adding new statements requires an entry in the STATEMENTS list and a
 corresponding entry in the ON GOSUB that uses this list.

 Adding a new intrinsic function requires entries in the INTRINSICS and
 INTRINSIC.OPCODES lists and a corresponding entry in that ON GOSUB.

 New opcodes should be defined in the C opcodes.h include file. The equivalent
 QMBasic include record is generated using the OPGEN program.
 
 Note OPGEN looks for the c header file "opcodes.h" in file directroy CSRC
 
 CREATE.FILE DATA CSRC DIRECTORY PATHNAME /home/YOURUSERNAME/ScarletDME/gplsrc
 
 Note may need to edit created VOC to change .. /gplsrc/CSRC to ../gplsrc
 
 BIG NOTE If you copied opcodes.h over from a windows system to linux make sure you convert the file to linux style
 or OPGEN will fail!
 
 use something like dos2unix opcodes.h
 