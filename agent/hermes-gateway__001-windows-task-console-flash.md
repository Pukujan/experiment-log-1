hermes-gateway__001-windows-task-console-flash
tags: hermes, gateway, windows, scheduled-task, powershell, console-window

symptom: Hermes gateway start flashes/opens Windows shell windows
active profile: docker
good task: Hermes_Gateway_docker
good task action: wscript.exe //B //Nologo "C:\Users\pujan\AppData\Local\hermes\profiles\docker\gateway-service\Hermes_Gateway_docker.vbs"
bad task: Hermes_Gateway
bad task action: C:\Users\pujan\AppData\Local\hermes\gateway-service\Hermes_Gateway.cmd
bad task last result: -1073741510
expected default action: wscript.exe //B //Nologo "C:\Users\pujan\AppData\Local\hermes\gateway-service\Hermes_Gateway.vbs"
nonadmin repair failed: schtasks /Create /TN Hermes_Gateway /TR "wscript.exe //B //Nologo \"C:\Users\pujan\AppData\Local\hermes\gateway-service\Hermes_Gateway.vbs\"" /SC ONLOGON /RL LIMITED /F -> ERROR: Access is denied.
admin repair succeeded: hermes --profile default gateway install --force --start-on-login --no-start-now
fixed default task action: wscript.exe //B //Nologo "C:\Users\pujan\AppData\Local\hermes\gateway-service\Hermes_Gateway.vbs"
verify default task: schtasks /Query /TN Hermes_Gateway /V /FO LIST
verify docker task: schtasks /Query /TN Hermes_Gateway_docker /V /FO LIST
verify active gateway: hermes --profile docker gateway status --deep
current good status: Hermes_Gateway_docker running pid 53384, pid file present, lock file held, gateway_state running, telegram connected
current non-window issue: discord retrying because privileged intents are not enabled in Discord developer portal
