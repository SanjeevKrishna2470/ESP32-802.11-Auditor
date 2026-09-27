# Wireless Security Lab — Complete Setup Methodology

## 1. Lab Purpose

The lab was designed as a controlled environment for learning how 802.11 Wi-Fi networks operate and how an ESP32 running **ESP32 Marauder** can be used for wireless security auditing and observation.

The setup focused on:

* Understanding Wi-Fi discovery and management frames
* Identifying access points and client devices
* Understanding SSIDs and BSSIDs
* Observing beacon and deauthentication traffic
* Testing passive wireless monitoring
* Investigating ESP32 storage and peripheral support
* Exploring captive-portal concepts in an isolated environment
* Learning how a serial terminal can control an embedded security-testing platform
* Troubleshooting hardware and firmware integration

All testing was intended to use equipment and networks under the experimenter's control.

---

# 2. Hardware Used

The main hardware components were:

### ESP32

The ESP32 acted as the wireless security-testing platform.

It ran **ESP32 Marauder**, a firmware environment providing Wi-Fi/Bluetooth scanning, packet monitoring, configuration, and support for optional peripherals such as GPS and SD storage.

The ESP32 was connected to the computer through USB.

### Computer

The computer was used to:

* Connect to the ESP32 over USB
* Provide the serial terminal
* Configure and monitor the device
* Record experiment results
* Investigate firmware and filesystem behavior

### USB Data Cable

A USB data cable was required between the computer and ESP32.

It provided:

* Power
* Serial communication
* The connection used to communicate with Marauder

A charge-only cable would not provide the required serial communication.

### PuTTY

**PuTTY** was used as the serial terminal.

It provided the command-line interface through which commands were sent to the Marauder firmware.

### Wi-Fi Test Device

A separate phone, laptop, or other Wi-Fi-capable device was useful as an independent observer.

This was particularly important when testing wireless transmissions because the ESP32 should not automatically be assumed to reliably observe its own transmitted frames.

The basic architecture was:

```text
             USB
              │
              ▼
        ┌─────────────┐
        │   Computer  │
        │    PuTTY    │
        └──────┬──────┘
               │ Serial
               ▼
        ┌─────────────┐
        │    ESP32    │
        │   Marauder  │
        └──────┬──────┘
               │
          802.11 Wi-Fi
               │
        ┌──────┴──────┐
        │             │
        ▼             ▼
     Test AP      Test Client
```

Optional peripherals included:

```text
ESP32
  │
  ├── SD Card
  │
  └── GPS
```

---

# 3. ESP32 Firmware

The ESP32 was prepared with **ESP32 Marauder** rather than a normal Arduino application.

This distinction was important.

Marauder is a complete embedded firmware environment containing its own:

* Command interface
* Wi-Fi functionality
* Bluetooth functionality
* Scanning tools
* Packet-monitoring functionality
* Configuration
* Storage integration
* Peripheral support

Therefore, uploading an ordinary Arduino sketch to the ESP32 would replace the Marauder firmware.

The lab therefore treated Marauder as the operating environment of the ESP32.

---

# 4. Connecting the ESP32 to the Computer

The first physical step was connecting the ESP32 to the computer using a USB data cable.

After connecting it, the computer created a serial interface for the ESP32.

The corresponding serial/COM port was identified through the operating system's device-management tools.

The connection path was:

```text
ESP32
  │
  └── USB
       │
       └── Serial/COM Port
```

The COM port number can vary between computers and between USB connections.

---

# 5. Configuring PuTTY

PuTTY was used in **Serial** mode rather than SSH or Telnet mode.

The basic configuration consisted of:

```text
Connection type: Serial
Serial line:     ESP32 COM port
Speed:           Marauder's configured baud rate
```

The serial speed must match the baud rate expected by the installed Marauder firmware.

Once connected, PuTTY displayed the ESP32's startup messages and Marauder command interface.

This provided a direct way to communicate with the firmware without requiring a graphical interface.

---

# 6. Observing the ESP32 Boot Process

After establishing the serial connection, the ESP32 was restarted so the complete startup sequence could be observed.

The boot process provided useful diagnostic information.

Among the messages observed were:

```text
No core dump partition found!
Failed to mount SD Card
SD Card NOT Supported
Could not detect GPS baudrate
GPS Not Found
Finished settings nodes
Changing to main menu...
```

These messages demonstrated an important aspect of embedded-system testing:

**A peripheral initialization failure does not necessarily mean that the entire firmware has failed.**

In this case, Marauder continued loading even though SD-card and GPS functionality was unavailable.

The boot output was therefore treated as an important part of the diagnostic process rather than simply ignored.

---

# 7. Initial Marauder Command Interface

Once Marauder finished booting, its command interface became available through PuTTY.

The command interface acted as the control layer between the experimenter and the ESP32's wireless hardware.

The following commands were useful during initial setup and inspection:

```text
info
settings
channel
brightness
reboot
```

### `info`

Used to inspect available device/firmware information.

### `settings`

Used to inspect or configure available Marauder settings.

### `channel`

Used to inspect or configure the wireless channel.

### `brightness`

Used to adjust the device display/LED brightness where supported.

### `reboot`

Used to restart the Marauder firmware.

These commands were useful before beginning wireless experiments because they established the initial state of the device.

---

# 8. Initial Wireless Discovery

The first major experimental phase was wireless discovery.

The ESP32 was used to identify Wi-Fi networks and associated information.

The objective was initially to understand the wireless environment rather than interfere with it.

The primary discovery command used was:

```text
scanall
```

Other discovery-related functionality included:

```text
pingscan
```

The general workflow was:

```text
Start Marauder
       │
       ▼
Run wireless discovery
       │
       ▼
Collect AP/client information
       │
       ▼
Display indexed results
       │
       ▼
Inspect selected entries
```

---

# 9. Viewing Scan Results

After discovery, Marauder provided commands for examining the information that had been collected.

Relevant commands included:

```text
list -a
list -s
list -c
list -t
list -i
list -p
```

The exact information displayed depends on the Marauder build and the data collected.

Conceptually, the commands were used to inspect different categories of information such as:

* Access points
* Stations/clients
* Channels
* Captured information
* Other stored wireless entries

The `list` functionality was particularly important because it exposed the indexed results used during later experiments.

---

# 10. Selecting Discovered Entries

Marauder provided selection functionality through:

```text
select
```

The command supported selection of discovered entries using indexes or filtering, depending on the build.

The general concept was:

```text
Scan
 │
 ▼
Indexed results
 │
 ▼
Select an entry
 │
 ▼
Use the selected entry for a supported operation
```

Indexes were useful because they allowed a particular discovered object to be referenced without repeatedly entering its complete address.

For example:

```text
Index 0 → Access Point A
Index 1 → Access Point B
Index 2 → Access Point C
```

---

# 11. Understanding SSID and BSSID

During the scanning phase, an important distinction became apparent.

### SSID

The SSID is the human-readable network name.

Example:

```text
MyNetwork
```

### BSSID

The BSSID identifies a specific wireless access-point radio, normally represented by a MAC address.

Example:

```text
AA:BB:CC:DD:EE:FF
```

Multiple BSSIDs can legitimately advertise the same SSID.

This can occur with:

* Mesh systems
* Wireless extenders
* Multiple access points
* Enterprise Wi-Fi
* Multiple radio bands

Therefore, seeing duplicate SSIDs does not automatically indicate malicious activity.

---

# 12. Packet Counting

The command:

```text
packetcount
```

was useful for observing packet-related activity during wireless monitoring.

Packet counts provided a simple way to determine whether wireless traffic was being observed and whether an experiment was producing observable traffic.

This was useful when troubleshooting situations where a scan or monitoring function appeared not to be producing results.

---

# 13. Passive Wireless Monitoring

Marauder provided several passive monitoring functions.

Examples included:

```text
sniffraw
sniffbeacon
sniffprobe
sniffdeauth
sniffpmkid
sniffsae
```

These functions corresponded to different types of wireless traffic or information.

For this lab, passive monitoring was particularly useful because it allowed 802.11 behavior to be studied without intentionally disrupting other devices.

---

# 14. Beacon Monitoring

The command:

```text
sniffbeacon
```

was used to investigate beacon traffic.

Beacon frames are periodically transmitted by access points and contain information used by Wi-Fi devices to discover and understand wireless networks.

Conceptually:

```text
Access Point
     │
     │ Beacon
     ▼
Nearby Wi-Fi devices
```

Beacon information can help reveal characteristics such as:

* SSID
* BSSID
* Channel
* Signal strength
* Wireless capabilities

A separate phone or laptop was useful for independently verifying whether a test network or beacon was visible over the air.

This highlighted an important experimental principle:

> A device transmitting a wireless frame should not automatically be treated as a reliable observer of its own transmission.

---

# 15. Probe Monitoring

The command:

```text
sniffprobe
```

was used to observe probe-related wireless traffic.

Probe requests and responses are part of the Wi-Fi discovery process.

Studying them helped demonstrate that Wi-Fi discovery is not limited to access points broadcasting information; client devices can also participate in the discovery process.

This provided another perspective on how wireless devices identify available networks.

---

# 16. Passive Deauthentication Detection

One of the defensive experiments involved:

```text
sniffdeauth
```

The purpose was to **observe deauthentication frames**, rather than generate them.

A deauthentication frame is an 802.11 management frame used to terminate an existing wireless association.

During monitoring, relevant information could include:

* Transmitting MAC address
* Destination/client MAC address
* Channel
* Signal strength
* Number of observed frames
* Repeated bursts of deauthentication traffic

A single deauthentication frame is not necessarily malicious.

However, repeated bursts directed toward clients can be an indicator worth investigating.

The experiment therefore demonstrated how a wireless monitoring device can be used for detection rather than disruption.

---

# 17. Raw Packet Monitoring

The command:

```text
sniffraw
```

provided a lower-level monitoring function for wireless traffic.

This was useful for understanding that Wi-Fi communication consists of different frame types rather than simply "data packets."

The broader 802.11 frame categories include:

* Management frames
* Control frames
* Data frames

Beacon and deauthentication traffic are examples of management-frame activity.

---

# 18. Additional Wireless Monitoring

Other available monitoring functions included:

```text
sniffpmkid
sniffsae
```

These were treated as protocol-specific monitoring functions.

The purpose of documenting them was to identify the available capabilities of the Marauder environment rather than to imply that every command was required for the core laboratory experiment.

---

# 19. Captive Portal Investigation

The lab also explored the architecture of ESP32-hosted web pages.

A captive portal generally consists of:

```text
Wi-Fi connection
       │
       ▼
ESP32 access point
       │
       ▼
HTTP server
       │
       ▼
HTML page
```

Marauder builds that support the relevant functionality can expose commands such as:

```text
evilportal
evilportal -c start
evilportal -c sethtml <html.html>
```

These features were treated as a **controlled captive-portal technology investigation**.

For laboratory work, a unique test SSID and test-only information should be used.

Real credentials should not be collected.

The objective was to understand:

* How an ESP32 can host a web interface
* How HTML can be loaded from storage
* How a wireless client can interact with an embedded HTTP service
* How captive portals are architected

rather than collect credentials from real users.

---

# 20. ESP32 Memory and Storage Investigation

An important part of the lab was understanding where files and application data actually reside on an ESP32.

The ESP32 has several different types of memory/storage.

### SRAM

SRAM is primarily used for temporary runtime data.

It is not intended to permanently store HTML files.

### Flash

Flash memory retains information when the ESP32 loses power.

Firmware and filesystem partitions can reside in flash.

### Filesystem

Depending on the firmware and partition layout, a filesystem such as SPIFFS or LittleFS may provide persistent storage for files.

### SD Card

An SD card can provide additional persistent storage when the hardware and firmware support it.

The conceptual hierarchy was:

```text
                 ESP32
                   │
       ┌───────────┴───────────┐
       │                       │
     Runtime                Persistent
     memory                   storage
       │                       │
      SRAM                Flash / Filesystem
                               │
                               └── SD card*
```

`*` Only when supported and correctly configured.

---

# 21. Filesystem Investigation

Marauder provided filesystem-related functionality through commands such as:

```text
ls <directory>
```

The purpose was to investigate whether files were available through the firmware's filesystem interface.

This was particularly relevant to the captive-portal investigation because HTML content may need to be stored persistently rather than kept entirely in runtime memory.

An important observation from the lab was that the command interface did not behave like a normal Linux terminal.

For example, commands such as:

```text
ls
```

should not automatically be interpreted as access to the ESP32's entire flash filesystem.

The exact filesystem behavior depends on the Marauder build, partition layout, and hardware configuration.

---

# 22. SD Card Investigation

The Marauder boot log indicated that the SD card could not be mounted.

The relevant messages included:

```text
Failed to mount SD Card
SD Card NOT Supported
```

This made SD-card functionality another useful troubleshooting exercise.

Possible causes investigated included:

* Unsupported card
* Incorrect filesystem
* Incorrect wiring
* Incorrect chip-select configuration
* Board-specific GPIO differences
* Firmware/board mismatch
* SPI configuration conflicts
* Power or signal-integrity issues

The important methodological lesson was that hardware peripherals must be matched to the firmware's expected board configuration.

---

# 23. GPS Investigation

The firmware also attempted to initialize GPS hardware.

The boot output indicated:

```text
Could not detect GPS baudrate
GPS Not Found
```

Marauder also provided GPS-related commands such as:

```text
gpsdata
gps
```

and GPS tracking functionality such as:

```text
gpstracker
```

The absence of GPS hardware did not prevent the core Marauder interface from starting.

This demonstrated how optional peripherals can be tested independently from the core ESP32 functionality.

---

# 24. Configuration and Device Control

Several commands were useful for controlling the general state of the Marauder device.

Examples included:

```text
settings
channel
brightness
reboot
```

The purpose of these commands was to establish and maintain a predictable test environment.

For example:

```text
settings
```

could be used to inspect available configuration options, while:

```text
channel
```

could be used when an experiment required a particular wireless channel.

Finally:

```text
reboot
```

provided a simple way to return the device to a fresh startup state.

---

# 25. Troubleshooting Methodology

Rather than changing several variables simultaneously, the lab followed a component-by-component troubleshooting approach.

The general methodology was:

```text
Observe
   │
   ▼
Identify failure
   │
   ▼
Separate hardware/software possibilities
   │
   ▼
Change one variable
   │
   ▼
Restart/test
   │
   ▼
Compare results
```

For example, if an SD card failed to mount, possible explanations were separated into:

```text
Card
 │
 ├── Format
 ├── Capacity
 └── Compatibility

Hardware
 │
 ├── Wiring
 ├── Power
 └── GPIO configuration

Firmware
 │
 ├── Board target
 ├── Partition layout
 └── SD implementation
```

This prevented the investigation from becoming a collection of unrelated guesses.

---

# 26. Why PuTTY Was Important

PuTTY was more than just a way to type commands.

It provided visibility into the embedded system.

Through the serial interface, the experimenter could see:

* Boot messages
* Firmware initialization
* Hardware errors
* Marauder commands
* Scan results
* Runtime information
* Diagnostic output

This made the ESP32 effectively observable from the computer.

The overall control path was:

```text
Experimenter
      │
      ▼
    PuTTY
      │
      ▼
USB Serial
      │
      ▼
    ESP32
      │
      ▼
   Marauder
      │
      ▼
Wi-Fi/Bluetooth hardware
```

---

# 27. Laboratory Observation Workflow

The overall experiment followed a repeatable workflow.

## Phase 1 — Prepare

1. Connect the ESP32 to the computer.
2. Identify the ESP32 serial port.
3. Open PuTTY.
4. Select **Serial** connection mode.
5. Enter the ESP32's COM port.
6. Set the baud rate expected by the Marauder firmware.
7. Connect.

## Phase 2 — Verify

After connecting to the ESP32:

```text
info
settings
```

were used where appropriate to inspect the device.

The ESP32 was then restarted if necessary:

```text
reboot
```

The boot output was recorded.

Particular attention was paid to:

```text
No core dump partition found!
Failed to mount SD Card
SD Card NOT Supported
Could not detect GPS baudrate
GPS Not Found
```

The objective was to distinguish **nonfatal peripheral errors** from an actual firmware failure.

## Phase 3 — Discover

The wireless environment was then examined using:

```text
scanall
```

The resulting information could be inspected using relevant listing commands such as:

```text
list -a
list -s
list -c
```

The discovered APs and stations were then examined conceptually through their:

* SSIDs
* BSSIDs
* Channels
* Signal levels
* Other available information

## Phase 4 — Select and Inspect

Where supported by the build, discovered entries could be selected using:

```text
select
```

and further information could be examined using:

```text
info
```

Packet activity could be monitored using:

```text
packetcount
```

## Phase 5 — Monitor

Passive monitoring functions were then used according to the experiment:

```text
sniffbeacon
sniffprobe
sniffraw
sniffdeauth
```

The results were compared against observations from a separate Wi-Fi device where appropriate.

## Phase 6 — Investigate Storage

Storage functionality was investigated using available filesystem commands such as:

```text
ls <directory>
```

The SD-card status was also evaluated based on the boot messages.

## Phase 7 — Investigate Optional Hardware

GPS functionality was inspected using commands such as:

```text
gpsdata
gps
```

The absence of a GPS receiver was recorded as an environmental limitation rather than treated as a core firmware failure.

## Phase 8 — Document

For each experiment, the following information was recorded:

```text
Date/time
Hardware
Firmware version
Command/function tested
Channel
Observed devices
Observed behavior
Errors
Expected result
Actual result
Conclusion
```

---

# 28. Command Reference Used in the Lab

The following table summarizes the main commands relevant to the laboratory methodology.

| Command          | Role in the lab                                  |
| ---------------- | ------------------------------------------------ |
| `info`           | Inspect available device/firmware information    |
| `settings`       | Inspect/configure Marauder settings              |
| `channel`        | Inspect/configure the wireless channel           |
| `brightness`     | Configure display/LED brightness where supported |
| `reboot`         | Restart the ESP32/Marauder                       |
| `scanall`        | Perform wireless discovery                       |
| `pingscan`       | Perform supported network/device discovery       |
| `list -a`        | Inspect access-point results                     |
| `list -s`        | Inspect station/client results                   |
| `list -c`        | Inspect channel-related results                  |
| `list -t`        | Inspect supported result information             |
| `list -i`        | Inspect supported information/index data         |
| `list -p`        | Inspect supported packet/result information      |
| `select`         | Select/filter discovered entries                 |
| `packetcount`    | Observe packet activity                          |
| `sniffraw`       | Passively monitor raw 802.11 traffic             |
| `sniffbeacon`    | Passively observe beacon frames                  |
| `sniffprobe`     | Passively observe probe traffic                  |
| `sniffdeauth`    | Passively detect deauthentication frames         |
| `sniffpmkid`     | Monitor supported PMKID-related traffic          |
| `sniffsae`       | Monitor supported SAE-related traffic            |
| `gpsdata`        | Inspect GPS status/data                          |
| `gps`            | Access supported GPS functionality               |
| `ls <directory>` | Inspect supported filesystem contents            |

The exact behavior and arguments of individual commands can vary between Marauder versions and hardware targets. Therefore, the command reference should be treated as documentation of the lab environment rather than a universal command reference for every Marauder build.

---

# 29. Experimental Safety

The experiments were designed around equipment controlled by the experimenter.

A safe laboratory arrangement is:

```text
        LAB ENVIRONMENT

   ┌──────────────────────┐
   │ ESP32 / Marauder     │
   │                      │
   │ Controlled Wi-Fi     │
   │ test network         │
   └──────────┬───────────┘
              │
        Test devices
```

A dedicated test SSID should be used for experiments involving access-point behavior.

Real credentials should not be collected.

Passive monitoring should be performed only where the experimenter has authorization.

Experiments involving disruptive wireless traffic should not be performed against networks or devices that are not part of the authorized test environment.

---

# 30. Final Lab Architecture

The complete setup can therefore be summarized as:

```text
                         ┌──────────────────┐
                         │     COMPUTER     │
                         │                  │
                         │      PuTTY       │
                         └────────┬─────────┘
                                  │
                              USB Serial
                                  │
                                  ▼
                         ┌──────────────────┐
                         │      ESP32       │
                         │                  │
                         │ ESP32 Marauder   │
                         └────────┬─────────┘
                                  │
                     ┌────────────┼────────────┐
                     │            │            │
                     ▼            ▼            ▼
                  Wi-Fi        Bluetooth     GPIO
                  Radio          Radio       Devices
                     │
             ┌───────┴────────┐
             │                │
             ▼                ▼
        Test Access       Test Client
           Point          Phone/Laptop

Optional peripherals:

        ESP32
          │
     ┌────┴─────┐
     ▼          ▼
   SD Card     GPS
```

The computer and PuTTY formed the **control and observation layer**, while the ESP32 running Marauder formed the **wireless experimentation layer**.

This separation made the lab useful for learning both sides of embedded wireless security: the **software/command interface** and the **actual 802.11 behavior occurring over the air**.

---

# 31. Methodology Summary

The central methodology of the lab was:

**Connect → Observe → Configure → Discover → Monitor → Verify → Troubleshoot → Document.**

The ESP32 provided the wireless experimentation platform, Marauder provided the security-testing functionality, and PuTTY provided the serial control and diagnostic interface.

The main command workflow was:

```text
PuTTY
  │
  ▼
Connect to ESP32
  │
  ▼
Observe boot output
  │
  ├── info
  ├── settings
  └── channel
  │
  ▼
Wireless discovery
  │
  └── scanall
        │
        ▼
      list
        │
        ▼
      select
        │
        ▼
Wireless observation
  │
  ├── sniffbeacon
  ├── sniffprobe
  ├── sniffraw
  └── sniffdeauth
        │
        ▼
Record observations
        │
        ▼
Troubleshoot hardware/storage
        │
        ├── gpsdata
        └── ls <directory>
        │
        ▼
Document results
```

Using a second Wi-Fi device as an independent observer was especially valuable because it allowed wireless behavior produced by the ESP32 to be verified externally rather than relying exclusively on the ESP32 itself.

The laboratory therefore combined:

* Embedded hardware
* Serial communication
* PuTTY
* ESP32 firmware
* ESP32 Marauder
* Wi-Fi protocols
* Wireless monitoring
* SSID/BSSID analysis
* Management-frame analysis
* Filesystem/storage concepts
* Hardware troubleshooting
* Experimental documentation

into one reproducible wireless-security learning environment.
