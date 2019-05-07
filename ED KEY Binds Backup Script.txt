@goto script
This script was written for the elite dangerous.  What it does is backup your current bindings to your desktop.  You will then be able to restore the backups to the game folder.  When backups are restored, it will backup the original folder then copy the backed up files into the game folder.
:script
@echo off&setlocal enabledelayedexpansion
cd %desktop%
set savepath="%userprofile%\desktop\BindingsBackup"
set bindings="%localappdata%\Frontier Developments\Elite Dangerous\Options\Bindings"
title Backup and Restore Elite Dangerous Bindings

:top
cls
echo Bindings location: 
echo %bindings%
echo.
echo Backup location:
echo %savepath%
echo.
echo.
echo ++++++++++++++++++++++++++++++++++++++++++++++++++
echo + This script will backup your bindings folder   +
echo + to your desktop.                               +
echo +                                                +
echo + 1. Backup bindings                             +
echo + 2. Restore bindings                            +
echo + 3. Open bindings folder                        +
echo + 4. Open backup folder                          +
echo +                                                +
echo + 5. Exit                                        +
echo +                                                +
echo ++++++++++++++++++++++++++++++++++++++++++++++++++
echo.
choice /C 12345 /N /M "Please enter a selection: "
if %errorlevel% == 1 goto backup
if %errorlevel% == 2 goto restore
if %errorlevel% == 3 start "" %bindings%&goto top
if %errorlevel% == 4 start "" %savepath%&goto top
REM if the user doesn't select 1-4 automatically exit.
exit
:backup
cls
REM check if the savepath exists
call :foldercheck %savepath%

REM if the folder exists then warn the user they are about to overwrite
if %errorlevel% == 1 (
   choice /C yn /N /M "Bindings have already been backed up.  Would you like to overwrite? [y,n]: "
   if !errorlevel! == 1 rd /s/q %savepath%\&md %savepath%
   if !errorlevel! == 2 goto top
) else (
REM if the folder doesn't exist then make it
   md %savepath% >nul
)     
REM copy the bindings to the new backup folder
xcopy /I /Q /S %bindings% %savepath%
echo.
echo Backup complete.
echo Backup is located on your desktop in the BindingsBackup folder
timeout 5 >nul
goto top

:restore
cls
REM check if the backup exists
call :foldercheck %savepath%
if %errorlevel% == 1 (
REM warn the user they are about to overwrite their current bindings with a backup

   echo If you continue your current bindings will be replaced by the bindings located in
   echo %userprofile%\desktop\BindingBackup. 
   echo.
   choice /c yn /n /m "Continue? [y,n]: "
   if !errorlevel! == 2 goto top
   if !errorlevel! == 1 (
      cd %bindings%
      cd ../
REM backup the current folder to Bindings.bak just in case
      xcopy /I /S Bindings Bindings.bak
REM delete the Bindings folder and everything in it
      rd /s/q Bindings
REM Create a new bindings folder and move the backup into it
      md Bindings
      xcopy /I /S %savepath% %bindings%
   )
) else (
      echo Error!  No backup located!
      timeout 3 >nul
      goto top
   )
goto top


REM This function comes from M. Jamal on https://stackoverflow.com/questions/138981/how-to-test-if-a-file-is-a-directory-in-a-batch-script
REM This function will return an errorlevel depending on whether the supplied folderpath is valid.
REM Errorlevel 0 means file/folder doesn't exist
REM Errorlevel 1 means exists, and it is a folder
REM Errorlevel 3 means exists, and it is a file
:foldercheck <path>
@pushd %~dp1
@if not exist "%~nx1" (
        popd
        exit /b 0
) else (
        if exist "%~nx1\*" (
                popd
                exit /b 1
        ) else (
                popd
                exit /b 3
        )
)
