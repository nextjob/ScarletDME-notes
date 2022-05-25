******
vbsrvr
******

QMCLIENT Command processor ../ScarletDME/GPL.BP/VBSRVR

Started with -Q command line option, which sets is_QMVbSrvr = TRUE in kernel
In start_Connection change the default command processor to VBSRVR via strcpy(command_processor, "$VBSRVR") 

Communicates with Client via READPKT and WRITEPKT  (both BCOMP internal mode functions)

op code for::

 READPKT    - OP.READPKT     /gplsc/op_tio.c    op_readpkt()  - Read QMClient data packet
                                                                           gplsrc\linuxio.c  read_socket
                                                                           gplsrc\linuxio.c  keyin 
                                                                           gplsrc\linuxio.c   do_input() ->  c std function read() from fd 0


 WRITEPKT   - OP.WRITEPKT    /gplsc/op_tio.c    op_writepkt()  -  Write QMClient data packet
                                                                           write_socket()  -  Write to socket / pipe
                                                                           to_outbuf()  -  Copy data to socket output buffer
                                                                           gplsrc\linuxio.c    flush_outbuf()  -  Flush socket output buffer
                                                                           gplsrc\linuxio.c    write_console  -> c std function write to fd 1



If one wanted to change the Client Server to JD3, would changing socket read / write to using READPKT / WRITEPKT do the trick?