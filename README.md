

name: RDP

on: workflow_dispatch

jobs:
  build:

    runs-on: windows-latest
    timeout-minutes: 9999

    steps:
    - name: Downloading ngrok.
      run: |
        Invoke-WebRequest https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-windows-amd64.zip -OutFile ngrok.zip
        Invoke-WebRequest https://raw.githubusercontent.com/HaeImAlan/WindowsRDP/master/start.bat -OutFile start.bat
        Invoke-WebRequest https://raw.githubusercontent.com/HaeImAlan/WindowsRDP/master/wallpaper.bat -OutFile wallpaper.bat
        Invoke-WebRequest https://raw.githubusercontent.com/HaeImAlan/WindowsRDP/master/loop.bat -OutFile loop.bat
    - name: Extracting Ngrok File.
      run: Expand-Archive ngrok.zip
    - name: Connect via ngrok.
      run: .\ngrok\ngrok.exe authtoken $Env:NGROK_AUTH_TOKEN
      env:
        NGROK_AUTH_TOKEN: ${{ secrets.NGROK_AUTH_TOKEN }}
    - name: Downloading anydesk.
      run: |
        Invoke-WebRequest https://download.anydesk.com/AnyDesk.exe -OutFile Anydesk.exe
      env:
        NGROK_AUTH_TOKEN: ${{ secrets.NGROK_AUTH_TOKEN }}
    - name: Enabling RDP Access.
      run: | 
        Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server'-name "fDenyTSConnections" -Value 0
        Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
        Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "UserAuthentication" -Value 1
        copy Anydesk.exe  C:\Users\Public\Desktop\Anydesk.exe
    - name: Creating Tunnel.
      run: Start-Process Powershell -ArgumentList '-Noexit -Command ".\ngrok\ngrok.exe tcp --region ap 3389"'
    - name: Connect to the RDP.
      run: cmd /c start.bat
    - name: Success! You may now close this tab.
      run: cmd /c loop.bat
      
