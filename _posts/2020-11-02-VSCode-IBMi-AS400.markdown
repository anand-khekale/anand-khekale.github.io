---
layout: single
classes: wide 
title: How to use Visual Studio Code while working on IBM i
header:
  og_image: /assets/images/forposts/vscode-ibmi/vscode_header.png
categories: 
 - ibmi 
 - vscode
 - code editor 
tags: 
 - code
 - editor
 - vscode
 - as400
permalink: /:year/:month/:title:output_ext
---
![vscode-ibmi]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/vscode-ibmi/vscode_header.png){: .align-center}  

In this article, let's see how we can use the Visual Studio Code (VS Code) while working on IBM i. This versatile editor is used for writing programs with languages like Java, JavaScript, Python, and many more, but how we can leverage it for writing RPGLE, SQLRPGLE, CLLE code is also important.  
{: style="text-align: justify;"}
In recent months VS Code has emerged as the most preferred code editor in the programming fraternity. It is an open-source, free, lightweight code editor from Microsoft. This editor has tons of extensions, themes, customization, and powerful language editing features.  
{: style="text-align: justify;"}
This article also covers the extensions, themes, and settings needed for the program to be written directly on the IBM i system, for its final compilation/deployment.
{: style="text-align: justify;"}
> Note: The code for this article can be downloaded from [here](https://github.com/anand-khekale/vscode-ibmi/archive/master.zip).

## How to install VS Code

Visit the [Visual Studio Code](https://code.visualstudio.com/) site, click on the download button for your operating system, once downloaded, install it like any other software program; it is easy and straight forward. I use their stable version. 
{: style="text-align: justify;"}
There is detailed documentation on how to [get started](https://code.visualstudio.com/docs) on it. 

![vscode-ibmi]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/vscode-ibmi/vscode_download.png){: .align-center} 

## Extensions in VS Code

VS Code provides many extensions that help improve code writing efficiency. I have given some of the extensions below to showcase its usefulness.
{: style="text-align: justify;"}
### 1. IBM i Languages by barrettotte

[IBM i Languages](https://marketplace.visualstudio.com/items?itemName=barrettotte.ibmi-languages) is an extension for the majority of native languages like RPG, CL, DDS, and RPGLE fixed/free. This extension provides syntax highlighting, keyword prompting, etc. See the snapshots below, where all the opcodes are highlighted. 
{: style="text-align: justify;"}
This is for RPGLE Free.

![vscode-ibmi]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/vscode-ibmi/vscode_ibmi_rpgle.png){: .align-center} 

This is for CLLE.

![vscode-ibmi]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/vscode-ibmi/vscode_ibmi_clle.png){: .align-center} 

And one for DDS.

![vscode-ibmi]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/vscode-ibmi/vscode_ibmi_dds.png){: .align-center} 

### 2. IBM Z Open Editor

This [COBOL](https://marketplace.visualstudio.com/items?itemName=IBM.zopeneditor) extension is so powerful, with few keystrokes, most of your program gets generated. Let's see, when I start typing IDENT and hit Enter, the entire IDENTIFICATION DIVISION gets generated using the snippets provided by the extension.
{: style="text-align: justify;"}
![vscode-ibmi]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/vscode-ibmi/vscode_cobol.gif){: .align-center} 

### 3. Extensions/Themes for Open-source Languages

The below table summarizes some of the extensions/themes I use, which help me a lot when writing a program.

|Extension|Description|
|--------|:-------|
|Bracket Pair Colorizer|  A customizable extension for colorizing matching brackets - [Link](https://marketplace.visualstudio.com/items?itemName=CoenraadS.bracket-pair-colorizer).|
|Code Spell Checker|  Spelling checker for source code - [Link](https://marketplace.visualstudio.com/items?itemName=streetsidesoftware.code-spell-checker).|
|Indenticator|    Highlights your current indent depth - [Link](https://marketplace.visualstudio.com/items?itemName=SirTori.indenticator).|
|One Dark Pro Theme|  Atom's iconic One Dark theme for Visual Studio Code - [Link](https://marketplace.visualstudio.com/items?itemName=zhuangtongfa.Material-theme).|
|Python|  Linting, Debugging (multi-threaded, remote), Intellisense, Jupyter Notebooks, code formatting - [Link](https://marketplace.visualstudio.com/items?itemName=ms-python.python).|
|SSH FS|  File system provider using SSH. - [Link](https://marketplace.visualstudio.com/items?itemName=Kelvin.vscode-sshfs).|
|Visual Studio IntelliCode|   AI-assisted development. - [Link](https://marketplace.visualstudio.com/items?itemName=VisualStudioExptTeam.vscodeintellicode).|
|vscode-icons|    Icons for Visual Studio Code. - [Link](https://marketplace.visualstudio.com/items?itemName=vscode-icons-team.vscode-icons).| 

These extensions help syntax highlighting and aid in errorless code.
Below is an example of how JavaScript code looks in the VS Code. Beautiful, isn't it?
{: style="text-align: justify;"}
![vscode-ibmi]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/vscode-ibmi/vscode_javascript.png){: .align-center} 

## How to install Extensions
![vscode-ibmi]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/vscode-ibmi/vscode_extension.png){: .align-right}Click on the extension symbol as displayed by the image,  type the desired extension in the search bar, when the extension appears, click on the install button, and you are done. More information [here](https://code.visualstudio.com/docs/editor/extension-gallery).   

## How to write code directly in IFS on IBM i?

It becomes convenient if we don't have to upload the code often to the IFS directory on IBM i manually. One extension always comes in handy that is SSH-FS, which makes the IFS folder available to us in the VS Code.
{: style="text-align: justify;"}
### How to use SSH-FS

Install the extension, as explained earlier. You should have a basic knowledge of how SSH works. Aaron Bartell has explained it here on [IT Jungle](https://www.itjungle.com/2014/09/17/fhg091714-story01/), below is a recap.
{: style="text-align: justify;"}
Make sure that the SSH server is running on IBM i, otherwise, issue the below command from the command line on IBM i.
{: style="text-align: justify;"}
```bash
STRTCPSVR SERVER(*SSHD)
```

Below are the steps to connect to the IBM i system from Windows Command Prompt. (I am connecting to [PUB400.com](https://pub400.com/) server with my user-id.)

![vscode-ibmi]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/vscode-ibmi/vscode_ssh1.png){: .align-center} 

When prompted, enter your IBM i password.

![vscode-ibmi]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/vscode-ibmi/vscode_ssh2.png){: .align-center} 
You will sign on to IBM i with a shell $ prompt.

![vscode-ibmi]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/vscode-ibmi/vscode_ssh3.png){: .align-center} 

After you have tested your SSH connection to the IBM i, configure the SSH-FS in VS Code. Use `Ctrl + Shift + P` on Windows or `Cmd + Shift + P` on Mac and type SSHFS in the search bar. 

![vscode-ibmi]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/vscode-ibmi/vscode_sshfs_setting.png){: .align-center} 

SSH-FS needs the below details, along with the name of the connection. 

1. The IP/Domain name of your IBM i system
2. SSH Port (generally port 22, PUB400  has 2222 port configured for SSH)
3. User Id
4. Password
5. Root folder (/home/USERID)

![vscode-ibmi]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/vscode-ibmi/vscode_sshfs_connect.png){: .align-right} After the configuration is done, under SSH FILE SYSTEMS Section of VS Code sidebar, when we right-click on Connect as Workplace folder, the IFS root folder will be accessible.  
We are ready to write our code. 

Below is an example of the code written using SSH FS extension, in the IFS. 

![vscode-ibmi]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/vscode-ibmi/vscode_sshfs_demo.png){: .align-center} 

### Write Free RPGLE code using SSH-FS

It becomes easy to write free format RPGLE using VS Code due to the extensions discussed earlier. To expedite the writing of code, I have created a snippets file, which is another feature of VS Code editor.
{: style="text-align: justify;"}
### What are Snippets?

VS Code has this feature called Snippets, where we can assign a shortcut to a block of code that we write quite often. We can create our snippet for the language we want to use. We have to create a file in JSON format in `File → Preferences →User Snippets`.
{: style="text-align: justify;"}
I have created rpgle.json so that these snippets will be available for a program file with extension .rpgle.  You can download and paste the content of the file from [here](https://github.com/anand-khekale/vscode-ibmi/archive/master.zip).
{: style="text-align: justify;"}

Go to `File→Preferences→User Snippets`

![vscode-ibmi]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/vscode-ibmi/vscode_snippet_path.png){: .align-center} 

You will be prompted with a search bar, enter **rpgle.json.**

![vscode-ibmi]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/vscode-ibmi/vscode_snippet_search.png){: .align-center} 

Paste the content of rpgle.json file as below.

![vscode-ibmi]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/vscode-ibmi/vscode_snippet_json.png){: .align-center} 

Below is the list of shortcuts to start with and these can be expanded as per our needs. → is used to denote the Tab key. 
{: style="text-align: justify;"}

|Shortcut | Description|
|---|:----|
|CTLOPT →|Add Ctl-Opt at the beginning of the code|
|DF →    |Define File with DCL-F|
|DW →    |Define display file with DCL-F|
|DC →    |Define Character Field with DCL-S|
|DV →    |Define VARCHAR field|
|DI →    |Define Integer field|
|DZ →    |Define Zoned field|
|PSDS →  |Defind program information datastructure|
|BEGSR → |Create a subroutine|
|CMT= →  |Add a comment with //=====//|
|CMT- →  |Add a comment with //——-//|
|DOW →   |Write a DoW loop|
|DOU →   |Write a DoU loop|
|IFENDIF →   |Write IF - Endif block|
|IFELSE →    |Write IF - ELSE - ENDIF block|

### Writing code using RPGLE Snippets

With snippets, it becomes fast and joyful to write code. You just have to use the shortcut and press the Tab key or the Enter key, and the code gets generated for you.
{: style="text-align: justify;"}

![vscode-ibmi]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/vscode-ibmi/vscode_snippet_demo.gif){: .align-center} 

### Copying and compiling the IBM i code

When we have completed coding in IFS, it will be better if we automate copying these files to respective source physical files and compile those. It will be a burden if you have to do it manually for each program. Hence, a need for shell script comes in, which I have pasted below. 
{: style="text-align: justify;"}
This shell script will take any file from its current directory with the appropriate extension, let's say .rpgle, and copy and compile it. For this to run, we have to use our SSH session or QP2TERM (CALL QP2TERM) session on the IBM i.  
{: style="text-align: justify;"}
From the $ prompt, make the file executable for the first time as below.
```bash
$ chmod +x buildpgm.sh
```
And run it with the below command.
```bash
$ ./buildpgm.sh 
```

Well, the native IBM i programs can be compiled directly from IFS, but if you have DSPFs, those need source physical file. So for now, I keep everything in respective source physical files. 
{: style="text-align: justify;"}
Below is such a script I have created, which does the job as explained. 

```bash
#!/QOpenSys/usr/bin/sh
# ------------------------------------------------------------------------- #
# Program       : buildpgm.sh
# Author        : www.anandk.dev
# Date Written  : 27th October 2020
# Inspired from : https://github.com/barrettotte/RPGLE-Twilio/blob/master/build.sh
# ------------------------------------------------------------------------- #
# This program reads current folder from IFS and takes below steps.
# 1. It assumes these sources i.e. dspf, rpgle, sqlrpgle, clle.  
# 2. It copies .rpgle, .sqlrpgle, .clle and .dspf to respective _SRC. 
# 3. It compiles the DSPF first, then RPGLE, SQLRPGLE and finally CLLE. 
# After coping this program to IFS folder where your sources are, use below command
# to make this file executable. 
# chmod +x buildpgm.sh
# Set the CUR_LIB to your library.
# Check if the SRCPF names are correct for you. 
# Set the application name which will be applied to all the memebers as member text.
# ------------------------------------------------------------------------- #

# Set the IFS directory, if not given, set current
IFS_DIR="${1:-$(pwd)}"

# Set the current library where the programs will be compiled. 
CUR_LIB="ANANDK1"

# Set source physical files 
DDS_SRC="QDDSSRC"
RPGLE_SRC="QRPGLESRC"
CL_SRC="QCLSRC"

# Set application name
APPLICATION="Weather Application"

# Execute the command by setting the library first
exec_cmd(){
    echo $1
    output=$(qsh -c "liblist -a $CUR_LIB ; system \"$1\"")
    if [ -n "$2" ]; then
    echo -e "$1\n\n$output" > "$IFS_DIR/$2.log"
    fi
}

# Copy DSPF to source PF and compile
crt_dspf(){
  echo -e '\n... executing commands... '
  filename=$(basename "$1")
  member="${filename%.*}"
  exec_cmd "CHGATR OBJ('$1') ATR(*CCSID) VALUE(819)"
  exec_cmd "CPYFRMSTMF FROMSTMF('$1') TOMBR('/QSYS.lib/$CUR_LIB.lib/$DDS_SRC.file/$member.mbr') MBROPT(*REPLACE)"
  exec_cmd "CHGPFM FILE($CUR_LIB/$DDS_SRC) MBR($member) SRCTYPE(DSPF) TEXT('$APPLICATION')"
  exec_cmd "CRTDSPF FILE($CUR_LIB/$member) SRCFILE($CUR_LIB/$DDS_SRC)" $member 
}

# Copy RPGLE to SRCPF and compile
crt_rpgle(){
  echo -e '\n... executing commands... '
  filename=$(basename "$1")
  member="${filename%.*}"
  exec_cmd "CHGATR OBJ('$1') ATR(*CCSID) VALUE(819)"
  exec_cmd "CPYFRMSTMF FROMSTMF('$1') TOMBR('/QSYS.lib/$CUR_LIB.lib/$RPGLE_SRC.file/$member.mbr') MBROPT(*REPLACE)"
  exec_cmd "CHGPFM FILE($CUR_LIB/$RPGLE_SRC) MBR($member) SRCTYPE(RPGLE) TEXT('$APPLICATION')"
  exec_cmd "CRTBNDRPG PGM($CUR_LIB/$member) DFTACTGRP(*NO) DBGVIEW(*SOURCE)" $member   
}

# Copy SQLRPGLE to SRCPF and compile
crt_sqlrpgle(){
  echo -e '\n... executing commands... '
  filename=$(basename "$1")
  member="${filename%.*}"
  exec_cmd "CHGATR OBJ('$1') ATR(*CCSID) VALUE(819)"
  exec_cmd "CPYFRMSTMF FROMSTMF('$1') TOMBR('/QSYS.lib/$CUR_LIB.lib/$RPGLE_SRC.file/$member.mbr') MBROPT(*REPLACE)"
  exec_cmd "CHGPFM FILE($CUR_LIB/$RPGLE_SRC) MBR($member) SRCTYPE(SQLRPGLE) TEXT('$APPLICATION')"
  exec_cmd "CRTSQLRPGI OBJ($CUR_LIB/$member) SRCFILE($CUR_LIB/$RPGLE_SRC) COMMIT(*NONE) DBGVIEW(*SOURCE)" $member   
}
                                                                   
# Copy CLLE to SRCPF and compile
crt_clle(){
echo -e '\n... executing commands... '
  filename=$(basename "$1")
  member="${filename%.*}"
  exec_cmd "CHGATR OBJ('$1') ATR(*CCSID) VALUE(819)"
  exec_cmd "CPYFRMSTMF FROMSTMF('$1') TOMBR('/QSYS.lib/$CUR_LIB.lib/$CL_SRC.file/$member.mbr') MBROPT(*REPLACE)"
  exec_cmd "CHGPFM FILE($CUR_LIB/$CL_SRC) MBR($member) SRCTYPE(CLLE) TEXT('$APPLICATION')"
  exec_cmd "CRTBNDCL PGM($CUR_LIB/$member) SRCFILE($CUR_LIB/$CL_SRC) DFTACTGRP(*NO) ACTGRP(*CALLER) DBGVIEW(*SOURCE)" $member   
}

search_dspf(){
    for FILE in "$IFS_DIR"/*
        do
            ext="${FILE##*.}"
            if [[ $ext == 'dspf' ]]; then
                echo -e "\n=========== Building DSPF - $FILE ============="
                crt_dspf $FILE 
            fi
        done
}

search_rpgle(){
    for FILE in "$IFS_DIR"/*
        do
            ext="${FILE##*.}"
            if [[ $ext == 'rpgle' ]]; then
                echo -e "\n=========== Building RPGLE - $FILE ============="
                crt_rpgle $FILE 
            fi
        done
}

search_sqlrpgle(){
    for FILE in "$IFS_DIR"/*
        do
            ext="${FILE##*.}"
            if [[ $ext == 'sqlrpgle' ]]; then
                echo -e "\n=========== Building SQLRPGLE - $FILE ============="
                crt_sqlrpgle $FILE 
            fi
        done
}

search_clle(){
    for FILE in "$IFS_DIR"/*
        do
            ext="${FILE##*.}"
            if [[ $ext == 'clle' ]]; then
                echo -e "\n=========== Building CLLE - $FILE ============="
                crt_clle $FILE 
            fi
        done
}

echo -e "|=== Starting to build the programs for $APPLICATION ===|"
search_dspf
search_rpgle
search_sqlrpgle
search_clle
echo -e '|=================================================================================|'
echo -e '| Program build completed, please check if you encountered any errors.            |'
echo -e '| Check the log files created in the below folder.                                |'
echo -e "| $IFS_DIR                                                            |"
echo -e '|=================================================================================|'
```

## Synchronizing your set up

It is necessary that, when you have completed all your customization for VS Code, you should sync your setup so that, if you change your laptop/desktop, those are retained and you don't have to do it all over again. 
{: style="text-align: justify;"}

When you go to `File → Preferences →Setting Sync` you can configure where you want to sync it. I have currently set it to sync with my GitHub account. 
{: style="text-align: justify;"}
![vscode-ibmi]({{ site.url }}{{ site.baseurl }}/assets/images/forposts/vscode-ibmi/vscode_setting_sync.png){: .align-center} 

## Wrapping up...

I find the VS Code editor highly customizable and efficient to use while I work on IBM i. It is also available on all operating systems. 
{: style="text-align: justify;"}

I hope this article has given you some ideas on how to use the editor.
{: style="text-align: justify;"}
As always, request you to get in touch with me on [LinkedIn](https://www.linkedin.com/in/anandkdev)/[Twitter](https://twitter.com/anandkdev) or [email](mailto:anand.khekale@gmail.com) with your valuable feedback. Thanks for reading!
{: style="text-align: justify;"}