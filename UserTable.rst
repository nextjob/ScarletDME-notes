***********
User Table 
***********

User Table:: 

 typedef volatile struct USER_ENTRY USER_ENTRY;
 struct USER_ENTRY
 {
  int16_t uid;                   /* Internal user id. Zero if spare cell.
                                      -1 = reserved for new phantom */
  int32_t pid;                    /* OS process id */
  int16_t puid;                  /* Parent user id. Zero if not phantom */
  int32_t login_time;             /* qmtime() at login */
  char username[MAX_USERNAME_LEN+1];  /* Login user name */
  char ip_addr[15+1];
  #define MAX_TTYNAME_LEN 15
  char ttyname[MAX_TTYNAME_LEN+1];
  int16_t flags;                 /* Also defined in INT$KEYS.H */
     #define USR_PHANTOM     0x0001 /* Is a phantom */
     #define USR_LOGOUT      0x0002 /* Logout in progress */
     #define USR_QMVBSRVR    0x0004 /* Is QMVbSrvr process */
     #define USR_ADMIN       0x0008 /* Administrator privileges */
     #define USR_QMNET       0x0010 /* Is QMNet (USR_QMVBSRVR also set) */
     #define USR_CHGPHANT    0x0020 /* "Chargeable" phantom; counts as licensed user */
     #define USR_MSG_OFF     0x0040 /* Message reception disabled */
     #define USR_WAKE        0x0080 /* Set by op_wake, cleared by op_pause */
  u_int16_t events;        /* Any bit set causes processing interrupt */
     #define EVT_LOGOUT      0x0001 /* Forced logout - immediate termination */
     #define EVT_STATUS      0x0002 /* Return status dump */
     #define EVT_UNLOAD      0x0004 /* Unload inactive cached object code */
     #define EVT_BREAK       0x0008 /* Set break inhibit to zero */
     #define EVT_HSM_ON      0x0010 /* Start HSM */
     #define EVT_HSM_DUMP    0x0020 /* Return HSM data */
     #define EVT_PDUMP       0x0040 /* Force process dump */
     #define EVT_FLUSH_CACHE 0x0080 /* Flush DH cache */
     #define EVT_JNL_SWITCH  0x0100 /* Switch journal file */
     #define EVT_TERMINATE   0x0200 /* Forced logout - graceful termination */
     #define EVT_MESSAGE     0x0400 /* Send immediate message */
     #define EVT_LICENCE     0x0800 /* Logout from licence expiry */
     #define EVT_REBUILD_LLT 0x1000 /* Rebuild local lock table */
  /* Lock wait data (protected by REC_LOCK_SEM) */

   int16_t lockwait_index;       /* 0 = not waiting,
                                      +ve = rec lock table index (record lock),
                                      -ve = file table index (file lock) */
   u_int16_t file_map[1];          /* Count of opens by file.
                                               Protected by FILE_TABLE_LOCK */
 };

 UMap(n)    Returns pointer to user map entry for user n 
 UserPtr(n) Returns pointer for table entry for user n 
 UPtr(n)    Returns pointer for table index n, not user n (from 1) 
 UFMPtr(uptr,fno)  Pointer to user/file map table entry. Fno from 1 

Public USER_ENTRY * my_uptr init(NULL);