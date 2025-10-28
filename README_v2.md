# General_FB v2.0 - Professional Time Pulse Generator

## Overview

The `General_FB` v2.0 is a professional, UDT-based time management function block written in SCL (Structured Control Language) for Siemens S7-1200/1500 PLCs. It generates multiple pulse signals that toggle at predefined time intervals ranging from microseconds to years, as well as custom time periods.

**Version 2.0 introduces a complete restructure using User Defined Types (UDTs) for maximum professionalism and maintainability.**

## Key Improvements in v2.0

- **UDT-Based Architecture**: Professional structured data types for all inputs, outputs, and internal variables
- **LTime Support**: Uses LTime data type to support extended time ranges (up to 292 years)
- **Fixed TIME Overflow**: Resolved TIA Portal TIME data type limitations for month and year periods
- **Organized Pulse Structure**: Pulses grouped by time scale (microseconds, milliseconds, seconds, etc.)
- **Status Monitoring**: Built-in status information including cycle time measurement
- **Better Maintainability**: Clear separation of concerns with dedicated UDTs

## File Structure

```
FunctionBlocks/
├── UDT_General_FB_Config.scl          - Configuration structure
├── UDT_General_FB_Pulses.scl          - Pulse outputs structure
├── UDT_General_FB_Internal.scl        - Internal working data structure
├── UDT_General_FB_TimeConstants.scl   - Time constants structure
├── General_FB_v2.scl                  - Main function block (v2.0)
└── General_FB_v2_Examples.scl         - Comprehensive usage examples
```

## UDT Structures

### 1. UDT_General_FB_Config (Input Configuration)

```scl
STRUCT
   Enable : Bool;              // Enable/Disable the function block
   CustomTime : LTime;         // Custom time input (extended range support)
   ResetCounters : Bool;       // Reset all internal counters
END_STRUCT;
```

### 2. UDT_General_FB_Pulses (Output Pulses)

Pulses are organized hierarchically by time scale:

```scl
STRUCT
   Microseconds : STRUCT
      Pulse_1us   : Bool;     // 1 microsecond
      Pulse_10us  : Bool;     // 10 microseconds
      Pulse_100us : Bool;     // 100 microseconds
   END_STRUCT;

   Milliseconds : STRUCT
      Pulse_1ms   : Bool;     // 1 millisecond
      Pulse_10ms  : Bool;     // 10 milliseconds
      Pulse_100ms : Bool;     // 100 milliseconds
   END_STRUCT;

   Seconds : STRUCT
      Pulse_1s    : Bool;     // 1 second
      Pulse_10s   : Bool;     // 10 seconds
   END_STRUCT;

   Minutes : STRUCT
      Pulse_1min  : Bool;     // 1 minute
      Pulse_10min : Bool;     // 10 minutes
   END_STRUCT;

   Hours : STRUCT
      Pulse_1h    : Bool;     // 1 hour
   END_STRUCT;

   Days : STRUCT
      Pulse_1d    : Bool;     // 1 day
      Pulse_7d    : Bool;     // 7 days (week)
   END_STRUCT;

   Months : STRUCT
      Pulse_30d   : Bool;     // 30 days (month)
   END_STRUCT;

   Years : STRUCT
      Pulse_365d  : Bool;     // 365 days (year)
   END_STRUCT;

   Custom : STRUCT
      Pulse       : Bool;     // Custom time pulse
      Active      : Bool;     // Custom time configured
   END_STRUCT;
END_STRUCT;
```

### 3. UDT_General_FB_Internal (Internal Data)

Internal working variables organized by time scale (not directly accessed by user).

### 4. UDT_General_FB_TimeConstants (Time Constants)

Pre-configured time constants using LTime for extended range support.

## Features

### Automatic Pulse Generation

| Time Scale | Pulses Available | Access Path |
|------------|------------------|-------------|
| Microseconds | 1μs, 10μs, 100μs | `Pulses.Microseconds.Pulse_*` |
| Milliseconds | 1ms, 10ms, 100ms | `Pulses.Milliseconds.Pulse_*` |
| Seconds | 1s, 10s | `Pulses.Seconds.Pulse_*` |
| Minutes | 1min, 10min | `Pulses.Minutes.Pulse_*` |
| Hours | 1h | `Pulses.Hours.Pulse_*` |
| Days | 1d, 7d | `Pulses.Days.Pulse_*` |
| Months | 30d | `Pulses.Months.Pulse_*` |
| Years | 365d | `Pulses.Years.Pulse_*` |
| Custom | Variable | `Pulses.Custom.Pulse` |

### Status Information

```scl
Status : STRUCT
   IsEnabled : Bool;              // FB is enabled
   IsInitialized : Bool;          // FB is initialized
   CustomTimeActive : Bool;       // Custom time is configured
   CycleTime_us : UDInt;          // Current cycle time in microseconds
END_STRUCT;
```

## Installation in TIA Portal

### Step 1: Import UDTs

Import the UDT files in the following order:

1. `UDT_General_FB_TimeConstants.scl`
2. `UDT_General_FB_Config.scl`
3. `UDT_General_FB_Pulses.scl`
4. `UDT_General_FB_Internal.scl`

**How to import:**
- Open TIA Portal
- Navigate to "PLC data types" → "Add new data type" → "Import source file"
- Select each .scl file and compile

### Step 2: Import Function Block

5. Import `General_FB_v2.scl` into "Program blocks" → "Add new block" → "Import source file"
6. Compile the function block

### Step 3: Verify Compilation

- Ensure all UDTs compile without errors
- Ensure the function block compiles without errors
- Check for any warnings (microsecond precision warnings are normal)

## Usage Examples

### Example 1: Basic Usage

```scl
VAR
    TimeManager : "General_FB";
    Config : "UDT_General_FB_Config";
    LED : Bool;
END_VAR

// Configure
Config.Enable := TRUE;
Config.CustomTime := LTIME#1S500MS;  // 1.5 seconds

// Call FB
TimeManager(Config := Config);

// Use pulse
LED := TimeManager.Pulses.Seconds.Pulse_1s;
```

### Example 2: Event Counter

```scl
VAR
    TimeManager : "General_FB";
    Config : "UDT_General_FB_Config";
    Edge_1s : R_TRIG;
    Counter : Int;
END_VAR

// Configure and call
Config.Enable := TRUE;
TimeManager(Config := Config);

// Count seconds
Edge_1s(CLK := TimeManager.Pulses.Seconds.Pulse_1s);
IF Edge_1s.Q THEN
    Counter := Counter + 1;
END_IF;
```

### Example 3: Multi-Rate Blinking

```scl
VAR
    TimeManager : "General_FB";
    Config : "UDT_General_FB_Config";
    LED_Fast : Bool;
    LED_Medium : Bool;
    LED_Slow : Bool;
END_VAR

Config.Enable := TRUE;
TimeManager(Config := Config);

// Different blink rates
LED_Fast := TimeManager.Pulses.Milliseconds.Pulse_100ms;  // 100ms
LED_Medium := TimeManager.Pulses.Seconds.Pulse_1s;        // 1 second
LED_Slow := TimeManager.Pulses.Seconds.Pulse_10s;         // 10 seconds
```

### Example 4: Custom Time Period

```scl
VAR
    TimeManager : "General_FB";
    Config : "UDT_General_FB_Config";
    CustomOutput : Bool;
END_VAR

// Configure custom time (2 minutes 30 seconds)
Config.Enable := TRUE;
Config.CustomTime := LTIME#2M30S;

TimeManager(Config := Config);

// Use custom pulse
CustomOutput := TimeManager.Pulses.Custom.Pulse;

// Check if custom time is active
IF TimeManager.Pulses.Custom.Active THEN
    // Custom time is configured
END_IF;
```

### Example 5: Data Logger

```scl
VAR
    TimeManager : "General_FB";
    Config : "UDT_General_FB_Config";
    Edge_10s : R_TRIG;
    Edge_1min : R_TRIG;
    Edge_1h : R_TRIG;
    DataToLog : Real;
END_VAR

Config.Enable := TRUE;
TimeManager(Config := Config);

// Log every 10 seconds
Edge_10s(CLK := TimeManager.Pulses.Seconds.Pulse_10s);
IF Edge_10s.Q THEN
    // Log high-frequency data
    LogData_10s(DataToLog);
END_IF;

// Log every minute
Edge_1min(CLK := TimeManager.Pulses.Minutes.Pulse_1min);
IF Edge_1min.Q THEN
    // Log medium-frequency data
    LogData_1min(DataToLog);
END_IF;

// Log every hour
Edge_1h(CLK := TimeManager.Pulses.Hours.Pulse_1h);
IF Edge_1h.Q THEN
    // Log low-frequency data
    LogData_1h(DataToLog);
END_IF;
```

## LTime Format Reference

The CustomTime input uses **LTime** data type which supports extended time ranges:

| Format | Example | Description |
|--------|---------|-------------|
| Nanoseconds | `LTIME#1NS` | 1 nanosecond |
| Microseconds | `LTIME#1US` | 1 microsecond |
| | `LTIME#100US` | 100 microseconds |
| Milliseconds | `LTIME#1MS` | 1 millisecond |
| | `LTIME#500MS` | 500 milliseconds |
| | `LTIME#1S500MS` | 1.5 seconds |
| Seconds | `LTIME#1S` | 1 second |
| | `LTIME#30S` | 30 seconds |
| | `LTIME#1M30S` | 1 minute 30 seconds |
| Minutes | `LTIME#1M` | 1 minute |
| | `LTIME#5M` | 5 minutes |
| | `LTIME#1H30M` | 1 hour 30 minutes |
| Hours | `LTIME#1H` | 1 hour |
| | `LTIME#12H` | 12 hours |
| | `LTIME#1D12H` | 1.5 days |
| Days | `LTIME#1D` | 1 day |
| | `LTIME#7D` | 7 days |
| | `LTIME#30D` | 30 days |
| | `LTIME#365D` | 365 days (1 year) |

**Complex Example:**
```scl
Config.CustomTime := LTIME#1H23M45S670MS;  // 1 hour, 23 min, 45 sec, 670 ms
```

## Technical Considerations

### 1. TIME vs LTime Data Types

**Problem with TIME:** The standard TIME data type in TIA Portal has a maximum value of approximately 24.85 days (T#24D20H31M23S647MS), which equals 2,147,483,647 milliseconds. This limitation prevented support for monthly and yearly pulses.

**Solution with LTime:** Version 2.0 uses LTime (Long Time) data type which supports time ranges up to 292 years, completely solving the overflow issue.

### 2. Cycle Time Dependency

Pulse accuracy depends on:
- PLC cycle time (OB1 scan rate)
- System clock resolution
- Execution priority

**Recommendations:**
- For millisecond precision: Standard OB1 (cyclic execution)
- For microsecond precision: Use cyclic interrupt OBs or hardware timers
- Monitor `Status.CycleTime_us` to verify system performance

### 3. System Time Function

The FB uses `READ_CLK` system function to get current time:
- Works on S7-1200/1500 PLCs
- Requires system clock to be set
- Automatically handles day rollovers

### 4. Memory Usage

Total memory per instance:
- Config: ~12 bytes
- Pulses: ~20 bytes
- Internal: ~160 bytes
- Status: ~8 bytes
- **Total: ~200 bytes per instance**

### 5. Performance

Expected CPU load:
- Typical: < 1% CPU usage
- Depends on: PLC model, cycle time, number of instances
- Monitor using TIA Portal diagnostics

## Troubleshooting

### Issue 1: Pulses Not Toggling

**Symptoms:** Pulse outputs remain constant

**Solutions:**
- Verify `Config.Enable := TRUE`
- Check `Status.IsInitialized := TRUE`
- Ensure FB is called cyclically in OB1
- Verify system clock is set (`READ_CLK` returns valid time)

### Issue 2: Inaccurate Timing

**Symptoms:** Pulses toggle at incorrect intervals

**Solutions:**
- Check PLC cycle time (should be consistent)
- Monitor `Status.CycleTime_us` for irregularities
- Reduce OB1 scan time if too long
- For microsecond precision, use cyclic interrupt OBs

### Issue 3: Compilation Errors

**Symptoms:** Cannot compile UDTs or FB

**Solutions:**
- Import UDTs before importing FB
- Ensure TIA Portal version supports LTime (V13 SP1 or later)
- Check for syntax errors in custom modifications
- Verify all UDT files are in the project

### Issue 4: Month/Year Pulses Not Working

**Symptoms:** Pulse_30d or Pulse_365d don't toggle

**Solutions:**
- Ensure you're using General_FB_v2.scl (not v1.0)
- Verify LTime data type is supported in your PLC
- Check that enough time has elapsed (30/365 days)
- Use simulation mode to speed up testing

## Migration from v1.0 to v2.0

If you're using the original General_FB, follow these steps to migrate:

### Step 1: Backup Your Project

Create a complete backup before migration.

### Step 2: Update Function Block Calls

**Old (v1.0):**
```scl
VAR
    TimeManager : General_FB;
END_VAR

TimeManager(
    Enable := TRUE,
    CustomTime := T#1S500MS
);

LED := TimeManager.Pulse_1s;
```

**New (v2.0):**
```scl
VAR
    TimeManager : General_FB;
    Config : UDT_General_FB_Config;
END_VAR

Config.Enable := TRUE;
Config.CustomTime := LTIME#1S500MS;  // Note: LTIME instead of T#

TimeManager(Config := Config);

LED := TimeManager.Pulses.Seconds.Pulse_1s;  // Note: structured access
```

### Step 3: Update Pulse Access

| v1.0 Access | v2.0 Access |
|-------------|-------------|
| `Pulse_1us` | `Pulses.Microseconds.Pulse_1us` |
| `Pulse_1ms` | `Pulses.Milliseconds.Pulse_1ms` |
| `Pulse_1s` | `Pulses.Seconds.Pulse_1s` |
| `Pulse_1min` | `Pulses.Minutes.Pulse_1min` |
| `Pulse_1h` | `Pulses.Hours.Pulse_1h` |
| `Pulse_1d` | `Pulses.Days.Pulse_1d` |
| `Pulse_1w` | `Pulses.Days.Pulse_7d` |
| `Pulse_1mo` | `Pulses.Months.Pulse_30d` |
| `Pulse_1y` | `Pulses.Years.Pulse_365d` |
| `CustomPulse` | `Pulses.Custom.Pulse` |

## Best Practices

1. **Single Instance:** Create one TimeManager instance for the entire application
2. **Centralized Configuration:** Store Config in a global DB for easy access
3. **Edge Detection:** Use R_TRIG/F_TRIG for one-shot events
4. **Status Monitoring:** Check Status outputs for diagnostics
5. **Reset Counters:** Use Config.ResetCounters carefully (will reset all timers)
6. **Custom Time:** Validate CustomTime > 0 before using Custom.Pulse
7. **Documentation:** Document which pulses are used for which purposes

## Performance Optimization

1. **Disable Unused Instances:** Set Enable := FALSE when not needed
2. **Minimize Instances:** Use one instance with multiple outputs
3. **Cyclic Execution:** Call in OB1 for best performance
4. **Monitor Cycle Time:** Keep Status.CycleTime_us < 10ms for best accuracy

## Licensing

This function block is provided as open-source for use in PLC projects. Feel free to modify and distribute.

## Version History

- **v2.0** (Current)
  - Complete UDT-based restructure
  - LTime support for extended time ranges
  - Fixed TIME overflow for month/year pulses
  - Organized pulse outputs by time scale
  - Added status monitoring
  - Improved documentation and examples

- **v1.0** (Legacy)
  - Initial release
  - Basic pulse generation
  - TIME data type (limited to 24 days)
  - Flat output structure

## Support

For issues, questions, or contributions:
- Review the examples in `General_FB_v2_Examples.scl`
- Check the troubleshooting section above
- Verify all UDTs are properly imported

## Credits

Developed for professional PLC applications using Siemens TIA Portal and S7-1200/1500 PLCs.
