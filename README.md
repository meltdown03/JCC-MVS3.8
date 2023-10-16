# JCC-MVS3.8

I got this from a MVS 3.8 running the TK4 distribution and packaged them into XMIT files.
Use IND$FILE and RECV to copy and unload them onto your system.

## UNLOAD JCL
These are the names to use for %DS%:
```
CNTL
INCLUDE
LINKLIB
OBJ
PROCLIB
RENT.OBJ
SRC
```

For each of the xmits, change the %DS% name in the XMITIN and SYSUT2 DD to match the current one:
You can also adjust the SPACE to less for all xmits except `JCC.LINKLIB`, `JCC.OBJ` and `JCC.RENT.OBJ`.
I need to make one job that does it all, and one XMIT file, but this is the quick and dirty way.

(You can substitute IBMUSER with whatever user you are using)
```jcl
//IBMUSERR JOB (RECV) 'RECEIVE FILE',CLASS=A,MSGCLASS=X              
//RECV370 EXEC PGM=RECV370,REGION=8196K                   
//STEPLIB   DD DSN=SYSC.LINKLIB,DISP=SHR *May not be needed or different on your system                
//RECVLOG   DD SYSOUT=*                                     
//SYSPRINT  DD SYSOUT=*                                     
//SYSIN     DD DUMMY                                        
//SYSUT1    DD UNIT=SYSDA,SPACE=(CYL,(10,10))               
//XMITIN    DD DSN=IBMUSER.JCC.%DS%.XMIT,DISP=SHR               
//SYSUT2    DD DSN=IBMUSER.JCC.%DS%,DISP=(NEW,CATLG,DELETE),
//             UNIT=SYSDA,SPACE=(TRK,(30,5,10))             
//
```
## INSTALL

They should go into the HLQ **JCC**, first make an alias using:

```
profile noprefix
define alias (name(JCC) rel(UCPUB000))
```
Respond to the console to allow altering the Master catalog if it asks, but it may not ask

You may have a different user catalog name, run `listcat` to see it

Then set your normal prefix back with: `profile prefix(IBMUSER)`

Next, just rename all the unloaded datasets and remove `IBMUSER.` from them so they are at HLQ JCC.
Another option is to change the SYSUT2 DD line in unload JCL above to just `JCC.%DS%` and then you don't have to
rename it.

Finally, copy members from `JCC.PROCLIB` to `SYS2.PROCLIB` or `SYSC.PROCLIB` or wherever you normally put your procedures.
```jcl
//IBMUSERC JOB (ACCNT) 'IEBCOPY',CLASS=A,MSGCLASS=X,NOTIFY=IBMUSER,REGION=2M 
//IEBCOPY  EXEC  PGM=IEBCOPY                                       
//SYSPRINT DD  SYSOUT=*                                            
//FROM     DD  DSN=JCC.PROCLIB,DISP=SHR                      
//TO       DD  DSN=SYS2.PROCLIB,DISP=SHR                            
//SYSUT3   DD  UNIT=SYSDA,SPACE=(TRK,(5))                          
//SYSUT4   DD  UNIT=SYSDA,SPACE=(TRK,(5))                          
//SYSIN    DD *                                                    
  COPY INDD=FROM,OUTDD=TO                                          
/*                                                                 
//                                                                 
```
## USING JCC
Look in JCC.CTL for some examples, but these don't seem to use the PROC's, so I would check
out the GCC examples and modify them or look on TK4/5 in `SYS2.JCLLIB`
```jcl
//IBMUSERJ JOB (JCC),                                                        
//            'Test JCCMVS',                                                    
//            CLASS=A,                                                          
//            MSGCLASS=X,                                                       
//            REGION=8M,NOTIFY=IBMUSER,                                    
//            MSGLEVEL=(1,1)                                                    
//S1      EXEC JCCCG                          
//SYSIN DD *                                                                    
#include <stdio.h>                                                              
#include <stdlib.h>                                                             
                                                                                
static void song( int bottles )                                                 
{                                                                               
 while( (printf(" %d bottles of beer on the wall, %d bottles of beer.\n"        
      " Take one down and pass it around, %d bottle%s of beer on the wall.\n\n",
         bottles, bottles, bottles-1, bottles>2? "s":""), bottles > 2) )        
  while( (--bottles,0) ) {}                                                     
 while( (puts(" 1 bottle of beer on the wall, 1 bottle of beer.\n"              
   " Take one down and pass it around, no more bottles of beer on the wall.\n\n"
      " No more bottles of beer on the wall, no more bottles of beer.\n"        
  " Go to the store and buy some more, 99 bottles of beer on the wall."),0) ) {}
}                                                                               
                                                                                
int main()                                                                      
{                                                                               
 while( (song(99), exit(0), 0) ) {}                                             
}                                                                               
/*                                                                              
//
```
