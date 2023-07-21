The package was written in Italian, but can be easily translated. As for the translation of the README, we relied on automatic translators and apologize for any translation errors.

# HomeAssiatant Alarm management Alarm control panel

Complete and modular package for alarm management in HomeAssistat.

https://github.com/Home-Assistant-Pro-Team/Gestione-Allarme-Home-Assiatant-Alarm/assets/62516592/c7a11ed0-9c74-4b61-ac54-cbe815ff5645

This project focuses on using HomeAssistant's [alarm_control_panel](https://www.home-assistant.io/integrations/manual/) platform, with the goal of implementing features that traditional alarms do not offer. **It is important to note that this does not mean that the alarm created is more reliable than a commercial alarm**. However, with the use of this package, it is possible to implement some additional functions not provided by traditional alarms.

The installation of this package is designed to be as simple as possible, although some parts require specific customization. The special feature of the design is that it tries to make alarm management as dynamic as possible; for example, it is not necessary to declare alarm sensors.

To run the alarm, simply use the alarm_control_panel folder, which manages the alarm sensors.

However, this project goes beyond the simple use of the "alarm_control_panel" folder and offers more comprehensive alarm management. In the following sections, we will explain in detail the functions of each part of the project.

It is important to note that the only mandatory folders to use are "alarm control panel" and "custom_template," while the other files are optional. However, if you have the required devices, we recommend the use of all available files to enhance the alarm functionality.
### **Index**
- [Requisiti](#requisiti)
- [Intro](#intro)
- [Alarm control panel](#alarm-control-panel)
	- [General alarm](#general-alarm)
	- [Armed night](#armed-night-e-armed-away)
	- [Armed away](#armed-night-e-armed-away)
- [Skill device](#skill-device)
	- [Notifiche](#notifiche)
		- [Voip](#notifiche-voip)
		- [Media Player](#notifiche-media_player)
		- [Push](#notifiche-push)
	- [Battery status alarm](#battery-status-alarm)
	- [Campanello](#campanello)
	- [Sirena](#sirena)
	- [Controlcode](#controlcode)
		- [General code](#general-code)
		- [With card](#with-card)
		- [Keypad](#keypad--tastierino-esterno)
	- [Detect jummer](#detect-jummer)
	- [Led allarme](#led-allarme)
	- [NFC](#nfc)
	- [Smart lock](#smart-lock)
		- [NFC](#smart-lock-nfc)
		- [Keypad](#smart-lock-keypad)
	- [Person](#person)
	- [Detached](#detached)
		- [Shelly](#detached-shelly)
		- [General (Shelly ed Esphome)](#general-detached-alarm)
	- [Status ups](#ups-alarm)
	- [Status router](#router-alarm)
	- [Notifiche](#notifiche)
	- [CCTV](#cctv)
- [Scene](#scene)
	- [Action alarm](#action-alarm)
	- [Insert armed away](#insert-armed-away)
	- [Insert armed night](#insert-armed-night)
	- [External button alarm](#external-button-alarm)
- [Card](#card)
- [ChangeLog](#change-log)
### **Requisiti**The package is organized into several folders.

In YAML files where "node_anchors" are present, you need to customize the document by adding your own entities. For example:
```
homeassistant:
  customize:
    package.node_anchors:
        Allarme: &alarm alarm_control_panel.home_alarm
```
The first thing to do is to load the "custom_templates" folder in the "conf" directory. 

Also, you need to customize the entities in the "alarm.jinja" file to suit your needs.
- **alarm.jinja**
	In this file you need to enter codes and the alarm entity used

	```
	{% macro state_alarm() %}
		{

	{# Inserire la propria entità alarm_control_panel creata con file general_alarm #}
			"alarm": "alarm_control_panel.home_alarm",

	{# Inserire il codice dell'allarme #}
			"code": "1234",

	{# Inserire il codice di servizio che può essere attivato e disattivato da UI. #}
	{# ATTENZIONE: Se il codice viene cambiato occorre riavviare HA #}
			"code_service" : "9877",

	{# Inserire il codice di emergenza per disattivare l'allarme ma inviare comunque notifica #}
			"emergency_code": "5555",

	{# Numero di tentativi per digitare codice allarme #}
			"alarm_code_attempts": "3",

	{# Inserire il codice di sblocco della serratura solo se necessario; altrimenti, inserire il codice dell'allarme. #}		
			"code_porta": "6666"

		}
	{% endmacro %}

- **personal.jinja**

	This file will be used for other projects within this GitHub repository. In the file, we set up personal data that will be used in all projects. Simply enter your entities respecting the JSON indentation.

	Let's see how to customize it. Although not all information is required for this package, it is advisable to fill in all fields to take full advantage of it in other projects.

	In this section, we define the entities and sensors for each person. If you do not want to associate a cell phone number or an alarm sensor with a specific person, simply assign the value "none" in the corresponding section of the file. To add or remove people from the dictionary list, you need to pay attention to the JSON syntax.

	```
	{% macro persons() %}
	[
		{
			"person": "person.marco",
			"battery": "sensor.cellulare_marco_battery_level",
			"notify": "mobile_app_cellulare_marco",
			"sveglia": "sensor.cellulare_marco_prossimo_allarme",
			"cellulare": "331000000"
		},
		{
			"person": "person.serena",
			"battery": "sensor.cellulare_serena_livello_della_batteria",
			"notify": "mobile_app_samsung_s21",
			"sveglia": "none",
			"cellulare": "335000000"
		}
	]
	{% endmacro%}
	```
	In this section, we will list our media players used for notifications. Be sure to correctly enter the selected media players for Alexa and TTS notifications (e.g., Google), carefully following the correct syntax.

	```
	{% macro media_players(type) %}
		{% set list_media = 
			[
				'media_player.camera',
				'media_player.studio',
				'media_player.googlehome_cameretta',
				'media_player.googlehome_bagno',
				'media_player.googlehome_cucina',
				'media_player.googlehome_salone'
			]
		%}
		{% for integrations in integration_entities(type) if integrations in list_media %}
			{{ integrations }}
		{% endfor %}
	{% endmacro %}
	```
## **Alarm control panel**:
The main files are located in the *alarm_control_panel* folder.
#### **General alarm**:
- The alarm_control_panel entity is created to manage the alarm
- The *input_boolean.alarm_triggered_state* is created:	
	- This input is used to determine whether the alarm has been triggered since it was entered. 
	- The state of this boolean is used in other files in the package.
- A boolean is created to manage the guest mode.
- Two groups are created: **exclude_alarm_entities and include_alarm_entities**, which allow to include or exclude entities that are not needed for the package or are not automatically detected. The entities are used in the following files:
	- BATTERY STATUS DEVICES file: "battery_status_alarm": 
		- Additional sensors can be added to monitor the battery status. 
		- It is not mandatory to use the default battery sensor, but you can choose any other device entity to monitor (e.g., lock.danalock).
		- It is possible **only** to include the sensors 
	- PHYSICAL BUTTONS "file: real_input_alarm."
		- It is possible **only** to exclude sensors 
		- Note: Entities using the event: shelly.click service (shelly used with 'Momentary' buttons) can neither be excluded nor included.
	- SENSORS FOR ALARM file:'armed_away & armed_night':
		- It is possible to include or exclude binary_sensors in both groups simultaneously.
		- Note: use only binary_sensors
	- ALARM SHUTDOWN: file: action_alarm:
		- If the alarm is triggered during the pending(lights on) state, the lights are turned off with the option to leave on only those included in the "exclude_alarm_entities" group, but only if it is **after sunset**.
	- DETECT_JAMMER: It is possible to exclude sensors that you do not want in the list while to include them you need to assign the correct device_class signal_strength to the sensors.
	- INSERT_ARMED/AWAY-NIGHT: 
		- It is possible **only** to exclude sensors. 
		- It is possible to exclude covers that you do not want to use in the package for example awnings.
#### **Armed night e Armed away**:
![armed_night](https://github.com/Home-Assistant-Pro-Team/Allarme/assets/62516592/f3d16a84-9211-49a9-b6f9-b9a20527a467)
- A select template is created to have the list of all sensors used for alarm operation, filtered by domain and device_class.
- When a sensor is selected from the select, it is automatically added to the list of alarm-enabled sensors; if already present, it is removed.
- When the alarm status changes to ARMED_NIGHT or ARMED_AWAY, sensors with device_class: windows that are in the "on" (open) state are excluded from the trigger (they are not removed from the list).
- When a window that had previously been excluded from operation is closed and 10 minutes elapses, it is automatically re-entered among the sensors for alarm control, but ONLY during the ARMED_NIGHT state.
- It is possible to exclude from the selected sensors only for ARMED_AWAY mode during a single alarm input, either before activating it or after. This exclusion remains in effect until the alarm is deactivated.
- Individual sensors can be excluded or included even with an alarm in progress from the user interface (UI).
- In notifications, the last sensor that actually changed state is always identified, indicating the time when this happened.
- You can decide which sensors to include or exclude from the list used in the "select" function in the "general_alarm" file by entering the entities in the exclude_alarm_entities and include_alarm_entities groups.

*NB Make sure you have set the **device_class: window** attribute correctly on the devices used for windows. If it is not set correctly, the state of the window will not be detected correctly and it will not be excluded from alarm operation if it is left open but will be treated as a generic alarm sensor.*

## **Skill device**
There are several files in the **skill_device** folder that can be used according to your personal needs. However, none of these files are essential for the alarm to work properly.

#### **Notifiche**:

![notify](https://github.com/Home-Assistant-Pro-Team/Allarme/assets/62516592/1cd2b3d2-65b1-4ea3-a445-fdd4cb4e091e)

To make it easier to choose notifications based on the devices used, we are introducing a centralized system. This system allows customizing the notifications activated on different types of devices, such as push, VoIP call, and media player. It is important to point out that the files related to this choice are not essential for general operation.
  - #### **Notifiche voip**: 
    We use the addon ["ha-sip" developed by Arne Gellhaus](https://github.com/arnonym/ha-plugins), which allows us to receive calls directly from Home Assistant. This addon must be installed and running in order for us to take advantage of its functionality. In the file "alarm.jinja" you can enter the phone number associated with each person. In case you do not want to set a phone number for a particular person, simply assign the value "none" to the corresponding "cell phone" field. By following these small directions, we will be able to receive notifications as described below.

    Barring certain exceptions, such as the keypad emergency code, we will receive a call that must be answered within 30 seconds. After this time has elapsed, the system will place a new call to the next person on the list whose phone number has been set. If we answer the call, we will hear a message describing the problem for which we are receiving the call, giving us the opportunity to select several options. In case we enter the alarm code, it will be disabled and no further calls will be made to the remaining people in the list. By entering the asterisk, we will prevent calls to the next people in the list. In case you enter an invalid code, you will receive a notice that the code you entered is invalid. If we do not make selections within 20 seconds, the call will be terminated and a call will be made to the next person in the list.

	To enable a call, simply remove the "#" character from the corresponding conditions in the file. Conversely, if you wish to disable a call, you can comment out the corresponding conditions by entering the "#" character.

	Once these operations are completed, the automations in the package will be ready to send calls to the involved devices.

    *Keep in mind that this file was written considering the use of a FritzBox for handling SIP calls. To create a voip extension, simply log into your router, following go to PHONE -> PHONE DEVICES -> CONFIGURE NEW DEVICES -> PHONE -> LAN/WIFI -> assign a user and password and customize the example given in [github](https://github.com/arnonym/ha-plugins). In case you use a different provider or device, you will need to change the SIP Registrar service parameter to adapt the file to your system's specifications.*
  - #### **Notifiche media_player**: 
    In the "alarm.jinja" file, it is necessary to include the media_players to which to send the notification. Supported devices include Alexa and TTS (e.g., Google). The default TTS service is "google_translate_say."

	To enable a notification, simply remove the "#" character from the corresponding conditions in the file. Conversely, if you want to disable a notification, you can comment out the corresponding conditions by inserting the "#" character.

	You can customize the default volume of notifications by changing the value in the automation variable. In case you want to change the volume of a single notification, you can add the "volume" parameter during event creation in the corresponding automation. It is important to note that some notifications may already have preset volumes based on the alarm status.
  - #### **Notifiche push**: 
    To send notifications, we use Home Assistant's native application, "App companion." To enable a notification, simply remove the "#" character from the corresponding conditions in the file. Conversely, if you want to disable a notification, you can comment out the corresponding conditions by inserting the "#" character.

	In the "alarm.jinja" file, you need to enter the mobile notification service assigned to the affected person.

	Once these steps are completed, the automations in the package will be ready to send notifications to the affected devices.
#### **Battery status alarm**: 
In this file, you can configure receiving a notification when the battery power of a device monitored by the alarm drops below 10%. You can also add devices that are not in the default list, such as smart locks, by adding the device entity to the "include_alarm_entities" group in the "general_alarm" file.
#### **Campanello**:
An ESP was installed and at the front door bell to create a small siren. When a sensor of the alarm is triggered, the doorbell emits a short sound as an alarm warning. When the alarm is actually triggered (triggered), the doorbell sounds intermittently for the duration of the triggered, with a frequency of 1 second sound and 1 second pause. It is possible to stop the intermittence of the doorbell by disabling the alarm.
#### **Sirena**:
When a sensor of the alarm is triggered, the siren emits a short sound as an alarm warning. When the alarm is actually activated (triggered), the siren sounds for the time of the triggered. It is possible to interrupt the siren by disabling the alarm.
#### **Controlcode**:
You can use the external keypad or the keypad that came with the card; you can also use both keypads at the same time.

There are several files in the folder:
 - #### **General code**:
    This file should always be loaded regardless of the type of keypad used. It provides basic functionality for access code management.
 - #### **With card**: 
	This file can only be used with the card provided in the package. When using this file, it is important to keep some important considerations in mind. For example, the alarm code and the emergency code can start with zero. The with_card file performs a check for attempts to enter the disarm code. In the event that the maximum number of incorrect code attempts is exceeded, the alarm changes to a triggered state and a notification is sent. An emergency code can also be set to disarm the alarm and send an alert notification (voip and push) to people who are NOT at home. During the sequence, an event is created to be used as a trigger in other automations for example sending snapshoot cameras. It is important to note that by using the alarm code, it makes it possible to change settings from the card even when the alarm is active and remove the lock. It is also possible to configure a service code that can be shared, for example, with cleaning staff, which can be turned on and off from the user interface .

 - #### **Keypad / tastierino esterno**: 
    The package includes a readme file and an example of use with 3x4 12 Key Matrixe and EspHome. The keypad.yaml file is essential for proper alarm operation. By entering the unlock code, both the global alarm and the night alarm can be turned on and off. The system checks on attempts to enter the wrong code and sends a notification if the allowed limit is exceeded. It is possible to define an Emergency Code, which allows you to disable the alarm and send a danger notification (voip and push) only to people who are NOT in the house.  During the sequence, an event is created that can be used as a trigger in other automations, such as sending camera snapshots in case of emergency. It is also possible to configure a service code that can be shared, for example, with cleaning staff, which can be turned on and off from the user interface .
#### **Detect jummer**:

![jummer](https://github.com/Home-Assistant-Pro-Team/Allarme/assets/62516592/6f512908-cda2-4b41-b5a4-5c01bcd2a0d3)

It is based on the detection of devices that switch to the offline state.

To simplify the process, a select template is created to obtain a filtered list of sensors required for proper operation. This selection is made based on the domain and class of the device.

Through the user interface (UI), devices can be placed within specific groups for better organization. In addition, the user interface (UI) provides the ability to customize the number of devices in the "jammer_detect_device" group that are considered to be in an offline state.

*To integrate Shelly devices, you must have official integration and have enabled the RSSI sensor, while devices that have EspHome must have the sensor with [platform: wifi_signal](https://esphome.io/components/sensor/wifi_signal.html)*
#### **Led allarme**:
I installed an LED next to my front door to keep an eye on the status of my alarm system. The operation of the LED is as follows:
- When the alarm system is off, the LED remains off. This clearly indicates that the system is inactive and no protection is active.
- When the alarm system is triggered, the LED lights up, signaling that the system is active and ready to detect intrusion.
- If the alarm is triggered, such as by a break-in, the LED starts flashing, providing a clear visual indication that the alarm has been activated. The flashing of the LED continues until the alarm is deactivated for 10 minutes. After the 10 minutes has elapsed, the LED returns to the off state.
#### **NFC**:
NFC is only used to activate the alarm in the 'away' mode and to deactivate the alarm.. If the alarm is in the 'away' and 'night' modes, the alarm is deactivated. To ensure greater security, a check is made on the device ID. In this way, only authorized devices can activate or deactivate the alarm.
#### **Smart lock**:
There are three files in the folder that describe how to use a smart lock, such as the Danalock model, in three different ways. These files provide detailed instructions on how to integrate the smart lock into the alarm system and use it effectively.
 - #### **Smart lock nfc**: 
    The smart lock can also be controlled via an NFC tag, allowing the door to be opened or closed without the need to use a traditional key
 - #### **Smart lock keypad**: 
    It is possible to manage the opening or closing of the lock using a specific code and to turn the alarm on or off using the alarm code.
#### **Person**

![person](https://github.com/Home-Assistant-Pro-Team/Allarme/assets/62516592/5cd97800-c19e-495a-8f3f-1afbdeb00a5b)

To use this file, it is necessary to customize the anchors.
- The system can turn the global alarm on and off based on the presence of people in the house, as long as the guest mode is not active. 
- If the alarm is not active and there is only one person in the house, the system checks the battery status of the cell phone, if the status is below 15%, the system will turn off the automatic activation of the alarm until there are more people in the house or the battery is back above 15%. This helps prevent accidental alarm activation when there is only one person in the house and the cell phone battery is low.
- A notification can be triggered that alerts when the cell phone battery drops below 15% and the global alarm is active. The notification specifies low cell phone and can be used to avoid coming home with the alarm on and not having your cell phone available to turn it off.
- It is possible to automatically disable the alarm set in night mode when a family member returns home. Basically, if the alarm is on and a family member enters the house, the system detects the presence of the family member and deactivates the alarm, notifying the person returning. The alarm automatically returns to night mode after 5 minutes or when the front door is closed.

*To ensure proper operation of person tracking, we strongly recommend that you read the article written by [Henrik Sozzi](https://henriksozzi.it/2021/05/posizione-delle-persone-con-home-assistant/). This article provides a comprehensive explanation of the operation and configuration of the tracking system. By reading it, you will be able to gain a thorough understanding of how to properly set up tracking and achieve the desired results.*
#### **Detached**

![detached](https://github.com/Home-Assistant-Pro-Team/Allarme/assets/62516592/e255a218-6273-4a33-a41e-501632a50ea2)

  - #### **Detached shelly**: 
    This file is used to activate the "detached" function on all Shelly devices that work with the official Home Assistant integration. The "detached" function allows you to disable the physical buttons on the devices, but continue to use the relays via Home Assistant. The file works like this: when we execute the "detached_on" script, it is first checked that the same script has not been executed twice in a row. Next, the current state of the buttons is saved in a sensor, after which all devices are set to "detached" mode. On the other hand, when we execute the "detached_off" script, it is checked that the same script has not been executed twice in a row. Then, all devices are reset to the settings in use before the "detached" function was activated.
  - #### **General detached alarm**: 
    The file consists of two automations. 
  	- The concept refers to an automation that triggers an alarm if the entered mode is global and any key in the house is pressed. The automation recognizes all Shelly devices with official integration and all Esphome devices with firmware in the folder loaded. It is possible to exclude devices from operation by adding the entity in the group.exclude_alarm_entities present in the general_alarm file. However, external buttons, switches, and diverters connected to Shelly devices are viewed differently from each other. For buttons set as momentary an event (click.shelly) is created, while for switches and diverters an entity is created in the sensor section called input, which by default is disabled. Therefore, you must manually enable the input sensor for each individual shelly device affected by going to Settings > Devices and Services > selecting the shelly device affected and enabling the input sensor.
  	- Detached alarm: When the alarm goes off, the function allows you to disable the physical keys throughout the house. In addition, when the alarm returns to "disarmed" mode but previously the alarm was triggered, the automation restores the key settings to the previous state. The "Detached alarm" function is compatible with Shelly devices that have official integration (detached_shelly.yaml) and with Esphome devices that use the sample firmware (firmware_esphome).
#### **UPS alarm**:

![ups](https://github.com/Home-Assistant-Pro-Team/Allarme/assets/62516592/6a458651-22df-43bb-aeb5-5fe0bcf3d1b4)

In the system, a Tecnoware1100 was used. Through this device, I receive a notification if a power failure occurs and a notification when power is restored.
#### **Router alarm**

![router](https://github.com/Home-Assistant-Pro-Team/Allarme/assets/62516592/3fc716b1-304d-460a-8096-a3422afad0ce)

If you are using a Fritz!Box 6890 router with LTE fallback, you will receive a notification when the Internet connection is dropped or restored. This device can automatically switch to the LTE connection if the main connection is interrupted and will send a notification to inform you of the connection status. In this way, you will be aware of Internet interruptions and resets.
#### **CCTV**:

![cctv](https://github.com/Home-Assistant-Pro-Team/Allarme/assets/62516592/44641639-bac9-444b-80f9-e501c44404f8)

Cameras should be entered in the anchor list. When the alarm is triggered, a 30-second recording is made and a notification is sent to all recipients entered in the notify list in alarm.jinja for each camera. The notification includes a screenshot of the camera and two action buttons: the first allows you to view the camera live, while the second allows you to access the folder to play the recording you just made.

*This file was written and tested for Android devices.*
## **Scene**
#### **Action alarm**:

![triggered](https://github.com/Home-Assistant-Pro-Team/Allarme/assets/62516592/3d0be2ac-565b-4b09-bf8e-b260d44d0d7c)

Household actions are programmed to be performed when an infraction is detected with the alarm on. Here is how the actions work according to the different phases:
- Infraction detected (Pending): 
	- All lights are turned on. This can help create a deterrent effect.
- Alarm Triggered:
	- All lights are turned off. This creates an effect of surprise and disorientation for possible intruders, limiting their visibility inside the house.
	- Roller shutters are closed.
	- The current state of the automations (group.automation_action_armed) that are to be deactivated via a scene is saved. This allows the settings of the automations to be restored once the alarm is removed.
	- Automations placed in the specified group (group.automation_action_armed) are deactivated. This means that automatic actions associated with these automations, such as turning on lights via motion sensors, will not be activated during the alarm.
- Alarm removed:
	- If the alarm is turned off during pending (when the lights are on):
		- All lights except those included in the group specified (group.exclude_alarm_entities) in the "general_alarm" file are turned off, but only if it is after sunset. This allows some lights to be kept on for safety or comfort reasons, such as outdoor lights or lights in certain areas of the house.
		- If it is before sunset, all lights are turned off.
- Alarm removed after it is triggered:
	- Automations are reactivated using the previously created scene. This means that the automations resume their normal activities as before the alarm, following the automated routines within the house.

*Keep in mind that this file was written for personal use and may not reflect everyone's habits. However, it was included in the project in order to provide insights and possibilities for customization.*
#### **Insert armed away**:

![insert_armed_away](https://github.com/Home-Assistant-Pro-Team/Allarme/assets/62516592/89fa1411-465e-47a4-8034-4261180cac68)

When the global alarm is activated in the "armed away" mode, several actions are performed to ensure the safety and energy efficiency of the home. Here is how these actions work:
- Turning off TVs and Android TV
- All lights are turned off to minimize energy consumption
- Turning off air conditioning systems (climate), except refrigerator climates if they are integrated with smartthinq_sensors. 
- Turning off fans (fans).

*Keep in mind that this file was written for personal use and may not reflect everyone's habits. However, it was included in the project in order to provide insights and possibilities for customization.*
#### **Insert armed night**:

![auto_night](https://github.com/Home-Assistant-Pro-Team/Allarme/assets/62516592/03fa2a36-f4b0-41b3-ab29-34239b3b538a)

The "Insert armed night" file contains automations to manage the activation and deactivation of the night alarm. Here is how these automations work:
- Automatic insert armed night alarm: 

	The alarm is activated under certain conditions: 
	
	- If all beds are occupied using the [bedpresence](https://github.com/Home-Assistant-Pro-Team/Bed-Presence/blob/main/READMI_EN.md) occupancy sensors connected to the Aqara flood sensors. 
	- The alarm is triggered only if the last room detected is the bedroom or nursery.
	- Both of these conditions must be met for at least 5 consecutive minutes.
	- The night alarm is activated only during the time interval between 21:00 and 03:00, unless the guest mode is active.

	When the night alarm is activated and the guest mode is not active, the following actions are performed:
	- All lights in the house are turned off, except those in the bedroom and nursery (configuration of lights in the correct areas is required).
	- Televisions that are not in the rooms are turned off.
However, notifications playing on Alexa or other Google devices are not interrupted to avoid unwanted interruptions.
	- All climate devices and fans present are turned off.
	- Shutters are closed with a different percentage based on the value of boolean.winter_summer. If winter_summer is set to off, the shutters are closed completely. On the other hand, if it is set to on, the shutters are closed at a custom percentage.
- Disable night alarm: 

	Deactivation of the night alarm is handled at two separate times. Deactivation occurs only if the guest mode is not on and if the time is between 5:00 and 11:00.

	In the first moment, the alarm is deactivated at the sound of the first available alarm set (alarm.jinja). At the same time, all shutters except those in the rooms are opened.

	At the second time, the alarm is turned off and the shutters are opened when all beds are vacant for at least 5 consecutive minutes. 

	In this way, the night alarm is deactivated at two separate times, ensuring appropriate management both in the event of a wake-up call and when the beds are free for a specified period of time. At the same time, automation is configured to prevent the alarm from going off on non-working days.

*Keep in mind that this file was written for personal use and may not reflect everyone's habits. However, it was included in the project in order to provide insights and possibilities for customization.*

#### **External button alarm**:

In the following scenario, we wish to leverage a button (e.g., an Aqara button) to manage the alarm. The goal is to allow the night alarm to be quickly deactivated when we are at home and reactivated once we leave. This can be useful for those who work at night or early in the morning, but still want to ensure the safety of the family while they sleep.

The automation associated with the button and with the active night alarm works as follows: when the button is pressed, the alarm is turned off immediately. Thereafter, the automation waits for the door to be closed and then reactivates the alarm. If more than one person was in the house at the time the alarm was deactivated and the [bedpresence](https://github.com/Home-Assistant-Pro-Team/Bed-Presence/blob/main/READMI_EN.md) sensors still detect someone in bed, the night alarm is reactivated; otherwise, the global alarm is reactivated. In case a minute passes without the door being closed, the night alarm is reactivated, while a notification is sent to warn that the door remained open.

Automation is executed if the guest mode is not active.

*In the file, you need to customize the automation trigger with your device and according to your integration. Keep in mind that this file was written for personal use and may not reflect everyone's habits. However, it was included in the project in order to provide insights and possibilities for customization.*
## **Card**
In the video, it is shown that a panel with a virtual keypad has been created to access alarm settings and display alarm status. In order to use this card, the following components are required:

- [Browser mode](https://github.com/thomasloven/hass-browser_mod) 
- [Button card](https://github.com/custom-cards/button-card)
- [Layout card](https://github.com/thomasloven/lovelace-layout-card)
- File "whit_card.yaml": This file contains the configuration of the virtual keyboard card. 
- File "jammer_detect.yaml": Be sure to properly include this file in your Home Assistant configuration to ensure proper card operation

It is also recommended to have a dashboard in YAML mode as described well by [MaxAlbani](https://www.maxalbani.it/2023/04/home-assistant-dashboards-lo-strumento-per-creare-interfacce-grafiche/#modalita) in his article.

When you use the virtual keyboard card, the settings popup will show only the file entities you have decided to use.

You can use the card in storage mode without activating the "button_card_templates" folder (although not recommended). To do this, select the desired dashboard, open the options menu and choose "Edit." Next, access the textual configuration editor and add "button_card_templates:" to the beginning of the dashboard, making sure to respect the correct indentation. Finally, copy the template file to the appropriate location. Remember to pay attention to formatting and indentation during the copying process. Keep in mind that this mode may not be recommended, but you can use it if necessary.
```
button_card_templates:
  no_background:
  styles:
  	  card:
	    - background-color: rgba(0,0,0,0.0)
	    - border-width: 0px
	  display:
	  tap_action:
		action: none
...........
title: xxxxx
views:
  - theme: Backend-selected
    title: test_alarm
    path: a
    badges: []
    cards:
...........
```
### **Contributi**
This project is open for contributions. If you would like to provide feedback, report a bug, or request a new feature, please create an issue on the repository.
## Change Log

### Versione: 1.2
- PERSONAL FILE: Personal data has been separated into a new file in the 'custom_templates' folder for notifications, allowing unique use across all GitHub projects.
- The 'pending' trigger for sending CCTV notifications has been added.
- Individual templates have been replaced with macros to reduce the amount of code.
- It is possible to exclude certain sensors only for ARMED_AWAY mode during a single alarm input, either before triggering or after. This exclusion remains in effect until the alarm is deactivated.


### **Support Us**
If you enjoyed this project, we'd love to have your support. Even a simple coffee can make a difference.
The funds raised will be used to purchase new material and carry out new projects. You can contribute by clicking the button below.
Heartfelt thanks for your support!

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/M4M1MI00I)
