*****
cproc
*****
CPROC - Interactive Terminal Command Processor ../ScarletDME/GPL.BP/CPROC

Many of the system commands  (also known as Verbs) are nothing more than basic programs (however there are "internal" commands handled directly by CPROC, see list below).

These commands can be:

Commands  can be entered from a "terminal" and have an associated "VOC" file entry: 
VOC type "V" for Verb or locally catalogued subroutine.

+--------+--------+--------------------------------------------------------------+
| Field2 | Field3 |        Description                                           |
+========+========+==============================================================+
|   CA   | name   |     Catalogued verb                                          |
+--------+--------+--------------------------------------------------------------+
|   CS   | path   |     Locally catalogued function runfile path                 |
+--------+--------+--------------------------------------------------------------+
|   IN   | n      | Internal verb number n (see below for list of Internal Verbs)|
+--------+--------+--------------------------------------------------------------+
|   OS   | text   |     Operating system command                                 |
+--------+--------+--------------------------------------------------------------+
| Field4 = dispatch info (@option) for all verbs                                 |
+--------+--------+--------------------------------------------------------------+
| Field5 = security subroutine (optional)                                        |
+--------+--------+--------------------------------------------------------------+ 

Commands processed as part of a Paragraph (VOC type "PA" which in turn is made up of a list of commands) 


pseudocode::

 If this is the first time through CPROC (kernel(K$CPROC.LEVEL,0) ==0):
    initialize various things in the SYSCOM common block
    execute $LOGIN program
    if not valid login abort.cproc
    If phantom session:
        setup COMO file record for this session
        set phantom command options
    endif
    Execute MASTER.LOGIN paragraph
    Execute LOGIN paragraph

 else * Stacked CPROC, abort or LOGTO RESET
    get.voc.parser.rec (ln428)
    Inc CPROC stack depth ( kernel(K$CPROC.LEVEL,kernel(K$CPROC.LEVEL,0)+1) )
 endif (ln520)

 if EXECUTE command (flag setting in Object Header defined in header.h both basic and c versions) Not exactly sure how that gets set - needs investigating
    xeq.command is command name (stored in $syscom common block not sure how it gets there!, look at kernel((K$CPROC.LEVEL,..) see what it does
    
    executed via creating "local" voc record: voc.rec = 'PA':@fm:xeq.command) 
    calls execute.paragraph which ends up calling proc.para.sentence and eventually end up at execute.commad: (ln1406)
    goto abort.cproc (ln557) see below
            
        
 if is phantom process (ln562)
     Execute the phantom command
     gosub proc.sentence
     goto abort.cproc
  else
    this is a single command invocation of QM (user typed QM "LIST VOC" on os command line window).
        gosub proc.sentence
        goto abort.cproc
 end


 **
 * Main command processor loop
 **
  connected.port:

 Loop
    gosub reset.environment
    gosub set.links             ;* Unsnap subroutine links
    unload.object               ;* Unload inactive object code
    debug.off                   ;* Ensure debugger turned off
    debug.initialised = @false  ;* Force restart of debugger

    * Fetch new command
    loop
        gosub get.command.line which returns at.command
        until at.command # ""
        repeat
        new.sentence = at.command
        gosub proc.sentence 

 repeat  - loop repeats until some sort of abort or exit


 proc.para.sentence:
 proc.sentence: 
 proc.command:           note: these are all entry points that eventually end up at execute.command

    parse sentence and look for VOC record for command

 execute.commad:

 CASE VOC is TYPE:

    User Defined VOC type
        Process Command (CALL @handler) with handler defined in $VOC.PARSER record
        
    VERB    voc.entry.type[1,1] = "V" 

    Verbs 
        Type "CA" - call catalogued command stored in Field3 via @call
        Type "CS" - call Locally catalogued function via @call
        Type "IN" - internal CPROC command (see internal verbs list below)
        Type "OS" - OS command - gosub os.command
        Type "EX" - Executable - gosub run.exe  <-- need to look more into these two

    PROC voc.entry.type[1,2] = 'PQ' - old PROC PROCESSOR for R83  compatibility 

    REMOTE voc.entry.type[1,1] = 'R'

    SENTENCE voc.entry.type[1,1] = "S"

    PARAGRAPH voc.entry.type[1,2] = "PA" 

    MENU voc.entry.type[1,1] = "M"

    KEYWORD voc.entry.type[1,1] = "K"

    PRIVATE CATALOG ENTRY (BASIC PROGRAM  which is executed via creating local voc record: voc.rec = 'V':@fm:'CA':@fm:verb) and jumping back up to execute.commad (ln1406)

    GLOBAL CATALOG ENTRY  (BASIC PROGRAM  which is executed via creating local voc record: voc.rec = 'V':@fm:'CA':@fm:verb) and jumping back up to execute.commad (ln1406)

 CASE 1 - Error Message - not in VOC

 exit.command:
    If CPROC entry
        Dec CPROC level 
         i = kernel(K$CPROC.LEVEL,0)
         delete.common '$':i   ;* Delete unnamed common
         i = kernel(K$CPROC.LEVEL, i - 1)
 Return



 abort.cproc:
    dec command level with code:
        i = kernel(K$CPROC.LEVEL,0) - 1
        void kernel(K$CPROC.LEVEL,i)     ;* Decrement command level

 terminate.cproc:   return to terminate.cproc
    From Documentation:
        Sometimes a subroutine needs to return to the calling routine but it is not known how many internal subroutines may be active. 

            ERROR.LABEL: RETURN TO ERROR.LABEL

        This will cause all internal subroutines to return to the RETURN statement and then return to the calling program



 Internal Commands:

 Verb type = "IN"  ;* Internal CPROC command processed by corresponding CPROC internal subroutines:

      on voc.rec<3> gosub int.quit,  ;*  1  Quit (or OFF, see VOC entry for OFF)
         int.clr,                    ;*  2  Clear screen
         int.display,                ;*  3  Display text at terminal
         int.run,                    ;*  4  Run program
         int.abort,                  ;*  5  ABORT
         int.clearselect,            ;*  6  Clear select list
         int.date,                   ;*  7  DATE
         int.time,                   ;*  8  TIME
         int.break,                  ;*  9  BREAK
         int.bell,                   ;* 10  BELL
         int.go,                     ;* 11  GO
         int.status,                 ;* 12  STATUS
         int.set.date,               ;* 13  SET.DATE
         int.help,                   ;* 14  HELP
         int.update.account,         ;* 15  UPDATE.ACCOUNT
         int.who,                    ;* 16  WHO
         int.logto,                  ;* 17  LOGTO
         int.if,                     ;* 18  IF
         int.cleardata,              ;* 19  CLEARDATA
         int.clearprompts,           ;* 20  CLEARPROMPTS
         int.clear.stack,            ;* 21  CLEAR.STACK
         int.echo,                   ;* 22  ECHO
         int.hush,                   ;* 23  HUSH
         int.sleep,                  ;* 24  SLEEP
         int.clearinput,             ;* 25  CLEARINPUT
         int.clear.locks,            ;* 26  CLEAR.LOCKS
         int.lock,                   ;* 27  LOCK
         int.logout,                 ;* 28  LOGOUT
         int.debug,                  ;* 29  DEBUG
         int.stop,                   ;* 30  STOP
         int.report.src,             ;* 31  REPORT.SRC
         int.pterm,                  ;* 32  PTERM
         int.date.format,            ;* 33  DATE.FORMAT
         int.set,                    ;* 34  SET
         int.umask,                  ;* 35  UMASK
         int.pdump,                  ;* 36  PDUMP
         int.pause,                  ;* 37  PAUSE
         int.clear.abort,            ;* 38  CLEAR.ABORT
         int.set.exit.status,        ;* 39  SET.EXIT.STATUS
         int.report.style,           ;* 40  REPORT.STYLE
         int.logmsg                  ;* 41  LOGMSG

 End of CPROC description
