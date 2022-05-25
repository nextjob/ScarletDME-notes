********
qmEngine
********

THIS IS A WORK IN PROGRESS (good luck to all!)

qmEngine is a shared library containing an ?imbedded? ScarletDME system.
The qmEngine and the calling code all execute in the same process space. 
 
API / Functions exposed to user process::

  int qmEng_init(void)
    initialize qmEngine and start our command processor ENGSRVR
    initializes Vars used to communicate with VM

 File Handling
  int qmEng_QMOpen(char* filename)
  void qmEng_Close(int fno)
  char* qmEng_Read(int fno, char* id, int* err)
  char* qmEng_Readu(int fno, char* id, int* err) <- note we do not give the wait option like qmclient, let call figure out what they want to do.
  void qmEng_QMWrite(int fno, char* id, char* data) 
  int qmEng_Delete()
  int qmEng_Deleteu()
  int qmEng_lock()
  int qmEng_Release()
  
      qmEng_EXECUTE
      qmEng_CALL
      qmEng_SELECT
      qmEng_READNEXT
      qmEng_SELECT_TCL
      
      
 int qmEng_finish(void)
   free any buffers pointed to by EngLoadBuff / EngStoreBuff
     s_free_all(); 
     clean_stop(); 

Function and vars not exposed to user::

 int qmEng_exe(int command_id)
        execudes command specified, returning status
        
 File Handling Vars
 
 int EngFileNbr - File number 
 char EngFilename[255]
 char EngItemId[255]
 char *EngStrBuff -  String buffer may be resized (delete / reallocated) by OP_ENGMVDATA
 int   EngStrBsz  -  size of buffer
 int   EngCmd     -  EngSrvr command id (mimic VBSRVR?)
 int   EngYieldFlg - has VM yielded to engine (set in OP_ENGYIELD)
 int   EngCmdStat   - returned status of EngSrvr command (set in each routine that processes a command in EngSrvr)


New op codes added to VM / BCOMP::

  ENG.MVDATA(VAR,I) Move data from / to qmEng and EngSrvr
     VAR - VM Variable (BASIC)
     KEY Int Value that specifies qmEng Variable and direction
     
      VAR = ENG.MVDATA(DMMY,key)
                   key
    -------------  ---             
     ENG_LD_CMD  -  1 Take value in EngCmd and place in VAR (ie acts as command read)
     ENG_LD_FNBR - 10 Take the value in EngFileNbr and place in VAR (from previous open)
     ENG_LD_FNAM - 11 Take string in EngFileName and place in VAR 
     ENG_LD_ID   - 12 Take string in EngItemID and place in VAR
     ENG_LD_BUFF - 20 Take the value in buffer pointed to by EngLdBuff and store into qmBasic variable VAR
     
     DMMY = ENG.MVDATA(VAR,key)
                   key
     -----------   ---
     ENG_ST_STAT - 101 Take value in VAR  and place in EngCmdStat (ie acts as status write)
     ENG_ST_FNBR - 110 Take the value and place in EngFileNbr (from openfile)
     ENG_ST_BUFF - 120 Take the value in qmBasic variable VAR and store in buffer pointed to by EngStrBuff 
                    note - String buffer may be resized (delete then reallocated) by OP_ENGMVDATA when necessary (updates EngStrBsz)
    OP_ENGMVDATA in engine.c

  
 ENG.YIELD -yield VM -  basically a clone of op_pause (see pause command in qm doc)
   ENG.YIELD: on entry the process's wake flag is evaluated:
   while USR_WAKE flag not set do
      (Note Normal state of wake flag is 0, so on entry if zero we wait until it is set to 1)
     process_events
     test for and break on interrupt
     test for and break on break key
     sleep(1)
   loop
   set process status flag to 0
   clear wake flag

VM / EngSrvr::


  The command processor ENGSRVR will run in the VM (patterned off of VBSRVR)
  Kicked off as part of the qmEng_init process 
  EngSrvr starts up and runs until it hits the op_EngYield 
  
  ENGSRVR code:
  
  LOOP
    If First.Pass then
      NULL  * no command to process
    Else
      GOSUB PROCESS ENGSRVR_CMMD_ID
      *  PROCESSES COMMAND AND POPULATES C VARIALBE VIA ENG.MVDATA 
      *  
    End
    ENG.YIELD
       VM Yields until  qmEng_exe sends another request and sets the wake flag (my_uptr-> |= USR_WAKE;) 
       VM Resumes
    ENGSRVR_CMMD_ID = ENG.MVDATA(dmmy,ENG_LD_CMMD)
    WHILE ENGSRVR_CMMD_ID NE QUIT.SERVER
    REPEAT
   
    GOSUB QUIT.SERVER
    RETURN or END or how ever command processor terminates

Tricky part:
  We run in 2 threads
    Thread 1 API 
    Thread 2 VM
    


Building qmEngine::

  make -f EngMakeFile 
  
  compile qmEngTest
  
  echo "compile qmEngTest and link with libqmEng.so"
  gcc -L./Engobj -Wall -Wformat=2 -Wno-format-nonliteral -DLINUX -D_FILE_OFFSET_BITS=64 -I ./gplsrc/ -DGPL -g  ./gplsrc/qmEngTest.c -o qmEngTest -lqmEng -lpthread

  Rem! must export lib path 
   
  export LD_LIBRARY_PATH=/home/USERnameHere/ScarletDME/Engobj:$LD_LIBRARY_PATH
  echo $LD_LIBRARY_PATH
  
  Look at tail end of syslog
  tail -f /var/log/syslog
      
