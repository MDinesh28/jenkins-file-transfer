# File Sharing Between Linux Jenkins Server and Windows Server

This guide outlines how to set up file sharing between a Linux Jenkins server and a Windows server using OpenSSH and passwordless authentication.

---

## Prerequisites

- A Linux server with Jenkins installed.
- A Windows server with OpenSSH installed and configured.
- `Publish Over SSH` plugin installed in Jenkins.

---

## Setup Steps

### 1. Confirm OpenSSH Setup
#### Linux:
- Install and start OpenSSH:
  ```bash
  sudo apt install openssh-server
  sudo systemctl start ssh
  sudo systemctl enable ssh
  ```

#### Windows:
- Install OpenSSH Server via Windows Features or PowerShell.
- Start and configure OpenSSH Server:
  ```powershell
  Start-Service sshd
  Set-Service -Name sshd -StartupType Automatic
  ```

---

### 2. Configure SSH Key-Based Authentication
#### Generate SSH Key Pair on Linux:
1. Run the following command to generate a key pair:
   ```bash
   ssh-keygen -t rsa -b 2048
   ```
2. Move or copy the private key for Jenkins:
   ```bash
   sudo mkdir -p /var/lib/jenkins/.ssh
   sudo cp /root/.ssh/id_rsa /var/lib/jenkins/.ssh/id_rsa
   sudo chown -R jenkins:jenkins /var/lib/jenkins/.ssh
   sudo chmod 700 /var/lib/jenkins/.ssh
   sudo chmod 600 /var/lib/jenkins/.ssh/id_rsa
   ```

#### Transfer the Public Key to Windows:
1. Append the public key to `administrators_authorized_keys`:
   ```bash
   ssh-copy-id -i /var/lib/jenkins/.ssh/id_rsa.pub Administrator@<Windows_IP>
   ```
2. Alternatively, manually copy the public key's content from `/var/lib/jenkins/.ssh/id_rsa.pub` and paste it into `C:\Users\Administrator\.ssh\administrators_authorized_keys`.

#### Verify SSH Connection:
1. Test the connection:
   ```bash
   ssh Administrator@<Windows_IP>
   ```
2. Ensure it connects without prompting for a password.

---

### 3. Configure Jenkins for File Sharing
#### Install the `Publish Over SSH` Plugin:
1. Navigate to **Manage Jenkins** → **Manage Plugins** → Install the `Publish Over SSH` plugin.

#### Add SSH Configuration:
1. Go to **Manage Jenkins** → **Configure System** → Scroll to **Publish Over SSH**.
2. Add the Windows server configuration:
   - **Name:** `WindowsServer`
   - **Hostname:** `<Windows_IP>`
   - **Username:** `Administrator`
   - **Key:** Paste the contents of `/var/lib/jenkins/.ssh/id_rsa`.

---

### 4. Use SSH Configuration in Your Jenkins Pipeline
#### Example Pipeline Script:
```groovy
pipeline {
    agent any
    stages {
        stage('Transfer File to Windows') {
            steps {
                script {
                    // Transfer the file to the Windows server
                    sh '''scp -o StrictHostKeyChecking=no /home/jenkins/ala.zip Administrator@<Windows_IP>:C:/md/'''
                }
            }
        }
    }
}
```

---

### 5. Test the Jenkins Pipeline
1. Trigger the Jenkins job.
2. Verify that the file has been successfully transferred to the specified directory on the Windows server.

---

## Notes
- Ensure that the Jenkins user has the necessary permissions to access the files and directories involved in the transfer.
- For troubleshooting, verify SSH connection manually and check the Jenkins job logs for errors.

---

This setup ensures seamless file sharing between Linux and Windows using Jenkins and OpenSSH.



#  Pipeline Transfer, Extract, and Cleanup ZIP File on Windows Server


1. **Transfer a ZIP File**:  
   It securely copies a ZIP file (`ala.zip`) from the Linux Jenkins server to a specified directory (`C:/md/`) on the Windows server using SCP.  

2. **Unzip and Delete**:  
   Once the ZIP file is transferred, it executes a PowerShell command on the Windows server to:  
   - Extract the contents of the ZIP file into the same directory.  
   - Remove the ZIP file after extraction to clean up.  

---

### Explanation of Key Parts

- **SCP Command**:  
  The `scp` command copies files over SSH. The `-o StrictHostKeyChecking=no` option disables SSH host key verification, ensuring the process doesn't stall due to unknown hosts.  

- **SSH and PowerShell Command**:  
  The `ssh` command connects to the Windows server, and the subsequent PowerShell command performs two operations:  
  - `Expand-Archive`: Unzips the file.  
  - `Remove-Item`: Deletes the ZIP file after extraction.  

- **PowerShell Syntax**:  
  Backslashes (`\\`) are used to escape characters in the Jenkins shell.  

---

### Suggestions for Use
- **Verify Paths**: Ensure the directories (`C:/md/`) exist on the Windows server. If not, modify the script to create them or adjust the paths.  
- **Permissions**: Ensure the `Administrator` user has the required permissions to perform the actions on the target directories.  
- **Testing**: Test each stage separately to ensure proper functionality.  


