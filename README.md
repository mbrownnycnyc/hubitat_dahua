# hubitat_dahua

This package provides a Hubitat integration for Dahua-supplied IP cameras, including Dahua and Amcrest cameras and doorbells.  It may work with other brands that are Dahua cameras under the hood.

Supported capabilities include ImageCapture, MotionSensor, PushableButton (for doorbells), cross line detection, cross region detection, and object detection (if supported by your camera).

All events are based on real-time events pushed from the camera.  There is no polling for events or status.

Thanks to techbill from the Hubitat forums for all of their testing, feedback, and suggestions.  And thanks to mluck for the inspiration.

# Manual Installation instructions:

Installing with Hubitat Package Manager (HPM) is recommended. 

If you must install manually, follow these steps:

* In the *Drivers Code* section of Hubitat, install the *cameraDriver*.    
* (Optional) If you want to use auto-discovery features with multiple cameras, install the *cameraDiscoveryApp* in the *Apps Code* section of Hubitat.  Skip to the section below on using the camera discovery app.

# Virtual Device configuration instructions:

* In the *Devices* section of Hubitat, add a *New Virtual Device* of type Dahua/Amcrest Camera.
* On the virtual device page, set the preferences:
    * Enter the IP and port number of your camera.
    * Enter the login credentials of your camera.  This should be the same username and password that you use to log in to the web interface of the camera by entering its IP.
    * Save Preferences and then refresh the page.  The name of the virtual device should be updated to match the name of your camera.
* **Note that some recent firmware versions have removed or disabled local API access.**
    * If your camera does not respond correctly, try another firmware version and/or camera.

# Usage instructions (virtual device):

* Use the `take` command to take a snapshot from the device.
    * This will store an image locally in the File Manager of your Hubitat hub (requires Hubitat software 2.3.4.132 or later).
    * To display or use the stored image, use my companion app *hubitat_imageServer* (available through HPM and GitHub).
* Observe events on the `motion` attribute (for video motion) and `smartDetectType` attribute (for AI object detection).
* Observe events on the attribute `crossLineDetected`and `crossRegionDetected` attributes, which will report the names of those events.
* For doorbell devices, the first physical button press on the doorbell will create a child virtual button controller.
    * Observe `pushed` events on button 1 for doorbell button press events.
* The `closeStream` command will close the event stream from the device.
    * You can use this if you want to pause receiving real-time events for some reason.
    * `Initialize` will re-start the event stream.

# Rule Machine Integration

The camera driver provides extensive event attributes that can be used with Hubitat's Rule Machine to create powerful automation based on camera events. Here are some practical examples:

## Smart Motion Detection Events

### Human Detection Automation
Create a rule that responds when a human is detected:

In Hubitat UI:

1. Go to **Apps > Rule Machine > Create New Rule**
2. **Select Trigger Events > Choose device** > Select your camera device
3. **Pick capability** > Select **"Custom attribute"** > **Select attribute** > **smartMotionHuman**
4. **Event type** > Select **"changed"**
5. **Select Actions > Add logic**:
   * Add condition: `%value% == "detected"`
   * Add action: Send notification "Human detected by camera with %device.smartMotionHumanConfidence% confidence"
   * Add action: Turn on lights in the area
6. **Save**

### Vehicle Detection Automation
Create a rule that responds to vehicle detection:

1. Go to **Apps > Rule Machine > Create New Rule**
2. **Select Trigger Events > Choose device** > Select your camera device
3. **Pick capability** > Select **"Custom attribute"** > **Select attribute** > **smartMotionVehicle**
4. **Event type** > Select **"changed"**
5. **Select Actions > Add logic**:
   * Add condition: `%value% == "detected"`
   * Add action: Send notification "Vehicle detected by camera"
   * Add action: Log vehicle detection to security system
6. **Save**

### Cross Line Detection for Security
Create a rule for perimeter monitoring:

1. Go to **Apps > Rule Machine > Create New Rule**
2. **Select Trigger Events > Choose device** > Select your camera device
3. **Pick capability** > Select **"Custom attribute"** > **Select attribute** > **crossLineDetected**
4. **Event type** > Select **"changed"**
5. **Select Actions > Add logic**:
   * Add condition: `%value% != "waiting"`
   * Add action: Send notification "Line crossing detected: %device.crossLineDetected%"
   * Add action: Trigger security alarm
6. **Save**

### Audio Anomaly Detection
Create a rule for unusual audio events:

1. Go to **Apps > Rule Machine > Create New Rule**
2. **Select Trigger Events > Choose device** > Select your camera device
3. **Pick capability** > Select **"Custom attribute"** > **Select attribute** > **audioAnomaly**
4. **Event type** > Select **"changed"**
5. **Select Actions > Add logic**:
   * Add condition: `%value% == "anomaly"`
   * Add action: Send notification "Audio anomaly detected: %device.audioAnomalyType%"
   * Add action: Start recording on all nearby cameras
6. **Save**

### Crowd Detection for Capacity Management
Create a rule for monitoring crowd density:

1. Go to **Apps > Rule Machine > Create New Rule**
2. **Select Trigger Events > Choose device** > Select your camera device
3. **Pick capability** > Select **"Custom attribute"** > **Select attribute** > **crowdDetection**
4. **Event type** > Select **"changed"**
5. **Select Actions > Add logic**:
   * Add condition: `%value% == "detected"`
   * Add action: Send notification "Crowd detected with density: %device.crowdDetectionDensity%"
   * Add action: Adjust ventilation system based on crowd level
6. **Save**

## Available Event Attributes

The driver provides the following event attributes that can be used in Rule Machine:

### Smart Motion Detection
- `smartMotionHuman` - Human detection status (waiting/detected)
- `smartMotionHumanConfidence` - Human detection confidence percentage
- `smartMotionVehicle` - Vehicle detection status (waiting/detected)
- `smartMotionVehicleConfidence` - Vehicle detection confidence percentage

### Video Status
- `videoLoss` - Video signal status (recovered/lost)
- `videoBlind` - Video blind status (normal/blinded)
- `videoAbnormalDetection` - Video abnormality status (normal/abnormal)
- `videoAbnormalType` - Type of video abnormality detected

### AI Detection Events
- `crossLineDetected` - Line crossing event (waiting/line name)
- `crossRegionDetected` - Region intrusion event (waiting/region name)
- `leftDetection` - Object left behind status (cleared/detected)
- `takenAwayDetection` - Object taken away status (cleared/detected)
- `wanderDetection` - Wander detection status (cleared/detected)
- `rioterDetection` - Rioter detection status (cleared/detected)
- `parkingDetection` - Parking violation status (cleared/detected)
- `moveDetection` - Movement detection status (cleared/detected)
- `crowdDetection` - Crowd detection status (cleared/detected)

### Audio Events
- `audioMutation` - Audio mutation status (cleared/detected)
- `audioAnomaly` - Audio anomaly status (normal/anomaly)

### System Events
- `ntpAdjustTime` - NTP time synchronization event
- `timeChange` - System time change event
- `rtspSession` - RTSP session status

## Tips for Rule Machine Integration

1. **Use conditional logic**: Always add conditions to check the `%value%` to ensure you respond to the correct event state
2. **Leverage secondary attributes**: Many events provide additional information (confidence, type, region) that can be used in notifications
3. **Combine multiple triggers**: Create complex rules that respond to multiple types of events from the same camera
4. **Test with debug logging**: Enable camera debug logging to see all events in the logs while testing your rules
5. **Consider event frequency**: High-frequency events like motion detection may need debouncing or rate limiting in your rules

# (Optional) Use the camera discovery app:

**Install the app**
* In the *Apps* section of Hubitat, click *Add User App* to actually install the *Dahua camera discovery* app.
* Enter the IP address, port, username, and password of your NVR or of a camera in your system.
* On the next page, you should see a list of all discovered cameras with an option to automatically create a virtual device for each.
    * The discovery process may intermittently miss certain cameras on your system.  Refresh the page to re-scan if necessary.
    * You will still need to visit each virtual device page to configure the specific camera settings.  Review the section above for virtual device configuration instructions.

# Disclaimer

I have no affiliation with any of the companies mentioned in this readme or in the code.
