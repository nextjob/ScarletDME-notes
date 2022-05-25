********
kernel.c
********

KERNEL.C - Run Machine Kernel

Data Structures

Program Structure::

 Created for invoked programs in k_call
 
 struct PROGRAM {
  struct PROGRAM* prev; **** IMPORTANT , link to nested program invocation 
  int16_t no_vars;
  DESCRIPTOR* vars;
  u_int32_t flags;
 /* MS 16 bits, kernel specific; LS 16 bits from object header */
 #define IS_EXECUTE 0x00010000L /* Started via EXECUTE */
 #define IS_CLEXEC 0x00020000L  /* Is pseudo CPROC from EXECUTE CURRENT.LEVEL */
 #define IGNORE_ABORTS 0x00040000L  /* Ignore aborts from EXECUTEd sentence */
 #define PF_IS_TRIGGER 0x00080000L  /* Is trigger program */
 #define SORT_ACTIVE 0x00100000L    /* Program has sort in progress */
 #define PF_IS_VFS 0x00200000L      /* Is VFS handler */
 #define PF_CAPTURING 0x00400000L   /* Capture data stacked for this CPROC */
 #define PF_IN_TRIGGER 0x00800000L  /* This or lower program is a trigger */
 #define PF_PRINTER_ON 0x01000000L  /* PRINTER ON? */
 #define FLAG_COPY_MASK 0x01800000L /* Copy these flags to called program */
  int32_t col1;
  int32_t col2;
  int16_t precision;
  u_char* saved_c_base;
  int32_t saved_pc_offset;
  char saved_prompt_char;
  STRING_CHUNK* saved_capture_head;
  STRING_CHUNK* saved_capture_tail;
  char* break_handler; /* Break handler name */
  OBJDATA* objdata;
 #define MAX_GOSUB_DEPTH 256
  int32_t gosub_stack[MAX_GOSUB_DEPTH];
  int16_t gosub_depth;
  u_char arg_ct;         /* Number of arguments passed */
  int16_t e_stack_depth; /* Depth on entry to program */
 };



PROCESS Structure::

 Only one of these exist, defined and created in kernel.h. 
  Note it contains a pointer to the "Current" program's PROGRAM structure
 
 struct PROCESS {
  /* Dispatch loop control */
  int16_t k_abort_code; /* @ABORT.CODE value */

  int16_t user_no; /* -1 for cleanup, -2 for qmlnxd */
  char username[MAX_USERNAME_LEN + 1];

  /* Program control */
  struct PROGRAM program; /* Current program state */
  int call_depth;
  bool debugging; /* Debugger active? */
  /* Common areas */
  ARRAY_HEADER* named_common; /* Head of named common header chain */
  ARRAY_HEADER* syscom;       /* $SYSCOM (also in named_common) */
  /* Opcode actions */
  bool for_init;          /* Used by FORINIT and FORTEST */
  int16_t break_inhibits; /* Count of BREAK OFF calls */
  u_int16_t op_flags;     /* Opcode prefix flags */
 /* Exact meaning of these flags is opcode dependent, especially the top byte */
 #define P_ON_ERROR 0x0001 /* ONERROR opcode executed */
 #define P_LOCKED 0x0002   /* NOWAIT opcode executed */
 #define P_LLOCK 0x0004    /* LLOCK opcode executed       ** See int$keys.h */
 #define P_ULOCK 0x0008    /* ULOCK opcode executed       ** See int$keys.h */
 #define P_READONLY 0x0010 /* READONLY opcode executed */
 #define P_PICKREAD 0x0020 /* PICKREAD opcode executed */
 #define P_REC_LOCKS (P_LLOCK | P_ULOCK) /* Either LLOCK or ULOCK */
  bool numeric_array_allowed;           /* Opcode allows numeric arrays? */
  int32_t status;                       /* Value from STATUS() function */
  int32_t inmat;                        /* Value of INMAT() function */
  int32_t os_error;                     /* Operating system error number */
  u_int32_t txn_id; /* Transaction id. 0 if none */
 };
 
Vars of Interest::
 
 Public void* object;   /* Pointer to current OBJECT */
 Public u_char* c_base; /* Base address of current object code */
 Public u_char* pc;     /* Next opcode byte */
 Public u_char* op_pc;  /* Address of current opcode */



init_kernel pseudocode::

 init_kernel()
        dio_init() - init selection arrays and record_cache
        zero PROCESS structure process
        c_base = NULL
        init_program - Initialize PROGRAM structure in process
        null pointers to common header chain
          process.named_common = NULL
          process.syscom = NULL
        tio_init - initialize terminal i/o
          if connection_type CN_NONE or QMVbSrvr
             do nothing
          else
             init_console()
        if GetUserName  & Password
            populate USER_ENTRY structure (pointed to by my_uptr)
        else
            fail login message return status false
        return
        
kernel pseudocode::
 
 kernel()
        setup signal handlers
          signal(SIGSEGV, fatal_signal_handler);
          signal(SIGILL, fatal_signal_handler);
          signal(SIGBUS, fatal_signal_handler);
          signal(SIGCHLD, sigchld_handler);
          signal(SIGUSR1, sigusr1_handler);
        load the command processor opcode object
          k_call(command_processor, 0, NULL, 0);
        Setup error processing path
          setjmp(k_exit)) /* Abort, Quit, Logout *
        recursion_depth = -1 ??
        Run the command processor
          k_run_program()
        exit_kernel:
          como_close();
        return;
        
                
k_call pseudocode::
                
 k_call(char* name, int num_args, u_char* code_ptr, int16_t stack_adj)
        If the code_ptr argument is null, we perform a search for the object "name"
        Otherwise we simply call the object at that address.        
        if code_ptr == null 
            Dynamically loaded object 
            hdr = pointer to loaded object  (load_object() in object.c)
        else 
            hdr = pointer code_ptr
        if we are here processing a call from an parent / nested process (c_base != NULL)  
            save previous PROGRAM state
        process.call_depth++;   
        init_program - Initialize PROGRAM structure on first entry or CALL
        set c_base (points to program / object hdr structure)
        Setup Hot Spot Monitor in hsm hsm = true
        Calculate and allocate space required for program variables in descriptor area
        Copy arguments currently on evaluation stack (e-stack) into new process.program.vars
        if necessary resize e-stack 
            allocate memory for new  e-stack
            copy existing e-stack items and free old stack 
        return
 
        k_call notes: Called from xxxx at system start up to execute command processor (CPROC or VBSRVR)
          Called in op_codes op_call() and op_callv() to execute external subprograms (BASIC CALL statement).
        
k_run_program pseudocode::
        
 k_run_program - Dispatch Loop
        Run (interpret) the object code via the loop:
        do {
        while (!k_exit_cause) {
            dispatch[*(op_pc = pc++)]();  
               dereference the the address location op_pc returning the OP Code index value into the dispatch array and call the p-code function stored there
        }
        Where op_pc is a pointer to the start of the OP Code in the object record (OP Code Object{linkID=650})
    
        How this works:
        
        There are two macro calls used in kernel.c that:
            
        1) Build the op_code function prototypes:

        /* Declare opcode functions */

        #define _opc_(code, key, name, func, format, stack_use) void func(void);
        #include "opcodes.h"
        #undef _opc_
            
        That result in the function prototype being generated for each entry in opcodes.h:
            
        void op_stop(void);
        void op_abort(void);
        void op_return(void);
                .
                .
        
                
        2) Build the dispatch table array:
            
        #define _opc_(code, key, name, func, format, stack_use) func,
        void (*dispatch[])(void) = {
        #include "opcodes.h"
        };
        #undef _opc_
            
        That results in an element for each op code function
            
        void (*dispatch[])(void) = {
        op_stop,
        op_abort,
        op_return,
             .
             .
        }
            
        We now have a mechanism to to invoke each op code function via it's index into the dispatch array.
        These are the same numbers used to refernce OP Code functions used by the basic compiler when building
        the program object record (see /GPL.BP/BCOMP)

===================================
Accessing Kernel Varaibles in Basic
===================================
  BASIC programs Set / Get Kernel variables via the intrinsic function kernel() op_kernel.c.
  
  Samples::
  
    i = kernel(K$CPROC.LEVEL,0)    returns CPROC nesting level 
          note: it appears a value of 0 returns the current value
            a value > 0 sets the variable to the new value

    i = kernel(K$CPROC.LEVEL,1) sets CPROC nesting level to 1
    
    Keys:
     K$INTERNAL           Set or clear internal mode
     K$INTERNAL.QUERY     Query internal mode
     K$PAGINATE           Test or modify pagination flag
     K$FLAGS              Test/return program header flags
     K$DATE.FORMAT        European date format?
     K$CRTWIDE            Return display width
     K$CRTHIGH            Return display lines per page
     K$SET.DATE           Set current date
     K$IS.PHANTOM         Is this a phantom process
     K$TERM.TYPE          Terminal type name
     K$USERNAME           User name
     K$DATE.CONV          Set default date conversion
     K$PPID               Get parent process id
     K$USERS              Get user list
     K$INIPATH            Get ini file pathname
     K$FORCED.ACCOUNT     Force entry to named account unless set in $LOGINS
     K$QMNET              Get/set QMNet status flag
     K$CPROC.LEVEL        Get/set command processor level
     K$HELP               Invoke help system
     K$SUPPRESS.COMO      Supress/resume como file output
     K$ADMINISTRATOR      Get/set administrator rights
     K$SECURE             Secure system?
     K$GET.OPTIONS        Get options flags
     K$SET.OPTIONS        Set options flags
     K$PRIVATE.CATALOGUE  Set private catalogue pathname
     K$CLEANUP            Clean up defunct users
     K$COMMAND.OPTIONS    Get command line option flags
     K$CASE.SENSITIVE     REMOVE.TOKEN() cases sensitivity
     K$SET.LANGUAGE       Set language for message handler
     K$COLLATION          Set/clear sort collation data
     K$GET.QMNET.CONNECTIONS  Get details of open QMNet connections
     K$INVALIDATE.OBJECT  Invalidate object cache
     K$MESSAGE            Enable/disable message reception
     K$SET.EXIT.CAUSE     Set k_exit_cause
     K$COLLATION.NAME     Set primary collation map name
     K$AK.COLLATION       Select AK collation map
     K$EXIT.STATUS        Set exit status
     K$AUTOLOGOUT         Set/retrieve autologout period
     K$MAP.DIR.IDS        Enable/disable dir file id mapping
	


    
