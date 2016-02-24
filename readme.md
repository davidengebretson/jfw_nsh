(This file last updated 2/23/16.)
Jaws script installer
Written by Dang Manh Cuong <dangmanhcuong@gmail.com> and Gary Campbell <campg2003@gmail.com>
This installer requires the NSIS program from http://nsis.sourceforge.net

# Features:
- Installs into all English versions of Jaws. This will be true as long as Freedom Scientific does not change the place to put scripts.
- Supports JAWS 17.0 shared scripts folder structure (as I currently understand it).
- Installs for the highest priveleg available to the user.  If it can install for all users, the /currentuser command line switch can be supplied to force a current user install.
- For all user installs the user can choose whether to install scripts as shared or for the current user.
- Gets the correct install path of Jaws from the registry.
- Checks for a Jaws installation before starting setup. If Jaws is not installed, displays a warning message and quits.
- contains macros for extracting, compiling, deleting, and modifying scripts, so user can create a package containing multiple scripts quickly and easily.
- Macro to copy script from shared to current user.

# Overview
This header file provides the following pages:
- Welcome
- License (optional)
- Install type
- Select JAWS versions/languages
- Destination folder (for Full and Custom installs)
- Confirm Installation Settings
- Finish

Three install types are supported:
- Just Scripts: installs the scripts but does not install uninstall information or make a folder in %ProgramFiles%.
- Full: installs scripts in the script folder for the selected versions/languages, creates a folder in %programfiles% (%localappdata% for current user install) wich contains an uninstaller and optional additional files such as README, etc.
- Custom: like Full but allows installation of the installer source.

For full or custom installations for all users the uninstaller and README files are installed in %programfiles%.  For current user installations they are installed in %localappdata%.

If the user's priveleges allow for installing for all users, an all user install is performed.  Otherwise a current user install is performed.  If privileges allow for all user installation, current user installation can be forced by adding the /currentuser command line switch.

The Versions/Languages page displays the JAWS versions installed on the system and the languages they support.  This is presented in a checkable list view with entries like "17.0/enu".  Select the desired versions/languages by pressing SPACE, which checks the highlighted entry.  For all users installations this page also optionally allows you to choose whether to install the script for the current user or as a shared script.

The Finish page offers to open a README file if desired.


# Usage
To use JFW.nsh, you set some defines, define a couple of macros that install the files, and insert the JAWSScriptInstaller macro.  See the included [sample](sample/vwapp.nsi) for an example.

## Dependencies
This header uses header files currently shipped with NSIS.  Support for the [uninstlog](https://github.com/campg2j003/uninstlog) header file is provided if it has been included.

Note: If uninstlog.nsh is not included by the script using this header file, no uninstaller section will be included by this header file, and the user is responsible for writing one.  This will not be indicated by the messages in the Confirm Installation Settings page.  This configuration has not been tested!

## About the JAWS script compiler and multiple languages
The JAWS script compiler (scompile.exe) always compiles the script for the language of the currently-running JAWS.  This means that, even though it generates a JSB file in the folder containing a JSS for another language, the script actually compiled is that of the running language.  This means that, although the proper script files for each language are installed, the user will have to manually compile the script while running JAWS in that language.

By default, the script files are expected to be contained in a folder called script in the folder containing this header.  

## Defines
The following can be defined in your installer before including the header.  Most have defaults if not defined.
If you want to enable support for choosing to install in either the current user or all users, define JAWSALLOWALLUSERS before including this file.  If not defined, the default is to install into the current user.  If you execute SetShellVarContext you should also set the variable JAWSSHELLCONTEXT to match.  The macro JAWSSETSHELLCONTEXT has been provided to allow setting the shell var context based on the value of a parameter.

Note: Due to addition of support for all/current user installations it is recommended that you define JAWSALLOWALLUSERS.  This define may be removed in the future.  The installer has not been tested with this undefined.

### User settings:
```
!Define ScriptName "Jaws Script for <program name>" ;name displayed to user and used for folder in %programfiles%
!define ScriptApp "<program name>" ; the base name of the app the scripts are for
!define JAWSMINVERSION "" ; min version of JAWS for which this script can be installed, e.g. "9.0"
!define JAWSMAXVERSION "" ; max version of JAWS for which this script can be installed, e.g. "17.0"
(If "" no restriction is imposed.)
!define JAWSALLOWALLUSERS ; comment this line if you don't want to allow installation for all users.  (Not recommended.)
;Uncomment and change if the scripts are in another location.
;!define JAWSSrcDir "script\" ;Folder relative to current folder containing JAWS scripts, empty or ends with backslash.

;Will be omitted if not defined.
!define LegalCopyright "Copyright Message"
;The file name of the license file in ${JAWSSrcDir}.  If not defined, no license page will be included.
!define JAWSLicenseFile "copying.txt"

;Optional installer finish page features
;Assigns default if not defined.
;!define MUI_FINISHPAGE_SHOWREADME "$instdir\${ScriptApp}_readme.txt"
;!define JAWSNoReadme ;uncomment if you don't have a README.
!define MUI_FINISHPAGE_LINK "$(GoToAuthorsPage)"
!define MUI_FINISHPAGE_LINK_LOCATION "http://site_url"

!define SetOverwriteDefault "on"  ;Set this to the value of SetOverwrite.
```

(ToDo: Reorder these in the installer and use its value with the define.)

### User-defined macros
The files for your script are specified by defining several macros.

```
; Define The following in the user's file before including the JFW.nsh header.
;We include the langstring header after the MUI_LANGUAGE macro.
!include "uninstlog.nsh" ; optional but strongly recommended!
```

Define the following macro to install the files for one version of JAWS.

```
!macro JAWSInstallScriptItems
;Contains the instructions to install the scripts in each version of JAWS.  If not defined, the installer will use a default version that tries to install every type of JAWS script file for an application I know of.
;Assumes uninstlog is open when called.
;Version in $0, lang in $1.
${JawsScriptFile} "${JAWSSrcDir}" "audacity.jdf"
${JawsScriptFile} "${JAWSSrcDir}" "audacity.jss"
${JawsScriptFile} "${JAWSSrcDir}" "audacity.qs"
...
!macroend ;JAWSInstallScriptItems

;The macro ${JawsScriptFile} SrcDir file
;is provided to install a script file into the proper folder based on JAWS version, current/shared, and script language.  It should be used for installing all script files.

;Items to be placed in the installation folder in a full install.  Note that if this macro is not defined, makensis will generate a warning about undefined symbols which may be ignored.
!macro JAWSInstallFullItems
...
!macroend ;JAWSInstallFullItems
```

Define the following macro to allow the installer source to be installed.  It must include the macro JAWSJFWNSHInstallerSrc.

```
!macro JAWSInstallerSrc
;Install your installer source files here.
!InsertMacro JAWSJFWNSHInstallerSrc
!MacroEnd ;JAWSInstallerSrc
```

If this macro is defined and the user selects the Installer Source component, the installer will create a folder in the installation folder called "Installer Source" and install the installer source files in it.  If this macro is not defined, a default macro is used that only installs the source for JFW.nsh.

The installer uses the Modern UI II package, so it includes mui2.nsh.  You should not include mui2.nsh before including JFW.nsh.  After the above defines you include this header and insert the JAWSScriptInstaller macro which produces the installer code:

```
!include "jfw.nsh"

!insertmacro JAWSScriptInstaller
;Strange though it seems, the language file includes must follow the invocation of JAWSScriptInstaller.
  !include "uninstlog_enu.nsh" ;required if uninstlog.nsh is used
  !include "uninstlog_esn.nsh" ;required if uninstlog.nsh is used
!include "installer_lang_enu.nsh"
!include "installer_lang_esn.nsh"
```

# Available macros
```
!Macro CompileSingle JAWSVer Source
;Assumes $OUTDIR points to folder where source file is and compiled file will be placed.
;JAWSVer - JAWS version/lang or version, i.e. "10.0/enu" or "10.0"
;Source - name of script to compile without .jss extension
;return: writes error message on failure, returns exit code of scompile (0 if successful) in $1.
;Recommend for scripts wich have only one source (*.JSS) file, or don't make any modification to any original files
;This macro saves time because it doesn't store and delete any temporary files.

;These are used to record files for the uninstaller.  They do nothing if uninstlog is not included.
!macro JAWSLOG_OPENINSTALL
!macro JAWSLOG_CLOSEINSTALL

!macro JAWSLOG_UNINSTALL

!macro JawsScriptSetPath ext
;Set $OUTDIR to proper location for script file type ext.
;Usage: ${JawsScriptSetPath} ext
;ext -- script file extension.
;Assumes $0 = version, $1 = lang, shell var context is set to $SHELLSCRIPTCONTEXT.

!macro JawsScriptFile SrcDir file
;Install a JAWS script file into the proper location.  Takes into account whether installing into current user's scripts or shared scripts, JAWS version, and language.
;Usage: ${JawsScriptFile} SrcDir file
  ;SrcDir -- folder containing source file, used in FileDated macro.
;File -- name of file in SrcDir, used in FileDated macro and elsewhere.
  ;Assumes uninstlog macros available (either the real thing or dummies defined here) -- for SetOutPath and FileDated.
  ;Assumes $0 = version, $1 = lang, shell var context is set to $SHELLSCRIPTCONTEXT.
;This macro installs the file with the ${FileDated} uninstlog macro.  If this isn't what you need, use ${JawsScriptSetPath} to set $OUTDIR followed by the appropriate installation commands.

!macro JAWSMultiuserInstallModePage
;Includes a page to allow the user to choose whether to install for all users or the current user.
;This macro has been tested but is not currently used.
```

The following macros are not used by the Audacity JAWS script installer.  I have not used them and have not tested them.

```
!Macro AdvanceCompileSingle JAWSVer Path Source
;Assumes $OUTDIR points to folder where source file is and compiled file will be placed.
;JAWSVer - JAWS version/lang or version, i.e. "10.0/enu" or "10.0"
;Path - desired context, either "current" or "all".
;Source - name of script to compile without .jss extension
;return: writes error message on failure, returns exit code of scompile (0 if successful)-- actually returns 1 on failure.

!Macro AddHotkey JKM Key Script
;Add hotkeys to *.jkm file
;Usually used by advanced user
;Assumes JKM file is in $OUTDIR.
;JKM - name of JKM file without the .jkm extension.
;key - string containing the key sequence, like "CTRL+JAWSKey+a".
;Script - name of script to bind to key.
;Entries will be added to the "Common Keys" section.

!macro ModifyScript JAWSVer File Code
;Use to add some code to the existing script
;Like adding: use "skypewatch.jsb"" to default.jss

!macro AdvanceModifyScript JAWSVer Path File Code
;Use to add some code to the existing script
;Like adding: use "skypewatch.jsb"" to default.jss


```

# Copyright
    Copyright (C) 2012-2016  Gary Campbell and Dang Manh Cuong.  All rights reserved.

    This program is free software; you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation; either version 2 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program; if not, write to the Free Software
    Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301  USA
    
    See the file copying.txt for details.

