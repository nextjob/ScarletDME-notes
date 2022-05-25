***********
pcode file
***********

Pre Compiled OP Code objects which reside in the "pcode" file /usr/qmsys/bin/pcode

The pcode file appears to be made up of the below list of compiled basic programs (in GPL.BP).
(All have RECURSIVE Compiler Directive, when this is set, BCOMP adds the OP Code object for the unit to the "pcode" file.)

RECURSIVE seems to have a special meaning in QM, it defines an op code object that can be executed (called) by a c op code.

From Martin Phillips groups.google.com - openqm "QM is written in QMBasic":

 "The clever bit is that some things that look like simple operations that might be expected to be in the C kernel of QM are actually written in QMBasic.
  For example, opening a file   requires QM to extract the pathname from the corresponding VOC record.
  It would not be difficult to write a chunk of C code that reads the VOC record and extracts the relevant field.
  But, add the complexities of Q-pointers, multifiles, distributed files, QMNet references, etc and this becomes quite complex.
  Instead, the opcode that does the open executes a bit of QMBasic to resolve the filename into a pathname. 
  It's not quite the same as a subroutine call as it occurs internally in the middle of the open opcode.
 
  There are many of these so called recursive QMBasic functions.
  Some are very short, others are quite long. Another example is the INPUT statement.
  This is even more interesting as it uses KEYCODE() which is itself a further recursive program.
 
  Recursive QMBasic is totally an internal feature.
  It makes development and maintenance of QM much quicker and simpler than writing the same functionality in C."

From Martin Phillips groups.google.com - openqm "Wood Tick's Guide to Hacking openQM":

 "Hi all,

 Let's clarify things a bit....

 The QM kernel is written in C. This includes the obvious opcodes such as
 arithmetic and string functions, file handling, etc as well as things such
 as the socket interface referenced in Symeon's posting.

 The command language, query processor, and even the QMBasic compiler are
 written in QMBasic (interesting "chicken and egg" problem). User feedback
 suggests that there is no performance problem with any of this. Our own
 tests show that we are slightly faster in a typical application mix than
 some of the mainstream multivalue players and a bit slower than others but
 not significantly. It is always possible to pick specific architectural
 differences where one environment will out perform another so let's not try
 to quantify performance in detail.

 For those who have not delved into the internals of QM (and we don't really
 recommend it!), we have a delightful concept called recursive Basic. As a
 simple example, consider the process of opening a file. The OPEN opcode must
 decode exactly what it is being asked to do, process the corresponding VOC
 record to resolve pathnames, Q-pointers, etc, and then do the actual open. 
 Although this could all be done in C, the VOC record processing is far
 easier in Basic. So, what we have is a way in which a C opcode can
 effectively call a chunk of Basic in the middle of its operation.

 If that example isn't complex enough for you, the INPUT opcode (a single
 byte in the object code), recurses into a 400 line QMBasic program
 (including comments/blank lines - I cannot be bothered to count the "real"
 lines for this discussion). This program uses the KEYCODE() function to read
 and decode a keystroke based on terminal type and the terminfo system. 
 KEYCODE is also a recursive operation (130 lines). So INPUT actually
 recurses inside a recursive - Wow!

 The kernel consists of nearly 120,000 lines of C but this includes vast
 chunks of comments, blank lines, and lots of conditional stuff that doesn't
 appear in the open source version. The QMBasic component is actually
 slightly smaller but, of course, a single line of Basic does much more than
 a single line of C.

 Our design aim was to write as much of the system in Basic as possible. We 
 would also recommend that any open source development should be done in
 Basic where possible because (a) it is a more widely understood language
 within the multivalue community, (b) it tends to be much faster to write,
 (c) maintenance and modification is easier, and (d) it is usually much
 easier to debug.

 Taking Symeon's point about performance, one reason why things work so well
 in Basic is that we provided a few additional internal functions to help
 specific areas that might not be so good written in the "public" parts of 
 QMBasic. These tend not to be available for user programs as they may change
 from one release to the next.

 For those not interested in open source, what language is used to write the
 system should be irrelevant so long as it works effectively."

pcode file::

  _AK
  _BANNER
  _BINDKEY
  _BREAK
  _CCONV
  _CHAIN
  _DATA
  _DELLIST
  _EXTENDLIS
  _FMTS
  _FOLD
  _FORMCSV
  _FORMLST
  _GETLIST
  _GETMSG
  _HF
  _ICONV
  _ICONVS
  _IN
  _INDICES
  _INPUT
  _INPUTAT
  _ITYPE
  _KEYCODE
  _KEYEDIT
  _LOGIN
  _MAXIMUM
  _MESSAGE
  _MINIMUM
  _MSGARGS
  _NEXTPTR
  _OCONV
  _OCONVS
  _OJOIN
  _OVERLAY
  _PCLSTART
  _PICKMSG
  _PREFIX
  _PRFILE
  _READLST
  _READV
  _REPADD
  _REPCAT
  _REPDIV
  _REPMUL
  _REPSUB
  _REPSUBST
  _SAVELST
  _SSELCT
  _SUBST
  _SUBSTRN
  _SUM
  _SUMALL
  _SYSTEM
  _TCONV
  _TRANS
  _TTYGET
  _TTYSET
