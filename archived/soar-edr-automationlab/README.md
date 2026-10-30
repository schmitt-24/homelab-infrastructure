# SOAR and EDR Homelab


## Objective

The Detection Lab project aimed to establish a controlled environment for simulating and detecting cyber attacks. The primary focus was to monitor and analyze data with LimaCharlie as a Endpoint Detection and Response (EDR) system, to mimic real-world attack scenarios with the HackingTool ‘LaZagne’. This hands-on experience was designed to deepen understanding of credential security, and defensive strategies.

### Tools Used

- LimaCharlie as an EDR.
- Tines as a SOAR.
- Slack as a communication program.
- Windows 10 2022 as a Server.

## Construction

Step 1:
</br>Download LimaCharlie Sensor. Connect LimaCharlie over the Powershell with the Server like so:
.\hcp_win_x64_release_4.29.0.exe -i < insert here installation key from limacharlie >
Then Implement HackTool in Server.
<div>
  <img src="/soar-edr-automationlab/soar_edr_automatedlab/img/construction/step_one.png" alt="First Step">
</div>

Step 2:
</br>Create Automation rule for detecting the preferred HackTool like so:

</br>events:
  - NEW_PROCESS
  - EXISTING_PROCESS
</br>op: and
</br>rules:
  - op: is windows
  - op: or
    </br>rules:
      - case sensitive: false
        </br>op: ends with
        </br>path: event/FILE_PATH
        </br>value: lazagne.exe
      - case sensitive: false
        </br>op: ends with
        </br>path: even/COMMAND_LINE
        </br>value: all
      - case sensitive: false
        </br>op: contains
        </br>path: event/COMMAND_LINE
        </br>value: lazagne
      - case sensitive: false
        </br>op: is
        </br>path: event/HASH
        </br>value: 467e49f1f795c1b08245ae621c59cdf06df630fc1631dc0059da9a032858a486’
</br>
Step 3: 
</br>Create a Slack Channel and a Tines Playbook.
<div>
  <img src="/soar-edr-automationlab/soar_edr_automatedlab/img/construction/step_two.png" alt="Second Step">
</div>
</br>
Step 4:
</br>Build your Tines playbook.
<div>
  <img src="/soar-edr-automationlab/soar_edr_automatedlab/img/tines_playbook/SOAR_EDR_playbook.png" alt="Playbook">
</div>
</br>
Step 5:
</br>Connect the Tines Playbook via credentials with LimaCharlie and slack and test it in Tines.
<div>
  <img src="/soar-edr-automationlab/soar_edr_automatedlab/img/construction/step_three.png" alt="Third Step">
</div>
</br> 
Full example of Construction:
<div>
  <img src="/soar-edr-automationlab/soar_edr_automatedlab/img/construction/soar_edr.png" alt="Full Example">
</div>
