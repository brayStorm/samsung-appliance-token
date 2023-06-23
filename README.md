# samsung-appliance-token
Fetch authorization token from Samsung applicances with an API on port 8888, for local consumption with Homeassistant.

Step 0: Use SmartThings app to connect applicance to Wifi.  
Optional Step 0a: Block from internet. (VLAN, Pause indefinitley, whatever)  
  
Step 1: Download cert.pem and listener.py to a local directory.  
Step 2: Open 2 bash sessions.  
Step 3: Run listener.py   `python listener.py` , you should see a message `Listening on localhost:8889`  
Step 4: Run this command in the other window, replacing the Host IP (YY) with that of the device you are currently on, and the target IP (ZZ) with that of the appliance.  
`curl -k -H "Content-Type: application/json" -H "DeviceToken: xxxxxxxxxxx" --cert cert.pem --insecure -X POST https://192.168.ZZ.ZZ:8888/devicetoken/request -H "Host: 192.168.YY.YY:8889" -v`  
  
This will return a Authorization Bearer. Replace WW in Step 5.

Step 5: To confirm it worked, you should be able to now run the following and get json back;
`curl -m 8 -k -H "Content-Type: application/json" -H "Authorization: Bearer WWWWW" --cert cert.pem --insecure -X GET https://192.168.ZZ.ZZ:8888/devices/0/operation`

Step 6 (prior to HA 2023.6.2): Add the following to `sensors.yaml`

    # Samsung Dryer as an example, should be the same for Airconditioner, Washer, and Dishwasher
    - platform: command_line
      command: curl -m 8 -k -H 'Content-Type:application/json' -H 'Authorization:Bearer WWWWW' --cert /ssl/cert.pem --insecure --silent -X GET https://192.168.ZZ.ZZ:8888/devices/0/operation | sed -r -e 's/:\{/:\{\},/' -e 's/\}\}/\}/'
      scan_interval: 30
      name: "Dryer"
      value_template: "{{ value_json['progress'] }}"
      json_attributes:
        - state
        - progress
        - progressPercentage
        - remainingTime



Step 6 (2023.6.2 onward): Add the following to `command_line.yaml`

    # Samsung Dryer as an example, should be the same for Airconditioner, Washer, and Dishwasher
    - sensor:
        command: curl -m 8 -k -H 'Content-Type:application/json' -H 'Authorization:Bearer WWWWW' --cert /ssl/cert.pem --insecure --silent -X GET https://192.168.ZZ.ZZ:8888/devices/0/operation | sed -r -e 's/:\{/:\{\},/' -e 's/\}\}/\}/'
        scan_interval: 30
        name: "Dryer"
        value_template: "{{ value_json['progress'] }}"
        json_attributes:
          - state
          - progress
          - progressPercentage
          - remainingTime
