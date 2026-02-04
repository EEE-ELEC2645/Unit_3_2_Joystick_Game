# ELEC2645 - Joystick Game (Target Collection)

Now we are comfortable controlling graphics on the LCD using the joystick, we can add some more complexity and start building some basic games! In this lab we also add improved Random Number Generation using the `RNG.h` peripheral, and the use a non-blocking delay by using the `HAL_GetTick()` function, rather than the simple `HAL_DELAY()`. 

The most important file is [Src/main.c](Src/main.c) which contains the main game loop and student activity.

## The Project

The program implements a simple collection game where the player controls a dot using an analog joystick. The objective is to move around the screen and collect randomly-placed targets. The game features:

1. **Player Control** - Move a blue dot around the screen using the joystick in 8 directions (N, NE, E, SE, S, SW, W, NW)
2. **Target Objects** - Multiple targets spawn randomly on screen
3. **Collision Detection** - Targets disappear when the player touches them
4. **Smooth Movement** - Pixel-based movement with configurable speed and update rate
5. **Boundary Detection** - Player is confined to the screen boundaries

The game uses circle-based collision detection, which provides natural, smooth collision behavior. Two circles collide when the distance between their centers is less than the sum of their radii.

## The Assignment

Your task is to enhance the game by adding **scoring and target respawning**:

**Your Tasks:**
1. **Add a score counter** - Create a variable to track how many targets the player has collected
2. **Display the score** - Show the current score on the LCD screen
3. **Implement target respawning** - When a target is collected, spawn a new one at a random location
4. **(Bonus) Variable speed** - Use joystick magnitude to control player speed (push harder = move faster)
5. **(Bonus) Timer** - Use `HAL_GetTick()` to implement a game timer. Reset Score and position when its finished

Once you have completed these tasks hopefully you will have a game that could almost be considered fun! Submit your `main.c` to the Code Submission area on Minerva.  

## Game Parameters

The following constants in [Src/main.c](Src/main.c) can be adjusted to modify gameplay:

```c
#define LCD_WIDTH 240          // Display width in pixels
#define LCD_HEIGHT 240         // Display height in pixels
#define PLAY_AREA_Y0 20        // Top margin for title text

#define PLAYER_RADIUS 5        // Size of player circle
#define TARGET_RADIUS 4        // Size of target circles
#define TARGET_COUNT 5         // Number of simultaneous targets

#define MOVE_SPEED 3           // Pixels to move per update
#define MOVE_DELAY_MS 50       // Milliseconds between movement updates
```

**Experimentation Ideas:**
- Increase `MOVE_SPEED` for faster gameplay
- Decrease `MOVE_DELAY_MS` for more responsive controls - there will be a limit to this!
- Increase `TARGET_COUNT` for more challenge
- Modify radii to change difficulty

## How the Game Loop Works

The main game loop follows a classic game architecture pattern:

1. **INPUT** - Read joystick position and determine direction
2. **MOVEMENT** - Update player pixel position based on input (rate-limited)
3. **RENDERING** - Erase old player position, draw new position
4. **COLLISION** - Check if player overlaps any targets using circle collision
5. **DISPLAY** - Refresh LCD to show all updates

### Collision Detection Algorithm

The game uses circle-based collision detection:

```c
// Two circles collide when distance between centers < sum of radii
distance_squared = (x2-x1)² + (y2-y1)²
radii_sum_squared = (r1+r2)²
collision = (distance_squared <= radii_sum_squared)
```

This approach avoids the relatively slow `sqrt()` operations by comparing squared distances. i.e. rather than comparing if d < r, we compare if d² < r².

### Target Spawning

Targets are placed using a trial-and-error algorithm:
1. Generate random X/Y coordinates within screen bounds
2. Check collision with player position
3. Check collision with other active targets
4. If collision detected, try again (up to 100 attempts)
5. If successful, draw target at location

This ensures targets don't spawn on top of the player or each other.

## Student Activity Location

Look for the `STUDENT TODO` comment in [Src/main.c](Src/main.c) around line 270 inside the collision detection loop:

```c
// ===== STUDENT TODO =====
// 1) Increment a score counter here.
// 2) Spawn a new target by calling:
//    Place_Target(i, target_x, target_y, target_active, player_x, player_y);
// ========================
```

This is where you should add your code for scoring and respawning targets.


## Setup Instructions

### Prerequisites

1. **Completed previous labs**
   - Blinky and LCD Test to ensure the hardware is working first!
   - Understanding of ADC and joystick control

2. **Configure the Project**
   - Open the `Unit_3_3_Joystick_Game` folder in VS Code
   - When prompted "*Would you like to configure discovered CMake project as STM32Cube project*", click **Yes**
   - Allow the STM32 extension to complete initialization
   - Select **Debug** configuration when prompted

3. **Verify Hardware Connection**
   - Connect the Nucleo board via USB
   - Check that the board appears under "STM32CUBE Devices and Boards" in the Run and Debug sidebar
   - Test with the **Blink** function to verify communication

4. **Build and Run**
   - Click **Build** in the bottom status bar to verify compilation
   - Open Run and Debug panel (`Ctrl+Shift+D`)
   - Select **"STM32Cube: STLink GDB Server"** and click Run
   - Continue past breakpoints with **F5** or the play button


   ## Hardware Configuration

### Analog Joystick Connection

| Joystick Pin | Signal | Nucleo Pin | Purpose |
|-------------|--------|-----------|---------|
| VCC         | Power (3.3V) | 3.3V | Joystick power supply |
| GND         | Ground | GND | Ground reference |
| X-axis      | Analog X | PA1 (ADC1 CH1) | Horizontal position |
| Y-axis      | Analog Y | PA2 (ADC1 CH2) | Vertical position |
| Button      | (Optional) | - | Can be added for digital input |

⚠️ **Important**: Connect joystick VCC to **3.3V**, not 5V, to prevent ADC saturation and ensure accurate readings.

### LCD Display Connection (ST7789V2)

| LCD Pin | Signal | Nucleo Pin | Purpose |
|---------|--------|-----------|---------|
| VDD     | Power (3.3V-5V) | VDD | Display power supply |
| GND     | Ground | GND | Ground reference |
| MOSI    | Serial Data | PB15 | SPI Master Output Slave Input |
| SCK     | Clock | PB13 | SPI Clock signal |
| CS      | Chip Select | PB12 | SPI Chip Select |
| DC      | Data/Command | PB11 | Command vs Data mode |
| BL      | Backlight | PB1 | Backlight control (active high) |
| RST     | Reset | PB2 | Display reset (active low) |

### Serial Communication
- **UART Interface**: USB connection via ST Link debugger
- **Baud Rate**: 115200
- **Purpose**: Debug output via `printf()` showing joystick calibration values

## Software Architecture

### Key Source Files

| File | Purpose |
|------|---------|
| [Src/main.c](Src/main.c) | **Main game logic** - Game loop, player movement, collision detection, target spawning |
| [Joystick/Joystick.c](Joystick/Joystick.c) | Joystick driver with circle mapping and polar coordinate conversion |
| [Joystick/Joystick.h](Joystick/Joystick.h) | Joystick API and configuration structures |
| [Inc/main.h](Inc/main.h) | Main header with configuration macros and function prototypes |
| [Src/gpio.c](Src/gpio.c) | GPIO peripheral initialization |
| [Src/adc.c](Src/adc.c) | ADC peripheral configuration for analog input |
| [Src/rng.c](Src/rng.c) | Hardware random number generator initialization |
| [Src/stm32l4xx_it.c](Src/stm32l4xx_it.c) | Interrupt handlers |
| [Src/system_stm32l4xx.c](Src/system_stm32l4xx.c) | System initialization and clock setup |

### LCD Driver
The project includes an external ST7789V2 LCD driver:
- **Location**: [ST7789V2_Driver_STM32L4/](ST7789V2_Driver_STM32L4/)
- **Key Functions**:
  - `LCD_init(cfg)` - Initialize LCD with configuration
  - `LCD_Fill_Buffer(color)` - Fill entire screen with color index
  - `LCD_printString(text, x, y, color, size)` - Display text at coordinates
  - `LCD_Draw_Circle(x, y, radius, color, filled)` - Draw circle (filled or outline)
  - `LCD_Refresh(cfg)` - Transfer frame buffer to LCD hardware
  - See the driver README for complete API documentation

### Joystick Library
The project includes a custom joystick library with advanced features:
- **Location**: [Joystick/](Joystick/)
- **Features**:
  - Circle mapping for uniform control feel
  - Polar coordinate conversion (magnitude and angle)
  - 8-direction discrete output (N, NE, E, SE, S, SW, W, NW, CENTRE)
  - Automatic center calibration
  - Configurable deadzone
- **Key Functions**:
  - `Joystick_Init(cfg)` - Initialize joystick ADC channels
  - `Joystick_Calibrate(cfg)` - Auto-detect center position
  - `Joystick_Read(cfg, data)` - Read and process joystick input
  - See [Joystick/Joystick.h](Joystick/Joystick.h) for full API