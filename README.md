# General_FB - Time Pulse Generator Function Block

## Overview

The `General_FB` is a comprehensive time management function block written in SCL (Structured Control Language) for Siemens PLCs. It generates multiple pulse signals that toggle at predefined time intervals ranging from microseconds to years, as well as custom time periods.

## Features

- **Automatic Pulse Generation**: Pre-configured pulse outputs for common time periods
- **Custom Time Input**: Flexible input for non-standard time periods (e.g., 1.5 seconds, 3.7 minutes)
- **Wide Range**: Supports time periods from 1 microsecond to 1 year
- **Enable/Disable Control**: Easy control to enable or disable all pulse generation
- **Optimized Access**: Uses S7 optimized access for better performance

## Inputs

| Input Name | Data Type | Default | Description |
|------------|-----------|---------|-------------|
| `CustomTime` | TIME | - | Custom time input for non-standard periods (e.g., T#1S500MS for 1.5 seconds) |
| `Enable` | BOOL | TRUE | Enable/Disable the function block |

## Outputs

### Automatic Pulse Outputs

| Output Name | Toggle Period | Description |
|-------------|---------------|-------------|
| `Pulse_1us` | 1 microsecond | Toggles every 1 μs |
| `Pulse_10us` | 10 microseconds | Toggles every 10 μs |
| `Pulse_100us` | 100 microseconds | Toggles every 100 μs |
| `Pulse_1ms` | 1 millisecond | Toggles every 1 ms |
| `Pulse_10ms` | 10 milliseconds | Toggles every 10 ms |
| `Pulse_100ms` | 100 milliseconds | Toggles every 100 ms |
| `Pulse_1s` | 1 second | Toggles every 1 second |
| `Pulse_10s` | 10 seconds | Toggles every 10 seconds |
| `Pulse_1min` | 1 minute | Toggles every 1 minute |
| `Pulse_10min` | 10 minutes | Toggles every 10 minutes |
| `Pulse_1h` | 1 hour | Toggles every 1 hour |
| `Pulse_1d` | 1 day | Toggles every 24 hours |
| `Pulse_1w` | 1 week | Toggles every 7 days |
| `Pulse_1mo` | 1 month | Toggles every 30 days |
| `Pulse_1y` | 1 year | Toggles every 365 days |
| `CustomPulse` | Variable | Toggles based on `CustomTime` input |

## Usage Examples

### Example 1: Using Pre-configured Pulses

```scl
// Declare instance of General_FB
VAR
    TimeManager : General_FB;
    BlinkLED : BOOL;
END_VAR

// In cyclic execution
TimeManager(Enable := TRUE);

// Use 1-second pulse to blink an LED
BlinkLED := TimeManager.Pulse_1s;
```

### Example 2: Using Custom Time Period

```scl
// Declare instance of General_FB
VAR
    TimeManager : General_FB;
    CustomBlink : BOOL;
END_VAR

// In cyclic execution
// Toggle every 1.5 seconds
TimeManager(
    Enable := TRUE,
    CustomTime := T#1S500MS
);

CustomBlink := TimeManager.CustomPulse;
```

### Example 3: Multiple Time Periods

```scl
// Declare instance of General_FB
VAR
    TimeManager : General_FB;
    FastBlink : BOOL;
    SlowBlink : BOOL;
    HourlyTrigger : BOOL;
END_VAR

// In cyclic execution
TimeManager(Enable := TRUE);

// Use different pulses for different purposes
FastBlink := TimeManager.Pulse_100ms;    // Fast blink (100ms)
SlowBlink := TimeManager.Pulse_1s;       // Slow blink (1 second)
HourlyTrigger := TimeManager.Pulse_1h;   // Hourly event trigger
```

### Example 4: Custom Time with Complex Period

```scl
// Declare instance of General_FB
VAR
    TimeManager : General_FB;
    SpecialPulse : BOOL;
END_VAR

// In cyclic execution
// Toggle every 2 minutes and 30 seconds
TimeManager(
    Enable := TRUE,
    CustomTime := T#2M30S
);

SpecialPulse := TimeManager.CustomPulse;
```

### Example 5: Edge Detection for Pulse Events

```scl
// Declare variables
VAR
    TimeManager : General_FB;
    Pulse_1s_Edge : R_TRIG;
    Counter : INT;
END_VAR

// In cyclic execution
TimeManager(Enable := TRUE);

// Detect rising edge of 1-second pulse
Pulse_1s_Edge(CLK := TimeManager.Pulse_1s);

// Increment counter every second
IF Pulse_1s_Edge.Q THEN
    Counter := Counter + 1;
END_IF;
```

## Implementation Notes

### Important Considerations

1. **System Time Function**: The function block uses `TIME()` as a placeholder. In your actual PLC implementation, replace this with the appropriate system time function:
   - **S7-300/400**: Use `%SW80` (system time in milliseconds) or `TIME()` system function
   - **S7-1200/1500**: Use `RD_SYS_T` or `RD_LOC_T` system functions
   - **S7-1500 Safety**: Use appropriate safety-rated time functions

2. **Cycle Time Dependency**: The accuracy of pulse generation depends on:
   - PLC cycle time
   - System clock resolution
   - OB1 or cyclic interrupt OB scan rate

3. **Very Fast Pulses (1μs - 100μs)**:
   - For microsecond-level pulses, consider using:
     - Cyclic interrupt OBs with faster execution
     - Hardware counters/timers
     - Time-of-day interrupts
   - Standard cyclic execution may not achieve microsecond precision

4. **Memory Usage**: The FB maintains 16 separate elapsed time counters. Consider memory constraints on smaller PLCs.

## Integration Steps

1. **Import the Function Block**:
   - Open TIA Portal or your Siemens PLC programming software
   - Import `General_FB.scl` into your project
   - Compile the function block

2. **Create Instance**:
   - In your main program (e.g., Main OB1), declare an instance of `General_FB`
   - Example: `VAR TimeManager : General_FB; END_VAR`

3. **Call in Cyclic Execution**:
   - Call the FB in your cyclic execution block (OB1 or cyclic interrupt OB)
   - Enable the FB: `TimeManager(Enable := TRUE);`

4. **Use Outputs**:
   - Connect the pulse outputs to your application logic
   - Use edge detection (R_TRIG, F_TRIG) if you need single-shot events

## Modifications for Specific PLC Platforms

### S7-1200/1500

Replace the `TIME()` function with system time reading:

```scl
// Add to VAR_TEMP
lastSysTime : DTL;
currentSysTime : DTL;

// Replace TIME() with:
RD_SYS_T(OUT => #currentSysTime);
// Calculate delta time from DTL values
```

### S7-300/400

Use system clock memory:

```scl
// Replace TIME() with:
#cycleTime := TIME_TO_DINT(%SW80);  // System time in ms
```

## Testing

To test the function block:

1. Create a test program that instances `General_FB`
2. Monitor the pulse outputs in online mode
3. Verify timing accuracy with a stopwatch or timer
4. Test edge cases (Enable/Disable transitions, CustomTime = 0)

## License

This function block is provided as-is for use in PLC projects.

## Version History

- **v0.1** - Initial release with support for microseconds to years
