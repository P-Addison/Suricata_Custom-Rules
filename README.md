# Applying Custom Rules in Suricata

## Scenario  

In this scenario, I am a security analyst responsible for monitoring traffic on my employer's network. I will be required to configure Suricata and use it to trigger alerts.

**First**, I'll explore custom rules in Suricata.

**Second**, I'll run Suricata with a custom rule to trigger it and examine the output logs in the `fast.log` file.

**Finally**, I’ll examine the additional output that Suricata generates in the standard `eve.json` log file.

Let's define the files I'll be working with in this lab activity:

  * The `sample.pcap` file is a packet capture file that contains an example of network traffic data, which I’ll use to test the Suricata rules. This will allow me to simulate and repeat the exercise of monitoring network traffic.

  * The `custom.rules` file contains a custom rule when the lab activity starts. I’ll add rules to this file and run them against the network traffic data in the `sample.pcap` file.

  * The `fast.log` file will contain the alerts that Suricata generates. The fast.log file is empty when the lab starts. Each time I test a rule, or set of rules, against the sample network traffic data, Suricata adds a new alert line to the fast.log file when all the conditions in any of the rules are met. The `fast.log` file can be located in the `/var/log/suricata` directory after Suricata runs. The fast.log file is considered to be a deprecated format and is not recommended for incident response or threat hunting tasks but can be used to perform quick checks or tasks related to quality assurance.

  * The `eve.json` file is the main, standard, and default log for events generated by Suricata. It contains detailed information about alerts triggered, as well as other network telemetry events, in JSON format. The `eve.json` file is generated when Suricate runs, and can also be located in the `/var/log/suricata directory`.

When I create a new rule, I'll need to test the rule to confirm whether or not it worked as expected. I can use the `fast.log` file to quickly compare the number of alerts generated each time I run Suricata to test a signature against the `sample.pcap` file.

## Task 1: Examine a Custom Rule in Suricata  
The `/home/analyst` directory contains a `custom.rules` file that defines the network traffic rules that Suricata captures.

In this task, I’ll explore the composition of the Suricata rule defined in the `custom.rules` file.

Use the cat command to display the rule in the `custom.rules` file:

      cat custom.rules  

The command returns the rule as the output in the shell:

![T1 1](https://github.com/user-attachments/assets/eb6f157c-ba77-493f-9d70-b33caa1db358)

This rule consists of three components: an action, a header, and rule options. Let's examine each component in more detail.

### Action

![T1 2](https://github.com/user-attachments/assets/35230e7a-fca9-46fe-85c7-333b6fff2675)

The action is the first part of the signature. It determines the action to take if all conditions are met. Common actions across network intrusion detection system (NIDS) rule languages include alert, drop, pass, and reject.

In our example, the file contains a single alert as the action. The alert keyword instructs Suricata to alert on selected network traffic. The IDS inspects the traffic packets and sends out an alert if it matches.

Note that the drop action also generates an alert but drops the traffic. A drop action only occurs when Suricata runs in IPS mode. The pass action allows the traffic to pass through the network interface and can be used to override other rules. The reject action does not allow the traffic to pass; instead, a TCP reset packet is sent, and Suricata drops the matching packet.

Rule order refers to the order in which rules are evaluated by Suricata. Rules are loaded in the order in which they are defined in the configuration file. However, Suricata processes rules in a different default order: pass, drop, reject, and alert. Rule order affects the final verdict of a packet.

### Header

![T1 3](https://github.com/user-attachments/assets/632e7786-e798-4b4a-b553-9f4ed48510a5)

The next part of the signature is the header. The header defines the signature’s network traffic, including attributes such as protocols, source and destination IP addresses, ports, and traffic direction.

The next field after the action keyword is the protocol field. In our example, the protocol is http, which means the rule applies only to HTTP traffic.

The parameters to the protocol http field are `$HOME\_NET any \-\> $EXTERNAL\_NET any`. The arrow indicates the direction of the traffic coming from the `$HOME\_NET` and going to the destination IP address `$EXTERNAL\_NET`.

`$HOME\_NET` is a Suricata variable defined in `/etc/suricata/suricata.yaml` that I can use in my rule definitions as a placeholder for my local or home network to identify traffic that connects to or from systems within my organization. In this lab, `$HOME\_NET` is defined as the 172.21.224.0/20 subnet. The word any means that Suricata catches traffic from any port within the `$HOME\_NET` network.

The $ symbol indicates the start of a variable. Variables are used as placeholders to store values.

So far, this signature triggers an alert when it detects any HTTP traffic leaving the home network and going to the external network.

### Rule Options

![T1 4](https://github.com/user-attachments/assets/87b6dd7f-3ed8-43cf-868a-f190048c6545)

The many available rule options allow me to customize signatures with additional parameters. Configuring rule options helps narrow down network traffic so I can find exactly what I’m looking for. Rule options are typically enclosed in a pair of parentheses and separated by semicolons.

Let's further examine the rule options in our example:

The msg: option provides the alert text. In this case, the alert will print out the text “GET on wire”, which specifies why the alert was triggered.

The `flow:established,to\_server` option determines that packets from the client to the server should be matched. (A server is defined as the device responding to the initial `SYN` packet with a `SYN-ACK` packet.)

The `content:"GET"` option tells Suricata to look for the word GET in the content of the `http.method` portion of the packet.

The `sid:12345` (signature ID) option is a unique numerical value that identifies the rule.

The `rev:3` option indicates the signature's revision, which is used to identify the signature's version. Here, the revision version is 3\.

To summarize, this signature triggers an alert whenever Suricata observes the text GET as the HTTP method in an HTTP packet from the home network to the external network.

## Task 2: Trigger a Custom Rule in Suricata

Now that I am familiar with the composition of the custom Suricata rule, I must trigger this rule and examine the alert logs that Suricata generates.

1. List the files in the `/var/log/suricata\` folder:
   
       ls \-l /var/log/suricata  

![T2 1](https://github.com/user-attachments/assets/ef37c95c-0f4f-4d45-b6fe-e86db4a042cb)

Note that before running Suricata, there are no files in the `/var/log/suricata` directory.

Run Suricata using the `custom.rules` and `sample.pcap` files:

    sudo suricata \-r sample.pcap \-S custom.rules \-k none

![T2 2](https://github.com/user-attachments/assets/eefafbb8-1790-46e7-b61e-52f4b0012d33)

This command starts the Suricata application and processes the `sample.pcap` file using the rules in the `custom.rules` file. It returns an output stating how many packets were processed by Suricata.

Now I’ll further examine the options in the command:

The `\-r sample.pcap` option specifies an input file to mimic network traffic. In this case, the sample.pcap file.

The `\-S custom.rules` option instructs Suricata to use the rules defined in the custom.rules file.

The `\-k none` option instructs Suricata to disable all checksum checks.

Checksums are a way to detect if a packet has been modified in transit. Because I am using network traffic from a sample packet capture file, I won't need Suricata to check the integrity of the checksum.

Suricata adds a new alert line to the `/var/log/suricata/fast.log` file when all the conditions in any of the rules are met.

List the files in the `/var/log/suricata` folder again:

    ls \-l /var/log/suricata

![T2 3](https://github.com/user-attachments/assets/1f48c035-6399-4093-860f-2a322250739c)

Note that after running Suricata, there are now four files in the `/var/log/suricata` directory, including the `fast.log` and `eve.json` files. I'll examine these files in more detail.

Use the `cat` command to display the `fast.log` file generated by Suricata:
  
    cat /var/log/suricata/fast.log

![T2 4](https://github.com/user-attachments/assets/05fee4b6-d648-48aa-bcad-93fa63b3cc6a)

Each line or entry in the fast.log file corresponds to an alert generated by Suricata when it processes a packet that meets the conditions of an alert-generating rule. Each alert line includes the message that identifies the rule that triggered the alert, as well as the source, destination, and direction of the traffic.

### Task 3: Examine eve.json Output  
In this task, I must examine the additional output that Suricata generates in the `eve.json` file. As previously mentioned, this file is located in the `/var/log/suricata/ directory`.

The `eve.json` file is the standard and main Suricata log file and contains a lot more data than the `fast.log` file. This data is stored in a JSON format, which makes it much more useful for analysis and processing by other applications.

Use the `cat` command to display the entries in the `eve.json` file:

    cat /var/log/suricata/eve.json

![T4 1](https://github.com/user-attachments/assets/77a42a21-e3e4-4ab7-aee4-5cd9c6269ca6)

The output returns the raw content of the file. I'll notice that there is a lot of data returned that is not easy to understand in this format.

Use the `jq` command to display the entries in an improved format:

    jq . /var/log/suricata/eve.json | less

![T4 2](https://github.com/user-attachments/assets/682b1b4a-2622-44f5-8015-f54495a7a937)

Press `Q` to exit the less command and to return to the command-line prompt. Now it is much easier to read the output as opposed to the cat command output.

Use the `jq` command to extract specific event data from the `eve.json` file:

    jq \-c "\[.timestamp,.flow\_id,.alert.signature,.proto,.dest\_ip\]" /var/log/suricata/eve.json

![T4 3](https://github.com/user-attachments/assets/eb2726f7-09b6-456a-b9bf-054e0567fd3c)

The `jq` command above extracts the fields specified in the list in the square brackets from the JSON payload. The fields selected are the timestamp (`.timestamp`), the flow id (`.flow\_id`), the alert signature or msg (`.alert.signature`), the protocol (`.proto`), and the destination IP address (`.dest\_ip`).

The following is an example of the output of the command above. The `.flow\_id` is the long numeric field highlighted in orange in each row returned.

Use the `jq` command to display all event logs related to a specific `flow\_id` from the eve.json file. The `flow\_id` value is a 16-digit number and will vary for each of the log entries. Replace X with any of the `flow\_id` values returned by the previous query:

    jq "select(.flow\_id==X)" /var/log/suricata/eve.json

![T4 4](https://github.com/user-attachments/assets/afe6897d-7f77-4a5b-a7c6-d5d8d5b35b87)

A network flow refers to a sequence of packets between a source and destination that share common characteristics such as IP addresses, protocols, and more. In cybersecurity, network traffic flows help analysts understand the behavior of network traffic to identify and analyze threats. Suricata assigns a unique `flow\_id` to each network flow. All logs from a network flow share the same `flow\_id`. This makes the `flow\_id` field a useful field for correlating network traffic that belongs to the same network flows.

### Conclusion  

I’ve completed this activity and should be able to use Suricata to trigger alerts on network traffic. I now have practical experience in running Suricata to:
* Create custom rules and run them in Suricata
* Monitor traffic captured in a packet capture file
* Examine the `fast.log` and `eve.json` output


