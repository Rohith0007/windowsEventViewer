# Windows Event Viewer Web Interface  

## Overview  
This project is a **web-based Windows Event Viewer** that retrieves event logs from remote machines using **WinRM (Windows Remote Management)**. It allows administrators to **securely fetch and analyze event logs** by providing **hostname, username, and password**, displaying them in a structured report format.

## Features  
- ✅ Fetch Windows Event Logs via **WinRM**  
- ✅ User authentication with **hostname, username, and password**  
- ✅ Interactive **UI for log analysis**  
- ✅ **Report generation** for easy viewing  
- ✅ **Bootstrap-based AdminLTE dashboard** for a modern look  

## Tech Stack  
- **Python** (Backend)  
- **Flask** (Web Framework)  
- **WinRM** (Remote communication)  
- **HTML, CSS** (Frontend)  
- **Bootstrap & AdminLTE** (UI & Dashboard Styling)  

## How It Works  
1️⃣ Enter **hostname, username, and password**  
2️⃣ Establish a **WinRM connection** to fetch event logs  
3️⃣ Display logs in an **interactive web UI**  
4️⃣ Export or view logs in a **report format**  

<p align="center">
  <img src="https://github.com/user-attachments/assets/393654bc-2a6d-4d42-bed6-a37c7edf5221" width="45%">
  <img src="https://github.com/user-attachments/assets/5cde9366-e1af-4723-b6de-0d0039c605a4" width="45%">
</p>

<p align="center">
  <img src="https://github.com/user-attachments/assets/ed4f2958-838c-4cb4-b1ce-8bd457bf85af" width="45%">
  <img src="https://github.com/user-attachments/assets/9d232e44-69f5-4531-b1c7-b27a7692527c" width="45%">
</p>

## Commands
    pip install pywinrm

    pip install flask

    pip install python-dotenv

    pip install pywinrm
    
    pip install python-evtx

    pip3 install xmltodict

    pip install Flask Flask-SQLAlchemy

    pip install flask-restx

    pip install Flask-Migrate

    winrm set winrm/config/service/auth '@{Basic="true"}'

    winrm set winrm/config/service '@{AllowUnencrypted="true"}'

## Env SetUP
    python3 -m venv venv

    source venv/bin/activate  # On macOS or Linux


## To run the app in normal Flask buit-in Server
    flask db migrate -m "Updated User model"

    flask db upgrade

    python run.py
    http://127.0.0.1:5000/

## To run the app in normal Flask and WSGI Server
    gunicorn "run:app"
    http://127.0.0.1:5000/

## WinRM Setup
    winrm quickconfig

### Configure WinRM to Use HTTPS
    $cert = New-SelfSignedCertificate -DnsName "your-server-name" -CertStoreLocation Cert:\LocalMachine\My

    New-Item -Path WSMan:\LocalHost\Listener -Transport HTTPS -Address * -CertificateThumbPrint $cert.Thumbprint -Force

### Open Firewall Ports
    New-NetFirewallRule -Name "WinRM HTTP" -DisplayName "WinRM HTTP" -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 5985

    New-NetFirewallRule -Name "WinRM HTTPS" -DisplayName "WinRM HTTPS" -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 5986

## Configure WinRM Service
    Set-Service -Name WinRM -StartupType Automatic
    Start-Service -Name WinRM


### Adjust Authentication Methods (Optional)
    winrm set winrm/config/service/auth '@{Basic="true"}'

    winrm set winrm/config/service/auth '@{Negotiate="false"}'

### Kerberos Authentication
    winrm set winrm/config/service/auth '@{Kerberos="true"}'

    winrm set winrm/config/service '@{AllowEncryption="true"}'

# Retrieving Logs on Local Machines

This section provides instructions on how to retrieve logs from local machines. The steps include fetching the username, hostname, and password, and adding them to a JSON file.

## Steps

1. **Get the Username**

   Open Command Prompt and run the following commands to retrieve the current user's username and hostname on the Windows machine:

   ```cmd
   echo %USERNAME%
   
   hostname
2. **Add it in the json file**

   In the folder app/data/modeltest

    ```json
    {
        "details": [
            {
                "hostname": "your-hostname-here",
                "username": "your-username-here",
                "password": "your-password-here"
            }
        ]
    }
    

https://www.hurryupandwait.io/blog/understanding-and-troubleshooting-winrm-connection-and-authentication-a-thrill-seekers-guide-to-adventure

https://pitstop.manageengine.com/portal/en/kb/articles/troubleshooting-winrm-errors#ErrorA_specified_logon_session_does_not_exist_It_may_already_have_been_terminated

https://www.hurryupandwait.io/blog/fixing-winrm-firewall-exception-rule-not-working-when-internet-connection-type-is-set-to-public


    ## Code
    
    #!/usr/bin/env python
    #    This file is part of python-evtx.
    #
    #   Copyright 2012, 2013 Willi Ballenthin <william.ballenthin@mandiant.com>
    #                    while at Mandiant <http://www.mandiant.com>
    #
    #   Licensed under the Apache License, Version 2.0 (the "License");
    #   you may not use this file except in compliance with the License.
    #   You may obtain a copy of the License at
    #
    #       http://www.apache.org/licenses/LICENSE-2.0
    #
    #   Unless required by applicable law or agreed to in writing, software
    #   distributed under the License is distributed on an "AS IS" BASIS,
    #   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    #   See the License for the specific language governing permissions and
    #   limitations under the License.
    #
    #   Version v0.1.1
    import os
    import json
    import xmltodict
    import Evtx.Evtx as evtx
    import Evtx.Views as e_views

    def main():
        import argparse
    
        parser = argparse.ArgumentParser(description="Dump a binary EVTX file into XML.")
        parser.add_argument("evtx", type=str, help="Path to the Windows EVTX event log file")
        parser.add_argument("hostname", type=str, help="Hostname for the output file naming")
        parser.add_argument("username", type=str, help="Username for the output file naming")
        parser.add_argument("password", type=str, help="Password for the output file naming")
        args = parser.parse_args()
    
        filename = f"{args.hostname}_{args.username}_{args.password}.json"
        output_dir = os.getcwd() + "/" + os.getenv("MODEL_OUTPUT_DIR")
        output_path = os.path.join(output_dir, filename)
        output_path = output_path.replace("/python-evtx/scripts", "")
        print(output_path)
    
        # Ensure directory exists
        os.makedirs(output_dir, exist_ok=True)
    
        with evtx.Evtx(args.evtx) as log:
            with open(output_path, 'a') as f:  # Append mode if file already exists
                print(e_views.XML_HEADER)
                f.write("[\n")  # Start of the JSON list, only if file is empty
                first_record = True
                for record in log.records():
                    if not first_record:
                        f.write(",\n")  # Add a comma before every new record except the first
                    json.dump(xmltodict.parse(record.xml()), f, indent=4)
                    first_record = False
                f.write("\n]")  # End of the JSON list
            print("</Events>")
    
    if __name__ == "__main__":
        main()
    
    
    
    
