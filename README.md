# ESP-MATTER vscode env

This my vscode base environment for esp-matter development on vscode on macOS.

It contains:

- a docker file that install esp-idf v5.1.2 for esp32h2 and esp-matter v1.2
- all vscode config file for container development. 

vscode is setup to connect to connect using rfc2217 on JTAG port. To connect to your board
you need to have a local install of esp-idf and run `esp_rfc2217_server.py -v -p 4000 /dev/cu.<PORT>`

Note: building image can take a while, if it fails with strange error, try to increase ram allocated 
to docker 
