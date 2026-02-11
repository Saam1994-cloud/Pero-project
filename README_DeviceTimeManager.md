# DeviceTimeManager_FB

## Overview

The **DeviceTimeManager_FB** is a professional PLC function block designed to intelligently manage and rotate up to 10 devices based on their accumulated work time. This function block ensures optimal device utilization by automatically selecting devices with the least runtime, while respecting device availability and operational requirements.

## Key Features

- **Smart Device Selection**: Automatically selects devices with the least work time
- **Priority-Based Selection**: Ready signal → Lowest work time → Lowest device number
- **Flexible Operation**: Manage 0-10 devices simultaneously
- **Automatic Rotation**: Toggle signal for scheduled device rotation
- **Comprehensive Monitoring**: Status outputs and warning flags
- **Professional Structure**: Single UDT for all data (Config, Devices, Status, Internal)
- **Work Time Tracking**: Tracks years, months, weeks, days, hours, minutes, seconds
- **Insufficient Device Warning**: Alerts when not enough ready devices are available

## Files

- **`DeviceTimeManager_FB.scl`** - Main function block
- **`UDT_DeviceTimeManager.scl`** - Unified data type definition
- **`DeviceTimeManager_FB_Examples.scl`** - Comprehensive usage examples
- **`README_DeviceTimeManager.md`** - This documentation file

## Architecture

### Data Structure (UDT_DeviceTimeManager)

The function block uses a single UDT that contains all necessary data:

```
UDT_DeviceTimeManager
├── Config (Inputs)
│   ├── Enable
│   ├── NumberOfDevicesToRun (0-10)
│   ├── ToggleSignal
│   └── ResetWorkTimes
├── Devices[1..10]
│   ├── Ready (Input)
│   ├── WorkTime (Input)
│   │   ├── Years, Months, Weeks, Days
│   │   ├── Hours, Minutes, Seconds
│   │   └── TotalSeconds (Calculated)
│   ├── StartCommand (Output)
│   ├── IsRunning (Output)
│   ├── Priority (Internal)
│   └── Selected (Internal)
├── Status (Outputs)
│   ├── IsEnabled, IsInitialized
│   ├── ActiveDeviceCount
│   ├── ReadyDeviceCount
│   ├── InsufficientDevices (Warning)
│   ├── RequestedDeviceCount
│   ├── MissingDeviceCount
│   ├── ToggleState
│   ├── ToggleChangeDetected
│   └── LastError
└── Internal (Internal calculations)
```

## How It Works

### 1. Device Selection Algorithm

The function block uses a three-tier priority system:

1. **First Priority: Ready Signal**
   - Only devices with `Ready = TRUE` can be selected
   - Non-ready devices are never started

2. **Second Priority: Work Time**
   - Devices with less accumulated work time are preferred
   - Work time is calculated in total seconds for accurate comparison

3. **Third Priority: Device Number**
   - When work times are equal, lower device numbers are selected first
   - Device 1 has highest priority, Device 10 has lowest priority

### 2. Toggle Functionality

When the `ToggleSignal` changes state:

1. All currently running devices are stopped
2. New device selection is performed
3. Devices that were previously running are avoided (if possible)
4. If insufficient devices, previously running devices may be reselected

### 3. Work Time Calculation

The work time structure allows flexible time tracking:

```
TotalSeconds = Seconds +
               Minutes × 60 +
               Hours × 3,600 +
               Days × 86,400 +
               Weeks × 604,800 +
               Months × 2,592,000 +
               Years × 31,536,000
```

### 4. Insufficient Device Handling

If `NumberOfDevicesToRun > ReadyDeviceCount`:
- Starts all available ready devices
- Sets `InsufficientDevices` warning flag
- Reports `MissingDeviceCount` = (Requested - Available)

## Usage

### Basic Setup

```pascal
DATA_BLOCK "DB_DeviceManager"
   VAR
      // Device work times
      Device1_WorkTime : "UDT_DeviceTimeManager".DeviceWorkTime;
      Device2_WorkTime : "UDT_DeviceTimeManager".DeviceWorkTime;
      // ... up to Device10

      // Device ready signals
      Device1_Ready : Bool;
      Device2_Ready : Bool;
      // ... up to Device10

      // Function block instance
      Manager : "DeviceTimeManager_FB";

      // Timer for rotation
      Timer : "General_FB";
   END_VAR
BEGIN
END_DATA_BLOCK
```

### Function Block Call

```pascal
"DB_DeviceManager".Manager(
   Enable := TRUE,
   NumberOfDevicesToRun := 3,                              // Run 3 devices
   ToggleSignal := "DB_DeviceManager".Timer.Data.Pulses.Days.Pulse_1d,  // Daily rotation
   ResetWorkTimes := FALSE,

   // Device 1
   Device1_Ready := "DB_DeviceManager".Device1_Ready,
   Device1_WorkTime := "DB_DeviceManager".Device1_WorkTime,

   // Device 2
   Device2_Ready := "DB_DeviceManager".Device2_Ready,
   Device2_WorkTime := "DB_DeviceManager".Device2_WorkTime,

   // ... Continue for all devices

   // Unused devices
   Device3_Ready := FALSE,
   // ... etc
);
```

### Accessing Outputs

```pascal
// Start commands (to field devices)
Pump1_Start := "DB_DeviceManager".Manager.Data.Devices[1].StartCommand;
Pump2_Start := "DB_DeviceManager".Manager.Data.Devices[2].StartCommand;

// Status monitoring
ActiveCount := "DB_DeviceManager".Manager.Data.Status.ActiveDeviceCount;
Warning := "DB_DeviceManager".Manager.Data.Status.InsufficientDevices;
ReadyCount := "DB_DeviceManager".Manager.Data.Status.ReadyDeviceCount;
```

## Application Examples

### Example 1: Pump Station (3 Pumps, Run 2)

- **Scenario**: Water pumping station with 3 pumps
- **Requirement**: Always run 2 pumps, rotate daily
- **Implementation**: `NumberOfDevicesToRun = 2`, daily toggle signal

**Behavior**:
- Day 1: Pumps with 2 least hours run
- Day 2: Different 2 pumps selected (rotation)
- If 1 pump fails: Runs only 1 pump + warning flag

### Example 2: Compressor Bank (10 Compressors, Variable Load)

- **Scenario**: Industrial compressed air system
- **Requirement**: Run 4-8 compressors based on demand
- **Implementation**: `NumberOfDevicesToRun` varies dynamically

**Behavior**:
- Low demand: Run 4 compressors with least hours
- High demand: Run 8 compressors with least hours
- Weekly rotation: Switch to different compressors

### Example 3: HVAC Chillers (5 Chillers, Run 3)

- **Scenario**: Building cooling system
- **Requirement**: 3 chillers running, rotate weekly
- **Implementation**: `NumberOfDevicesToRun = 3`, weekly toggle

**Behavior**:
- Selects 3 chillers with lowest runtime
- Weekly rotation ensures even wear
- If chiller maintenance: Runs 2 chillers + warning

## Inputs/Outputs

### Inputs (VAR_INPUT)

| Name | Type | Range | Description |
|------|------|-------|-------------|
| Enable | Bool | - | Enable/disable function block |
| NumberOfDevicesToRun | Int | 0-10 | Number of devices to run simultaneously |
| ToggleSignal | Bool | - | Signal to trigger device rotation |
| ResetWorkTimes | Bool | - | Reset all work times to zero |

### Device Inputs (VAR_IN_OUT)

For each device (1-10):

| Name | Type | Description |
|------|------|-------------|
| DeviceX_Ready | Bool | Device ready signal (from field) |
| DeviceX_WorkTime | Struct | Accumulated work time structure |

### Outputs

| Output | Type | Description |
|--------|------|-------------|
| Data.Devices[X].StartCommand | Bool | Start command for device X |
| Data.Devices[X].IsRunning | Bool | Device X is currently running |
| Data.Status.ActiveDeviceCount | Int | Number of devices running |
| Data.Status.ReadyDeviceCount | Int | Number of ready devices |
| Data.Status.InsufficientDevices | Bool | Warning: not enough devices |
| Data.Status.MissingDeviceCount | Int | Number of devices short |
| Data.Status.ToggleChangeDetected | Bool | Toggle signal changed this cycle |

## Error Codes

| Code | Description | Action |
|------|-------------|--------|
| 0 | No error | Normal operation |
| 1 | NumberOfDevicesToRun < 0 | Value clamped to 0 |
| 2 | NumberOfDevicesToRun > 10 | Value clamped to 10 |

## Integration with General_FB

The DeviceTimeManager_FB is designed to work seamlessly with the General_FB pulse generator:

```pascal
// Daily rotation
ToggleSignal := GeneralTimer.Data.Pulses.Days.Pulse_1d

// Weekly rotation
ToggleSignal := GeneralTimer.Data.Pulses.Days.Pulse_7d

// Hourly rotation (testing)
ToggleSignal := GeneralTimer.Data.Pulses.Hours.Pulse_1h
```

## Best Practices

### 1. Work Time Tracking

```pascal
// Update work time when device is running
IF Manager.Data.Devices[1].IsRunning THEN
   // Increment work time
   Device1_AccumulatedTime := Device1_AccumulatedTime + T#100MS;

   // Convert to work time structure
   IF Device1_AccumulatedTime >= T#1S THEN
      Device1_WorkTime.Seconds := Device1_WorkTime.Seconds + 1;
      Device1_AccumulatedTime := Device1_AccumulatedTime - T#1S;

      // Handle minute rollover
      IF Device1_WorkTime.Seconds >= 60 THEN
         Device1_WorkTime.Seconds := 0;
         Device1_WorkTime.Minutes := Device1_WorkTime.Minutes + 1;
      END_IF;
   END_IF;
END_IF;
```

### 2. Ready Signal Logic

```pascal
// Comprehensive ready signal
Device1_Ready := Device1_Running AND      // Device operational
                 NOT Device1_Fault AND     // No faults
                 NOT Device1_Maintenance AND  // Not in maintenance
                 Device1_Available;        // Available for use
```

### 3. Alarm Generation

```pascal
// Generate alarm if insufficient devices
IF Manager.Data.Status.InsufficientDevices THEN
   GenerateAlarm(
      Message := 'Insufficient devices available',
      Severity := WARNING,
      Details := CONCAT('Missing: ', INT_TO_STRING(Manager.Data.Status.MissingDeviceCount))
   );
END_IF;
```

### 4. HMI Display

```pascal
// Status display on HMI
HMI_ActiveDevices := Manager.Data.Status.ActiveDeviceCount;
HMI_RequestedDevices := Manager.Data.Status.RequestedDeviceCount;
HMI_ReadyDevices := Manager.Data.Status.ReadyDeviceCount;
HMI_Warning := Manager.Data.Status.InsufficientDevices;

// Device status indicators (1-10)
FOR i := 1 TO 10 DO
   HMI_DeviceRunning[i] := Manager.Data.Devices[i].IsRunning;
   HMI_DeviceReady[i] := Manager.Data.Devices[i].Ready;
   HMI_DeviceWorkHours[i] := Manager.Data.Devices[i].WorkTime.Hours +
                              Manager.Data.Devices[i].WorkTime.Days * 24;
END_FOR;
```

### 5. Data Persistence

```pascal
// Store work times in RETAIN memory
DATA_BLOCK "DB_PersistentData"
{ S7_Optimized_Access := 'TRUE' }
VERSION : 1.0
RETAIN
   VAR
      Device_WorkTimes : Array[1..10] of "UDT_DeviceTimeManager".DeviceWorkTime;
   END_VAR
BEGIN
END_DATA_BLOCK
```

## Testing Checklist

- [ ] Test with 0 devices to run
- [ ] Test with 10 devices to run
- [ ] Test with equal work times (verify device number priority)
- [ ] Test with all devices ready
- [ ] Test with some devices not ready
- [ ] Test with insufficient ready devices
- [ ] Test toggle functionality
- [ ] Test with NumberOfDevicesToRun > ReadyDeviceCount
- [ ] Verify work time calculations
- [ ] Verify warning flags
- [ ] Test dynamic change of NumberOfDevicesToRun
- [ ] Test ResetWorkTimes functionality

## Common Scenarios

### Scenario 1: Equal Work Times

**Setup**: 3 devices, all with 1000 hours, all ready, run 2

**Result**: Devices 1 and 2 selected (lowest device numbers)

### Scenario 2: Mixed Work Times

**Setup**:
- Device 1: 1000h, Ready
- Device 2: 800h, Ready
- Device 3: 1200h, Ready
- Run 2 devices

**Result**: Devices 2 and 1 selected (lowest work times)

### Scenario 3: Device Failure

**Setup**: Running devices 1 and 2, Device 1 fails (Ready = FALSE)

**Result**:
- Device 1 stops
- Next lowest work time device starts
- Warning flag if no replacement available

### Scenario 4: Toggle Rotation

**Setup**: Devices 1,2,3 running, toggle signal changes

**Result**:
- Devices 1,2,3 stop
- Next 3 devices with lowest work time start
- Avoids 1,2,3 if possible

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2024 | Initial release |

## Technical Specifications

- **Platform**: Siemens S7-1200/1500
- **Language**: SCL (Structured Control Language)
- **Optimization**: S7_Optimized_Access enabled
- **Max Devices**: 10
- **Work Time Resolution**: Seconds
- **Max Work Time**: ~136 years (ULInt seconds)

## Support and Documentation

For more examples, see `DeviceTimeManager_FB_Examples.scl`

## License

Part of the Pero-project PLC function block library.

---

**Author**: Claude Code
**Project**: Pero-project
**Category**: Industrial Automation / Device Management
