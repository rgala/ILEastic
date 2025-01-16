[![Open in Visual Studio Code](https://open.vscode.dev/badges/open-in-vscode.svg)](https://open.vscode.dev/sitemule/ILEastic)
# ILEastic
It is a self contained web application server for the ILE environment on IBM i 
to run microservices.

ILEastic is a service program that provides a simple, blazing fast programmable 
HTTP server for your application. You can easily plug your RPG code into a services 
infrastructure and make simple web applications without the need for any third party 
webserver products.

Basically it is an HTTP application server you can bind into your own ILE RPG 
projects, to give you an easy deploy mechanism, that fits into DevOps and 
microservices environments alike.

The self contained web application server makes it so much easier to develop 
web applications.

Simply compile and submit. No - You don't need CGI, Apache, nginx or IceBreak - 
simply compile and submit.

The design paradigm is the same as found in Node.JS - the project was initially 
called Node.RPG but the name was subject to some discussion, so ILEastic it is.
Where Node.JS uses JavaScript, ILEastic aims for any ILE language where RPG is
the most popular.

Except for initialization, it only requires two lines of code:
```
 il_listen ( config : pServletCallback); 
 il_responseWrite ( pResponse);
```

The `il_listen` is listening on the TCP/IP port and interface you define in the 
config structure. For each HTTP request it will call your "servlet" which is a 
callback procedure that takes a request and a response parameter.

![](image.png)


The idea is that you deploy your (open source of course) RPG packages at NPM so 
the RPG community can benefit from each other's work. The NPM ecosystem is the 
same for Node.JS and ILEastic.    


Example: 
```
**FREE

// -----------------------------------------------------------------------------
// This example runs a simple servlet using ILEastic
// Note: It requires your RPG code to be reentrant and compiled
// for multithreading. Each client request is handled by a seperate thread.
// Start it:
// SBMJOB CMD(CALL PGM(DEMO01)) JOB(ILEASTIC) JOBQ(QSYSNOMAX) ALWMLTTHD(*YES)        
// -----------------------------------------------------------------------------     
ctl-opt copyright('Sitemule.com  (C), 2018');
ctl-opt decEdit('0,') datEdit(*YMD.) main(main);
ctl-opt debug(*yes) bndDir('ILEASTIC');
ctl-opt thread(*CONCURRENT);
/include include/ileastic.rpgle
// -----------------------------------------------------------------------------
// Main
// -----------------------------------------------------------------------------     
dcl-proc main;

    dcl-ds config likeds(IL_CONFIG);

    config.port = 44001;
    config.host = '*ANY';

    il_listen (config : %paddr(myservlet));

end-proc;
// -----------------------------------------------------------------------------
// Servlet call back implementation
// -----------------------------------------------------------------------------     
dcl-proc myservlet;

    dcl-pi *n;
        request  likeds(IL_REQUEST);
        response likeds(IL_RESPONSE);
    end-pi;
  
    il_responseWrite(response:'Hello world');

end-proc;
```
# Your IBM i
In this project there are lots of references to `my_ibm_i` both in code, development tool and test.
This is of course the name of your IBM i. You can do yourself a favor and add the name `my_ibm_i` to 
your `hosts` file and let it point to the IP address of your IBM i - and all the code, 
development tool and test will work out of the box.

[Edit host file][eh]



# Installation
What you need before you start:

* IBM i 7.2 TR9 (or higher)
* YUM installed from ACS (to install: git and make-gnu (gmake))
* ILE C
* ILE RPG compiler

Also ensure that the open source tools are available in your path, according to this:

[set open source path](https://ibmi-oss-docs.readthedocs.io/en/latest/troubleshooting/SETTING_PATH.html)


From a IBM i menu prompt start the SSH daemon: `===> STRTCPSVR *SSHD`
and start SSH from Win/Mac/Linux.

First install the open source tools:
```
ssh my_ibm_i
yum install git
yum install make-gnu
```
Now you are ready to clone the ILEastic git repo: 

```
mkdir /prj
cd /prj 
git -c http.sslVerify=false clone --recurse-submodules https://github.com/sitemule/ILEastic.git
cd ILEastic
gmake  
```
Now you have library ILEastic on your IBM i - and you are good to go. You can simply copy the service programs
to you own project libraries along with the binding directory and header files. (You can skip the *MODULE objects.)

If you'd like to try the examples then you need to build them as well - as simple as:

```
cd examples 
gmake
```

# Test it:
Log on to your IBM i.
From an IBM i menu prompt 
```
CALL QCMD
ADDLIBLE ILEASTIC
SBMJOB CMD(CALL PGM(helloworld)) ALWMLTTHD(*YES) JOB(helloworld) JOBQ(QSYSNOMAX) 
SBMJOB CMD(CALL PGM(staticfile)) ALWMLTTHD(*YES) JOB(staticfile) JOBQ(QSYSNOMAX) 
```
Look for the complete list in the examples folder - and observe which port they are "listening" at.


Now test it in a browser: 

* http://my_ibm_i:44000  Hello world
* http://my_ibm_i:44012  Simple website demo


Please note that the job requires `ALWMLTTHD(*YES)`.


# Develop
You compile the project with gmake, and I have also included a setup folder for
vsCode so you can compile any changes with `Ctrl-Shift-B`. You need however to add
the name `my_ibm_i` to your host file since the .vsCode/tast.json file refers to this name.
The compile feature also requires that you have SSH started: `STRTCPSVR *SSHD`

# Unit Tests
For executing the unit tests located in the folder _unittests_ you need to 
previously install either [iRPGUnit][iru] or [RPGUnit][ru].

# Moving on
So far we have implemented the basic features like `il_listen`, `il_responseWrite` and
`il_addRoute` - look at the prototypes in `ILEastic.rpgle` header file for the complete 
list of features. There is still much work to do, however - all the plumbing 
around git / compile / deploy is working. We at Sitemule.com are striving 
to move the core of the IceBreak server into the ILEastic project over the next 
couple of months. So stay tuned.


# Note
This project was initially call Node.RPG, however people could not find the 
node.js code :) so obvously it was a bad name. Thanks for the feedback pointing 
me into a better direction.

Happy ILEastic coding!

[iru]: https://irpgunit.sourceforge.io
[ru]: https://rpgunit.sourceforge.io
[eh]: https://www.howtogeek.com/howto/27350/beginner-geek-how-to-edit-your-hosts-file/

