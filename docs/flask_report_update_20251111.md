Here's the updated batch script that processes all `.txt` files in a directory to their `.html` equivalents:

## Main Batch Processing Script (`process_reports.bat`)

```batch
@echo off
setlocal enabledelayedexpansion

echo ========================================
echo    AI REPORT BATCH PROCESSOR
echo ========================================
echo.

:: Configuration
set "SCRIPT_DIR=%~dp0"
set "INPUT_DIR=%SCRIPT_DIR%reports"
set "OUTPUT_DIR=%SCRIPT_DIR%html_reports"
set "PYTHON_SCRIPT=report_processor.py"
set "LOG_FILE=%SCRIPT_DIR%processing.log"

:: Create directories if they don't exist
if not exist "%INPUT_DIR%" (
    echo Creating input directory: %INPUT_DIR%
    mkdir "%INPUT_DIR%"
)

if not exist "%OUTPUT_DIR%" (
    echo Creating output directory: %OUTPUT_DIR%
    mkdir "%OUTPUT_DIR%"
)

:: Check if Python is installed
python --version >nul 2>&1
if errorlevel 1 (
    echo ERROR: Python is not installed or not in PATH
    echo Please install Python from https://python.org
    pause
    exit /b 1
)

:: Check if Python script exists
if not exist "%SCRIPT_DIR%%PYTHON_SCRIPT%" (
    echo ERROR: Python script not found: %PYTHON_SCRIPT%
    echo Please ensure the script is in the same directory
    pause
    exit /b 1
)

:: Initialize counters
set /a processed=0
set /a errors=0
set /a total=0

echo Starting batch processing...
echo Input: %INPUT_DIR%
echo Output: %OUTPUT_DIR%
echo.

:: Count total .txt files
for /f %%a in ('dir /b "%INPUT_DIR%\*.txt" 2^>nul ^| find /c /v ""') do set /a total=%%a

if !total! equ 0 (
    echo No .txt files found in %INPUT_DIR%
    echo Please place your report files in the 'reports' folder
    pause
    exit /b 1
)

echo Found !total! report files to process
echo.

:: Process each .txt file to .html
for %%f in ("%INPUT_DIR%\*.txt") do (
    set "filename=%%~nf"
    set "input_file=%%f"
    set "output_file=%OUTPUT_DIR%\!filename!.html"
    
    echo Processing: %%~nxf → !filename!.html
    echo ----------------------------------------
    
    :: Call Python script to process the file
    python "%SCRIPT_DIR%%PYTHON_SCRIPT%" "!input_file!" "!output_file!"
    
    if !errorlevel! equ 0 (
        echo ✓ Success: !filename!.html
        set /a processed+=1
    ) else (
        echo ✗ Failed: !filename!.txt
        set /a errors+=1
        
        :: Log error
        echo [ERROR] !date! !time! - Failed to process !filename!.txt >> "%LOG_FILE%"
    )
    echo.
)

:: Display summary
echo ========================================
echo BATCH PROCESSING COMPLETE
echo ========================================
echo Total .txt files found: !total!
echo Successfully converted: !processed!
echo Errors: !errors!
echo.

if !errors! gtr 0 (
    echo Check %LOG_FILE% for error details
) else (
    echo All files converted successfully!
)

echo.
echo Input:  %INPUT_DIR%\*.txt
echo Output: %OUTPUT_DIR%\*.html
echo.

:: Create index.html with all converted files
echo Creating index.html with all reports...
call :create_index

:: Open output directory in Explorer
set /p "open=Open output directory in Explorer? (Y/N): "
if /i "!open!"=="Y" (
    explorer "%OUTPUT_DIR%"
)

pause
exit /b 0

:create_index
set "INDEX_FILE=%OUTPUT_DIR%\index.html"

echo ^<!DOCTYPE html^> > "%INDEX_FILE%"
echo ^<html^>^<head^> >> "%INDEX_FILE%"
echo ^<title^>AI Target Detection Reports Index^</title^> >> "%INDEX_FILE%"
echo ^<style^> >> "%INDEX_FILE%"
echo body { font-family: Arial, sans-serif; margin: 40px; background: #f5f5f5; } >> "%INDEX_FILE%"
echo .container { max-width: 800px; margin: 0 auto; background: white; padding: 30px; border-radius: 10px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); } >> "%INDEX_FILE%"
echo h1 { color: #2c3e50; border-bottom: 2px solid #3498db; padding-bottom: 10px; } >> "%INDEX_FILE%"
echo .report-list { list-style: none; padding: 0; } >> "%INDEX_FILE%"
echo .report-item { padding: 12px; border-bottom: 1px solid #eee; } >> "%INDEX_FILE%"
echo .report-item:hover { background: #f8f9fa; } >> "%INDEX_FILE%"
echo .report-link { text-decoration: none; color: #2980b9; font-weight: 500; } >> "%INDEX_FILE%"
echo .report-link:hover { color: #3498db; } >> "%INDEX_FILE%"
echo .count { float: right; color: #7f8c8d; } >> "%INDEX_FILE%"
echo .summary { background: #e8f5e8; padding: 15px; border-radius: 5px; margin-bottom: 20px; } >> "%INDEX_FILE%"
echo ^</style^> >> "%INDEX_FILE%"
echo ^</head^> >> "%INDEX_FILE%"
echo ^<body^> >> "%INDEX_FILE%"
echo ^<div class="container"^> >> "%INDEX_FILE%"
echo ^<h1^>📊 AI Target Detection Reports^</h1^> >> "%INDEX_FILE%"
echo ^<div class="summary"^> >> "%INDEX_FILE%"
echo Generated on !date! at !time! >> "%INDEX_FILE%"
echo ^<br^>Total reports: !total! • Successful: !processed! • Errors: !errors! >> "%INDEX_FILE%"
echo ^</div^> >> "%INDEX_FILE%"
echo ^<ul class="report-list"^> >> "%INDEX_FILE%"

set /a count=0
for %%f in ("%OUTPUT_DIR%\*.html") do (
    if /i not "%%~nxf"=="index.html" (
        set /a count+=1
        echo ^<li class="report-item"^>^<a class="report-link" href="%%~nxf"^>^<span class="count"^>!count!^</span^>%%~nxf^</a^>^</li^> >> "%INDEX_FILE%"
    )
)

echo ^</ul^> >> "%INDEX_FILE%"
echo ^</div^> >> "%INDEX_FILE%"
echo ^</body^>^</html^> >> "%INDEX_FILE%"

echo ✓ Index created: index.html
goto :eof
```

## Advanced Batch Processor with Monitoring (`process_reports_advanced.bat`)

```batch
@echo off
setlocal enabledelayedexpansion

title AI Report Batch Processor - Advanced Mode

:: Configuration
set "CONFIG_FILE=%~dp0batch_config.ini"
set "SCRIPT_DIR=%~dp0"
set "DEFAULT_INPUT=%SCRIPT_DIR%reports"
set "DEFAULT_OUTPUT=%SCRIPT_DIR%html_reports"
set "PYTHON_SCRIPT=report_processor.py"
set "LOG_FILE=%SCRIPT_DIR%processing.log"
set "ARCHIVE_DIR=%SCRIPT_DIR%processed_reports"

:: Load configuration
if exist "%CONFIG_FILE%" (
    for /f "tokens=1,2 delims==" %%a in (%CONFIG_FILE%) do (
        if "%%a"=="INPUT_DIR" set "INPUT_DIR=%%b"
        if "%%a"=="OUTPUT_DIR" set "OUTPUT_DIR=%%b"
        if "%%a"=="ARCHIVE_PROCESSED" set "ARCHIVE_PROCESSED=%%b"
        if "%%a"=="CREATE_INDEX" set "CREATE_INDEX=%%b"
        if "%%a"=="AUTO_MONITOR" set "AUTO_MONITOR=%%b"
    )
)

:: Set defaults if not configured
if not defined INPUT_DIR set "INPUT_DIR=%DEFAULT_INPUT%"
if not defined OUTPUT_DIR set "OUTPUT_DIR=%DEFAULT_OUTPUT%"
if not defined ARCHIVE_PROCESSED set "ARCHIVE_PROCESSED=false"
if not defined CREATE_INDEX set "CREATE_INDEX=true"
if not defined AUTO_MONITOR set "AUTO_MONITOR=false"

:menu
cls
echo ========================================
echo    AI REPORT BATCH PROCESSOR - ADVANCED
echo ========================================
echo.
echo 1. Convert All .txt to .html
echo 2. Convert Single .txt File
echo 3. Monitor Directory (Auto-convert new .txt)
echo 4. Settings and Configuration
echo 5. View Processing Log
echo 6. Create Reports Index
echo 7. Exit
echo.
set /p "choice=Select option (1-7): "

if "%choice%"=="1" goto process_all
if "%choice%"=="2" goto process_single
if "%choice%"=="3" goto monitor
if "%choice%"=="4" goto settings
if "%choice%"=="5" goto view_log
if "%choice%"=="6" goto create_index
if "%choice%"=="7" exit /b 0

echo Invalid choice. Press any key to continue...
pause >nul
goto menu

:process_all
cls
echo Converting all .txt files in %INPUT_DIR% to .html
echo.

:: Create directories
if not exist "%INPUT_DIR%" mkdir "%INPUT_DIR%"
if not exist "%OUTPUT_DIR%" mkdir "%OUTPUT_DIR%"
if not exist "%ARCHIVE_DIR%" mkdir "%ARCHIVE_DIR%"

:: Check dependencies
python --version >nul 2>&1
if errorlevel 1 (
    echo ERROR: Python not found
    pause
    goto menu
)

if not exist "%SCRIPT_DIR%%PYTHON_SCRIPT%" (
    echo ERROR: Python script not found
    pause
    goto menu
)

set /a processed=0
set /a errors=0
set /a total=0

:: Count .txt files
for /f %%a in ('dir /b "%INPUT_DIR%\*.txt" 2^>nul ^| find /c /v ""') do set /a total=%%a

if !total! equ 0 (
    echo No .txt files found in %INPUT_DIR%
    pause
    goto menu
)

echo Found !total! .txt files. Starting conversion...
echo.

for %%f in ("%INPUT_DIR%\*.txt") do (
    set "filename=%%~nf"
    set "input_file=%%f"
    set "output_file=%OUTPUT_DIR%\!filename!.html"
    
    echo [!time!] Converting: %%~nxf → !filename!.html
    
    python "%SCRIPT_DIR%%PYTHON_SCRIPT%" "!input_file!" "!output_file!"
    
    if !errorlevel! equ 0 (
        echo [!time!] ✓ Success: !filename!.html
        set /a processed+=1
        
        :: Archive original .txt if enabled
        if /i "!ARCHIVE_PROCESSED!"=="true" (
            move "!input_file!" "%ARCHIVE_DIR%\%~nxf" >nul 2>&1
            echo [!time!] Archived original: %%~nxf
        )
    ) else (
        echo [!time!] ✗ Failed: !filename!.txt
        set /a errors+=1
        echo [!date! !time!] FAILED: !filename!.txt >> "%LOG_FILE%"
    )
)

echo.
echo = CONVERSION SUMMARY =====================
echo Total .txt files: !total!
echo Successfully converted: !processed!
echo Errors: !errors!
echo Output directory: %OUTPUT_DIR%
echo.

:: Create index if enabled
if /i "!CREATE_INDEX!"=="true" (
    call :create_index
)

echo Press any key to continue...
pause >nul
goto menu

:process_single
cls
echo Available .txt files:
echo.
dir /b "%INPUT_DIR%\*.txt" 2>nul
echo.
set /p "filename=Enter filename (without .txt): "

if not exist "%INPUT_DIR%\%filename%.txt" (
    echo File not found: %filename%.txt
    pause
    goto menu
)

echo Converting %filename%.txt to %filename%.html...
python "%SCRIPT_DIR%%PYTHON_SCRIPT%" "%INPUT_DIR%\%filename%.txt" "%OUTPUT_DIR%\%filename%.html"

if !errorlevel! equ 0 (
    echo ✓ Successfully converted: %filename%.html
    if /i "!ARCHIVE_PROCESSED!"=="true" (
        move "%INPUT_DIR%\%filename%.txt" "%ARCHIVE_DIR%\%filename%.txt" >nul 2>&1
        echo Original file archived
    )
) else (
    echo ✗ Failed to convert %filename%.txt
)

pause
goto menu

:monitor
cls
echo Directory Monitoring Mode - Auto-convert new .txt files
echo.
echo Watching: %INPUT_DIR%
echo Output: %OUTPUT_DIR%
echo Press Ctrl+C to stop monitoring...
echo.

:monitor_loop
for %%f in ("%INPUT_DIR%\*.txt") do (
    set "filename=%%~nf"
    if not exist "%OUTPUT_DIR%\!filename!.html" (
        echo [!time!] New .txt detected: %%~nxf
        python "%SCRIPT_DIR%%PYTHON_SCRIPT%" "%%f" "%OUTPUT_DIR%\!filename!.html"
        if !errorlevel! equ 0 (
            echo [!time!] ✓ Auto-converted: !filename!.html
            if /i "!ARCHIVE_PROCESSED!"=="true" (
                move "%%f" "%ARCHIVE_DIR%\%%~nxf" >nul 2>&1
                echo [!time!] Archived original
            )
        ) else (
            echo [!time!] ✗ Auto-convert failed: !filename!.txt
        )
        echo.
    )
)

:: Wait 5 seconds before checking again
timeout /t 5 /nobreak >nul
goto monitor_loop

:settings
cls
echo = CONFIGURATION ===========================
echo.
echo Current settings:
echo 1. Input Directory: %INPUT_DIR%
echo 2. Output Directory: %OUTPUT_DIR%
echo 3. Archive Original .txt Files: %ARCHIVE_PROCESSED%
echo 4. Create Index Page: %CREATE_INDEX%
echo 5. Auto-monitor Directory: %AUTO_MONITOR%
echo.
echo 6. Back to Menu
echo.
set /p "setting=Select setting to change (1-6): "

if "%setting%"=="1" (
    set /p "NEW_INPUT=Enter new input directory: "
    set "INPUT_DIR=!NEW_INPUT!"
)
if "%setting%"=="2" (
    set /p "NEW_OUTPUT=Enter new output directory: "
    set "OUTPUT_DIR=!NEW_OUTPUT!"
)
if "%setting%"=="3" (
    if /i "!ARCHIVE_PROCESSED!"=="true" (
        set "ARCHIVE_PROCESSED=false"
    ) else (
        set "ARCHIVE_PROCESSED=true"
    )
)
if "%setting%"=="4" (
    if /i "!CREATE_INDEX!"=="true" (
        set "CREATE_INDEX=false"
    ) else (
        set "CREATE_INDEX=true"
    )
)
if "%setting%"=="5" (
    if /i "!AUTO_MONITOR!"=="true" (
        set "AUTO_MONITOR=false"
    ) else (
        set "AUTO_MONITOR=true"
    )
)

:: Save configuration
echo INPUT_DIR=%INPUT_DIR% > "%CONFIG_FILE%"
echo OUTPUT_DIR=%OUTPUT_DIR% >> "%CONFIG_FILE%"
echo ARCHIVE_PROCESSED=%ARCHIVE_PROCESSED% >> "%CONFIG_FILE%"
echo CREATE_INDEX=%CREATE_INDEX% >> "%CONFIG_FILE%"
echo AUTO_MONITOR=%AUTO_MONITOR% >> "%CONFIG_FILE%"

goto menu

:view_log
cls
if exist "%LOG_FILE%" (
    echo Processing Log:
    echo ===============
    type "%LOG_FILE%"
) else (
    echo No log file found.
)
echo.
pause
goto menu

:create_index
echo Creating index page with all .html reports...
set "INDEX_FILE=%OUTPUT_DIR%\index.html"

echo ^<!DOCTYPE html^> > "%INDEX_FILE%"
echo ^<html^>^<head^>^<title^>AI Reports Index^</title^> >> "%INDEX_FILE%"
echo ^<style^> >> "%INDEX_FILE%"
echo body { font-family: Arial, sans-serif; margin: 40px; background: #f5f5f5; } >> "%INDEX_FILE%"
echo .container { max-width: 800px; margin: 0 auto; background: white; padding: 30px; border-radius: 10px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); } >> "%INDEX_FILE%"
echo h1 { color: #2c3e50; border-bottom: 2px solid #3498db; padding-bottom: 10px; } >> "%INDEX_FILE%"
echo .report-item { padding: 10px; border-bottom: 1px solid #eee; } >> "%INDEX_FILE%"
echo .report-link { text-decoration: none; color: #2980b9; font-weight: 500; } >> "%INDEX_FILE%"
echo .report-link:hover { color: #3498db; } >> "%INDEX_FILE%"
echo ^</style^>^</head^>^<body^> >> "%INDEX_FILE%"
echo ^<div class="container"^> >> "%INDEX_FILE%"
echo ^<h1^>AI Target Detection Reports^</h1^> >> "%INDEX_FILE%"
echo ^<p^>Generated on !date! !time%^</p^> >> "%INDEX_FILE%"

set /a count=0
for %%f in ("%OUTPUT_DIR%\*.html") do (
    if /i not "%%~nxf"=="index.html" (
        set /a count+=1
        echo ^<div class="report-item"^>^<a class="report-link" href="%%~nxf"^>%%~nxf^</a^>^</div^> >> "%INDEX_FILE%"
    )
)

echo ^</div^>^</body^>^</html^> >> "%INDEX_FILE%"
echo Index created: %INDEX_FILE%
pause
goto menu
```

## Configuration File (`batch_config.ini`)

```ini
INPUT_DIR=reports
OUTPUT_DIR=html_reports
ARCHIVE_PROCESSED=false
CREATE_INDEX=true
AUTO_MONITOR=false
```

## Quick Convert Script (`quick_convert.bat`)

```batch
@echo off
setlocal enabledelayedexpansion

echo Quick Convert - All .txt to .html
echo.

set "INPUT_DIR=reports"
set "OUTPUT_DIR=html_reports"

if not exist "%INPUT_DIR%" (
    echo Creating %INPUT_DIR%...
    mkdir "%INPUT_DIR%"
)

if not exist "%OUTPUT_DIR%" (
    echo Creating %OUTPUT_DIR%...
    mkdir "%OUTPUT_DIR%"
)

set /a count=0
for %%f in ("%INPUT_DIR%\*.txt") do (
    set "filename=%%~nf"
    echo Converting: %%~nxf
    python report_processor.py "%%f" "%OUTPUT_DIR%\!filename!.html"
    if !errorlevel! equ 0 (
        echo ✓ !filename!.html
        set /a count+=1
    ) else (
        echo ✗ Failed: %%~nxf
    )
)

echo.
echo Conversion complete: !count! files processed
echo Output: %OUTPUT_DIR%
pause
```

## Usage:

1. **Place all files in same directory:**
   - `report_processor.py` (the fixed Python script)
   - `process_reports.bat` (main batch processor)
   - `process_reports_advanced.bat` (advanced version)
   - `quick_convert.bat` (simple one-time conversion)

2. **Create folder structure:**
   ```
   your_directory/
   ├── report_processor.py
   ├── process_reports.bat
   ├── process_reports_advanced.bat
   ├── quick_convert.bat
   ├── reports/           (put your .txt files here)
   └── html_reports/      (html files will be created here)
   ```

3. **Run:**
   - `quick_convert.bat` - Fast one-time conversion
   - `process_reports.bat` - Full batch processing with index
   - `process_reports_advanced.bat` - Menu-driven advanced features

The batch scripts will automatically convert every `.txt` file in the `reports` folder to its corresponding `.html` file in the `html_reports` folder, maintaining the original filenames.
