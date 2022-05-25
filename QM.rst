****
qm.c
****

Main Module of ScarletDME

pseudocode::

 main()

   qm.init() - allocates memory for current working directory and saves string in entry_dir (kernal.h)
    
    test for CMD_FLASH (only succeeds if command_options set to CMD_FLASH at compile???)
    
    comlin() - process qm command line options  sets command_options and connection_type
                note connection_type = CN_CONSOLE (as set by kernal.h)
                
            Parse Command line options and set flags / process:
                
               If -start 
                qmstart()
                    Creates shared memory segment via bind_sysseg(TRUE, errmsg)
                    forks the current process and replaces the child process with qmlnxd and runs as a daemon
                    if there is a startup command defined (sysseg->startup[0] != '\0)
                        forks again and runs command in that child process
                    exit
               if -stop 
                stop_qm()
                exit    
                
               if -P 
                 is_phantom = TRUE (FALSE as set by kernal.h) 
                 connection_type = CN_NONE
                 
               if -Q 
                is_QMVbSrvr = TRUE (FALSE as set by kernal.h)
                telnet_binary_mode_in = TRUE;
                telnet_binary_mode_out = TRUE;
                
               if -C
                connection_type = CN_PIPE
                
               if -N
                connection_type = CN_SOCKET
                
               if connection_type CN_SOCKET
                start_connection(socket_handle)
                
                
            if connection type CN_SOCKET
                start_connection(socket_handle)) 
                Note if is_QMVbSrvr = TRUE we change the default command processor to VBSRVR
                    via strcpy(command_processor, "$VBSRVR") in start_Connection
                    if one wanted to change the Client Server to JD3, (require some rewrite of D3Client) this would be the ticket!
            else if connection type CN_PORT 
                start_connection(0)
                
                
    bind_sysseg(false - this is not -qmstart)
            make copy of configuration parameters for process (private version - can be changed by process)
    
    load pcode object code via macro  Pcode(a) and pcode.h
    
    init_kernel() initialize data areas - see detail in kernel
                    
    
    load_language()
    
    kernal()  - 
    
        k_call to start $CPROC (or $VBSRVR  if is_QMVbSrvr, see start_connection above) k_call detail in kernel{linkID=300}
        
        Note: setjmp(k_exit) landing spot for  Abort, Quit, Logout  (set via calls to k_error / longjmp() )
        
        k_run_program() see detail in kernel
            
     s_free_all()
    
    clean_stop()
    
return	

