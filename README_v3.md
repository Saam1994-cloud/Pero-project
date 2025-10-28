# General_FB v3.0 - Simplified Single-UDT Implementation

## Overview

**General_FB v3.0** is a professional time pulse generator for Siemens S7-1200/1500 PLCs with a **single, unified UDT structure**. This version combines all the professional features of v2.0 with the simplicity of having only ONE UDT file to import.

### What's New in v3.0?

- ✅ **Single UDT File**: Only 1 UDT to import (instead of 4!)
- ✅ **Simpler Structure**: All data accessible through `TimeManager.Data`
- ✅ **LTime Support**: Extended range for month/year pulses (fixed TIME overflow)
- ✅ **Professional Organization**: All features maintained from v2.0
- ✅ **Easy Import**: Just 2 files to import into TIA Portal

---

## Quick Start

### Step 1: Import Files to TIA Portal

**Only 2 files needed:**

1. **UDT_General_FB.scl** - Import to "PLC data types"
2. **General_FB_v3.scl** - Import to "Program blocks"

**How to import:**
- Open TIA Portal
- Right-click "PLC data types" → "External source" → Select `UDT_General_FB.scl`
- Right-click "Program blocks" → "External source" → Select `General_FB_v3.scl`
- Compile both

### Step 2: Use in Your Code

```scl
VAR
    TimeManager : General_FB;
    LED : Bool;
END_VAR

// Call the function block
TimeManager(
    Enable := TRUE,
    CustomTime := LTIME#1S500MS,    // 1.5 seconds
    ResetCounters := FALSE
);

// Use any pulse
LED := TimeManager.Data.Pulses.Seconds.Pulse_1s;
```

**That's it!** No complex configuration needed.

---

## File Structure

```
FunctionBlocks/
├── UDT_General_FB.scl              ← Only 1 UDT file!
├── General_FB_v3.scl               ← Main function block
└── General_FB_v3_Examples.scl      ← Usage examples
```

---

## UDT Structure

The `UDT_General_FB` contains everything in one organized structure:

```
UDT_General_FB
├── Config              (Input configuration)
│   ├── Enable
│   ├── CustomTime
│   └── ResetCounters
├── Pulses              (All pulse outputs)
│   ├── Microseconds
│   ├── Milliseconds
│   ├── Seconds
│   ├── Minutes
│   ├── Hours
│   ├── Days
│   ├── Months
│   ├── Years
│   └── Custom
├── Status              (Status information)
│   ├── IsEnabled
│   ├── IsInitialized
│   ├── CustomTimeActive
│   └── CycleTime_us
├── Internal            (Internal working data)
└── TimeConst           (Time constants)
```

---

## Function Block Interface

### Inputs

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `Enable` | Bool | TRUE | Enable/disable the function block |
| `CustomTime` | LTime | - | Custom time period (e.g., LTIME#1S500MS) |
| `ResetCounters` | Bool | FALSE | Reset all internal counters |

### Outputs

| Name | Type | Description |
|------|------|-------------|
| `Data` | UDT_General_FB | Complete data structure with all pulses and status |

---

## Available Pulses

All pulses are accessed through `TimeManager.Data.Pulses.*`:

### Microsecond Pulses
```scl
TimeManager.Data.Pulses.Microseconds.Pulse_1us      // 1 μs
TimeManager.Data.Pulses.Microseconds.Pulse_10us     // 10 μs
TimeManager.Data.Pulses.Microseconds.Pulse_100us    // 100 μs
```

### Millisecond Pulses
```scl
TimeManager.Data.Pulses.Milliseconds.Pulse_1ms      // 1 ms
TimeManager.Data.Pulses.Milliseconds.Pulse_10ms     // 10 ms
TimeManager.Data.Pulses.Milliseconds.Pulse_100ms    // 100 ms
```

### Second Pulses
```scl
TimeManager.Data.Pulses.Seconds.Pulse_1s            // 1 second
TimeManager.Data.Pulses.Seconds.Pulse_10s           // 10 seconds
```

### Minute Pulses
```scl
TimeManager.Data.Pulses.Minutes.Pulse_1min          // 1 minute
TimeManager.Data.Pulses.Minutes.Pulse_10min         // 10 minutes
```

### Hour Pulses
```scl
TimeManager.Data.Pulses.Hours.Pulse_1h              // 1 hour
```

### Day Pulses
```scl
TimeManager.Data.Pulses.Days.Pulse_1d               // 1 day
TimeManager.Data.Pulses.Days.Pulse_7d               // 7 days (week)
```

### Month Pulses (✅ Now Works - LTime!)
```scl
TimeManager.Data.Pulses.Months.Pulse_30d            // 30 days
```

### Year Pulses (✅ Now Works - LTime!)
```scl
TimeManager.Data.Pulses.Years.Pulse_365d            // 365 days
```

### Custom Pulse
```scl
TimeManager.Data.Pulses.Custom.Pulse                // Custom period
TimeManager.Data.Pulses.Custom.Active               // Is custom configured?
```

---

## Usage Examples

### Example 1: Simple LED Blink

```scl
VAR
    TimeManager : General_FB;
    LED : Bool;
END_VAR

TimeManager(Enable := TRUE);
LED := TimeManager.Data.Pulses.Seconds.Pulse_1s;
```

### Example 2: Multiple Blink Rates

```scl
VAR
    TimeManager : General_FB;
    LED_Fast : Bool;
    LED_Slow : Bool;
END_VAR

TimeManager(Enable := TRUE);

LED_Fast := TimeManager.Data.Pulses.Milliseconds.Pulse_100ms;  // 100ms
LED_Slow := TimeManager.Data.Pulses.Seconds.Pulse_10s;         // 10s
```

### Example 3: Custom Time Period

```scl
VAR
    TimeManager : General_FB;
    CustomLED : Bool;
END_VAR

TimeManager(
    Enable := TRUE,
    CustomTime := LTIME#1S500MS  // 1.5 seconds
);

CustomLED := TimeManager.Data.Pulses.Custom.Pulse;
```

### Example 4: Event Counter

```scl
VAR
    TimeManager : General_FB;
    Edge_1s : R_TRIG;
    Counter : Int;
END_VAR

TimeManager(Enable := TRUE);

Edge_1s(CLK := TimeManager.Data.Pulses.Seconds.Pulse_1s);
IF Edge_1s.Q THEN
    Counter := Counter + 1;
END_IF;
```

### Example 5: Data Logger

```scl
VAR
    TimeManager : General_FB;
    Edge_1min : R_TRIG;
    DataToLog : Real;
END_VAR

TimeManager(Enable := TRUE);

// Log data every minute
Edge_1min(CLK := TimeManager.Data.Pulses.Minutes.Pulse_1min);
IF Edge_1min.Q THEN
    // Log your data here
    LogData(DataToLog);
END_IF;
```

### Example 6: Alarm System

```scl
VAR
    TimeManager : General_FB;
    CriticalAlarm : Bool;
    AlarmBlink : Bool;
END_VAR

TimeManager(Enable := TRUE);

// Fast blink for critical alarms
IF CriticalAlarm THEN
    AlarmBlink := TimeManager.Data.Pulses.Milliseconds.Pulse_100ms;
END_IF;
```

---

## Status Monitoring

Monitor the FB status through `TimeManager.Data.Status.*`:

```scl
// Check if enabled
IF TimeManager.Data.Status.IsEnabled THEN
    // FB is running
END_IF;

// Check if initialized
IF TimeManager.Data.Status.IsInitialized THEN
    // FB is ready
END_IF;

// Check custom time status
IF TimeManager.Data.Status.CustomTimeActive THEN
    // Custom time is configured
END_IF;

// Get cycle time
CycleTime_us := TimeManager.Data.Status.CycleTime_us;
```

---

## LTime Format Reference

The `CustomTime` input uses **LTime** (Long Time) data type:

| Time Unit | Format | Example | Value |
|-----------|--------|---------|-------|
| Nanoseconds | `LTIME#xNS` | `LTIME#1NS` | 1 nanosecond |
| Microseconds | `LTIME#xUS` | `LTIME#100US` | 100 microseconds |
| Milliseconds | `LTIME#xMS` | `LTIME#500MS` | 500 milliseconds |
| Seconds | `LTIME#xS` | `LTIME#30S` | 30 seconds |
| Minutes | `LTIME#xM` | `LTIME#5M` | 5 minutes |
| Hours | `LTIME#xH` | `LTIME#12H` | 12 hours |
| Days | `LTIME#xD` | `LTIME#30D` | 30 days |

### Combined Time Values

```scl
LTIME#1S500MS           // 1.5 seconds
LTIME#2M30S             // 2 minutes 30 seconds
LTIME#1H30M             // 1 hour 30 minutes
LTIME#1D12H             // 1.5 days
LTIME#1H23M45S670MS     // Complex: 1h 23m 45s 670ms
```

### Extended Range Examples

```scl
LTIME#7D                // 1 week
LTIME#30D               // 1 month (30 days) ✅ NOW WORKS!
LTIME#365D              // 1 year (365 days) ✅ NOW WORKS!
LTIME#1000D             // 1000 days - also supported!
```

**LTime Range:** Up to **292 years** (vs TIME limited to 24.85 days)

---

## Technical Details

### TIME vs LTime - Problem Solved!

**The Problem (v1.0):**
- TIME data type max: **T#24D20H31M23S647MS** (2,147,483,647 milliseconds)
- Month (720 hours) and Year (8760 hours) exceeded this limit
- TIA Portal would not accept these values

**The Solution (v3.0):**
- Uses **LTime** data type
- Maximum value: **292 years**
- Fully supports microseconds to years
- No more TIME overflow errors!

### System Requirements

- **PLC**: Siemens S7-1200 or S7-1500
- **TIA Portal**: V13 SP1 or later (for LTime support)
- **System Function**: READ_CLK (automatic time reading)

### Performance

- **Memory per instance**: ~200 bytes
- **CPU Load**: < 1% typical
- **Accuracy**: Depends on OB cycle time
- **Microsecond precision**: Requires fast cyclic interrupt OBs

### Time Calculation Method

The FB uses:
1. **READ_CLK**: Reads PLC system time (DTL format)
2. **Nanosecond conversion**: Converts to nanoseconds for precision
3. **Delta calculation**: Tracks time elapsed since last scan
4. **Rollover handling**: Automatically handles day rollovers

---

## Troubleshooting

### Issue: Pulses not toggling

**Solution:**
- Verify `Enable := TRUE`
- Ensure FB is called in cyclic execution (OB1)
- Check PLC system clock is set (READ_CLK works)
- Verify `Data.Status.IsInitialized := TRUE`

### Issue: Month/Year pulses don't work

**Solution:**
- Ensure you're using **General_FB_v3.scl** (not v1.0)
- Verify TIA Portal supports LTime (V13 SP1+)
- Check sufficient time has elapsed for testing
- Use simulation to speed up testing

### Issue: Custom pulse not active

**Solution:**
- Verify `CustomTime > LTIME#0NS`
- Check `Data.Pulses.Custom.Active := TRUE`
- Ensure CustomTime is valid LTime format

### Issue: Compilation errors

**Solution:**
- Import **UDT_General_FB.scl** first, then **General_FB_v3.scl**
- Verify TIA Portal version (V13 SP1+ required for LTime)
- Check for typos in custom code

---

## Comparison of Versions

| Feature | v1.0 | v2.0 | v3.0 ⭐ |
|---------|------|------|--------|
| UDT Files | 0 | 4 files | **1 file** |
| Function Block | 1 | 1 | 1 |
| Data Type | TIME | LTime | LTime |
| Max Range | 24 days | 292 years | 292 years |
| Month Pulse | ❌ Error | ✅ Works | ✅ Works |
| Year Pulse | ❌ Error | ✅ Works | ✅ Works |
| Import Complexity | Simple | Complex | **Simple** |
| Professional Structure | ❌ | ✅ | ✅ |
| Single UDT | ❌ | ❌ | ✅ |

**Recommendation**: Use **v3.0** - Best of both worlds!

---

## Advanced Features

### Reset Counters

Reset all internal elapsed times without disabling the FB:

```scl
TimeManager(
    Enable := TRUE,
    ResetCounters := TRUE  // Reset once
);

// Remember to set back to FALSE
ResetFlag := FALSE;
```

### Multiple Time Managers

Create multiple instances for different purposes:

```scl
VAR
    SystemTimer : General_FB;
    ProcessTimer : General_FB;
    AlarmTimer : General_FB;
END_VAR

SystemTimer(Enable := TRUE);
ProcessTimer(Enable := ProcessRunning);
AlarmTimer(Enable := AlarmActive, CustomTime := LTIME#750MS);
```

### Conditional Enable/Disable

```scl
TimeManager(
    Enable := (SystemMode = AUTOMATIC) AND NOT EmergencyStop
);
```

---

## Best Practices

1. **Single Instance**: Use one TimeManager for entire application
2. **Global Access**: Declare in Global DB for multi-block access
3. **Edge Detection**: Use R_TRIG/F_TRIG for one-shot events
4. **Status Checks**: Monitor Status outputs for diagnostics
5. **Custom Time Validation**: Ensure CustomTime > 0 before using
6. **Documentation**: Document which pulses are used where
7. **Reset Carefully**: Use ResetCounters only when needed

---

## Examples in Separate File

See **General_FB_v3_Examples.scl** for:
- Complete OB1 implementation
- Data logger function block
- Alarm controller
- Heartbeat monitor
- And more!

---

## Migration Guide

### From v1.0 to v3.0

**Before (v1.0):**
```scl
TimeManager(
    Enable := TRUE,
    CustomTime := T#1S500MS
);
LED := TimeManager.Pulse_1s;
```

**After (v3.0):**
```scl
TimeManager(
    Enable := TRUE,
    CustomTime := LTIME#1S500MS  // Changed to LTIME
);
LED := TimeManager.Data.Pulses.Seconds.Pulse_1s;  // Added .Data.Pulses
```

### From v2.0 to v3.0

**Before (v2.0):**
```scl
VAR
    TimeManager : General_FB;
    Config : UDT_General_FB_Config;  // Separate config UDT
END_VAR

Config.Enable := TRUE;
TimeManager(Config := Config);
LED := TimeManager.Pulses.Seconds.Pulse_1s;
```

**After (v3.0):**
```scl
VAR
    TimeManager : General_FB;  // No separate config needed!
END_VAR

TimeManager(Enable := TRUE);  // Direct inputs
LED := TimeManager.Data.Pulses.Seconds.Pulse_1s;  // Added .Data prefix
```

---

## Support

For additional help:
1. Check **General_FB_v3_Examples.scl** for code examples
2. Review troubleshooting section above
3. Verify TIA Portal version (V13 SP1+ required)

---

## Version History

- **v3.0** (Current - Recommended) ⭐
  - Single UDT file for everything
  - Simplified structure with .Data access
  - LTime support maintained
  - Easy import (just 2 files!)
  - All v2.0 features preserved

- **v2.0** (Deprecated)
  - 4 separate UDT files
  - Complex import process
  - LTime support added
  - Professional structure

- **v1.0** (Legacy)
  - No UDT structure
  - TIME data type (limited range)
  - Month/Year pulses didn't work

---

## License

This function block is provided as open-source for use in PLC projects.

---

## Summary

**General_FB v3.0** is the perfect solution for time management in your PLC:

✅ **Simple**: Only 2 files to import
✅ **Professional**: Fully structured with UDTs
✅ **Complete**: Microseconds to years
✅ **Reliable**: LTime fixes TIME overflow
✅ **Flexible**: Custom time periods
✅ **Monitored**: Built-in status information

**Import, configure, and start using pulses in minutes!**
