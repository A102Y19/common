# Define the path to the requirements file
$requirementsFile = "requirements.txt"

# Read the requirements from the text file
$requirements = Get-Content -Path $requirementsFile

# Check each step and execute accordingly
foreach ($line in $requirements) {
    # Split the line into the step name and its requirement (True/False)
    $step, $required = $line -split ":\s*"

    # Perform actions based on the requirement
    switch ($step) {
        "PAK file download" {
            if ($required -eq "True") {
                Write-Host "Executing PAK file download..."
                # Step 1: Download PAK file from URL
                $sourcePath = "https://transferus.omp.com/package.pak"  # URL for downloading the PAK file
                $downloadsPath = "$env:USERPROFILE\Downloads"  # Default download path
                Invoke-WebRequest -Uri $sourcePath -OutFile "$downloadsPath\package.pak"
                Write-Host "PAK file downloaded successfully to $downloadsPath\package.pak"
            }
        }
        "transfer file" {
            if ($required -eq "True") {
                Write-Host "Executing transfer file to server..."
                # Step 2: Transfer the downloaded file to the server
                $serverPath = "\\itsascr.kk.com\mdd_hgj_hnjk"  # Remote server path
                Copy-Item -Path "$downloadsPath\package.pak" -Destination $serverPath -Force
                Write-Host "File transferred to server at $serverPath"
            }
        }
        "Copy files to RDP" {
            if ($required -eq "True") {
                Write-Host "Executing copy files to RDP..."
                # Step 3: Use PsExec to copy files to RDP server
                $rdpServer = "awsdmjhgbg009"  # RDP server name
                $rdpUsername = "utyhg"  # RDP username
                $rdpPassword = "baby09"  # RDP password
                $remotePath = "D:\OMPDump"  # Path to copy on RDP server
                $dumpFilesPath = "D:\dumpfiles"  # Path to move files to in RDP session

                # Use PsExec to copy the file
                Start-Process "psexec.exe" -ArgumentList "\\$rdpServer -u $rdpUsername -p $rdpPassword cmd /c 'copy D:\OMPDump\package.pak D:\dumpfiles'"
                Write-Host "File copied to RDP server at $remotePath"
            }
        }
        "backup" {
            if ($required -eq "True") {
                Write-Host "Executing backup..."
                # Backup Step (Example placeholder)
                $backupSource = "D:\OMPDump"  # Source folder for backup
                $backupDestination = "D:\Backup"  # Destination folder for backup

                # Example backup command (customize as needed)
                Copy-Item -Path "$backupSource\*" -Destination $backupDestination -Recurse -Force
                Write-Host "Backup completed from $backupSource to $backupDestination"
            }
        }
        default {
            Write-Host "Unknown step: $step"
        }
    }
}

# Send output as email (optional)
$EmailFrom = "your-email@example.com"
$EmailTo = "recipient@example.com"
$Subject = "Automation Script Execution Report"
$Body = "The script has been executed successfully. Please check the logs for details."
$SmtpServer = "smtp.example.com"  # Replace with your SMTP server

# Send email with results
Send-MailMessage -From $EmailFrom -To $EmailTo -Subject $Subject -Body $Body -SmtpServer $SmtpServer

Write-Host "Process completed and email sent."
