# JARVIS TELEMETRY INTEGRATION MANUAL
====================================================================

## 🤖 1. INTRODUCTION & PERFORMANCE BREAKTHROUGH (DEBIAN UPGRADE LOG)
Welcome to the JARVIS Telemetry Integration repository. This project establishes a real-time, bi-directional audio notification bridge between your printer's Klipper engine and your Windows workstation using dynamic Text-to-Speech (TTS). 

### 🔊 Bypassing the Debian Audio Breakage
When updating your hardware to a clean **Debian base and a stock Klipper deployment**, a major systemic limitation occurs: **Debian breaks the printer's native onboard speaker entirely, making it completely useless.** Because the stock Linux environment lacks the proprietary, closed-source audio drivers built into Creality's original factory firmware image, the physical machine loses all capability to produce sound. 

This repository completely solves that problem by offloading the entire audio system to your Windows computer over an optimized network pipe. Instead of struggling to make a broken onboard speaker talk, your printer transmits its data states directly to your workstation's raw hardware power. 

This solution doesn't just get your voice notifications back—**it delivers audio quality that is vastly superior to the built-in speaker.** Instead of listening to raspy, muffled, low-quality speaker sound coming from a buzzing printer control box, JARVIS speaks with flawless clarity directly through your PC's dedicated audio setup, studio monitors, or headphones!

### Why the Architecture Evolved
In your original unboxed layout, the system utilized Creality’s customized factory firmware includes (`cx_printer.cfg`, `timelapse.cfg`, and `cx_gmcro.cfg`). These built-in template streams carried intense data congestion and circular macro loops. When a hardware fault dropped an error box, this accidental software latency loop gave the router a brief head-start to leak custom strings over the network pipe before the motherboard completely locked down.

Following your upgrade to a clean Debian base and modern stock Klipper, all factory legacy includes were permanently deleted. This optimization brought system latency down to true real-time. Because stock, optimized Klipper code prioritizes physical safety above network text streaming protocols, it executes a hardware-level microcontroller unit (MCU) panic shutdown to freeze power the exact millisecond a sensor or BLTouch probe faults. This protects your toolhead nozzle and heated build plate from mechanical impact, but it severs network socket streams instantly—blocking raw G-code text responses from escaping the machine before a connection dropout hits your computer.

Furthermore, standard Klipper firmware logs background calibration loops and multi-point bed mesh data dumps using the exact same console indicator headers (`!!`) that it utilizes for critical red-box fault exceptions. This data crossover caused legacy script layouts to aggressively read out every single point, numeric coordinate, and trailing height decimal over your computer speakers.

This master package resolves these modern Debian-upgrade behaviors by offering two distinct tracking solutions:
* **METHOD A (Unified Firewall Engine):** Tracks macro alerts normally while implementing an internal Exclusion Shield to automatically drop background calibration metrics, allowing orange warning states to speak cleanly exactly once without looping.
* **METHOD B (Isolated Twin Sentinel):** Tracks normal macro cues cleanly on a primary thread, while concurrently deploying an independent background process to query the Moonraker status API database directly, catching connection losses instantly even during an immediate hardware power-cut.

---

## ⏳ 2. THE MASTER PAUSE BLUEPRINT: WHEN, WHERE, & WHY
To maintain absolute data integrity across your workspace, specific pauses are hardcoded into both the printer firmware and the computer scripts. 

### Pause Location A: The Printer Firmware (`G4 P2000` / `G4 P500`)
*   **Where:** In your `CANCEL_PRINT`, `G28` (Homing), and `BED_MESH_CALIBRATE` macros, directly following the `RESPOND` voice cue line.
*   **When to Use:** Every single time your printer configuration issues an outbound audio notification to the network pipe right before executing a physical axis movement or jumping into a system teardown state.
*   **Why It Is Required:** Raw network data streams take a few milliseconds to clear your motherboard cache, travel out of your device's local router card, cross your home network lines, and land in your computer's WebSocket buffer. If Klipper executes an immediate movement command or encounters a hardware halt immediately after writing a text string, it kills the processing clock cycles [1.21]. Without adding an intentional **`G4 P2000` (2-second pause)** or **`G4 P500` (half-second pause)**, the printer will destroy or truncate the message package inside the router's local memory stack before it can ever escape to your PC speakers [1.21]. The pause deliberately holds the printer's breath, ensuring your voice lines travel safely to your computer first [1.21].

### Pause Location B: The Windows Script Memory Lock (`TotalSeconds -gt 5`)
*   **Where:** Inside Rule 2 of your computer script code configurations.
*   **When to Use:** Active continuously whenever your script engine evaluates incoming console warnings and red pop-up exceptions (`!!`).
*   **Why It Is Required:** Klipper streams terminal errors at microsecond rates. If a sensor fails, the background pipeline will occasionally spam the exact same error packet dozens of times in a single second. Without a script-side time-release pause, Windows will attempt to speak every single packet simultaneously, causing JARVIS to stutter, overlap words, loop sentences indefinitely, or crash his text-to-speech module. The script applies a **5-second memory lock**. It speaks the fault box **exactly once**, completely dropping all incoming duplicates until 5 seconds have passed and your ears have heard the full alert phrase cleanly.

---

## 🌐 3. HOW TO FIND & CONFIGURE YOUR PRINTER IP ADDRESS
Before launching the automation, your computer script must know the exact digital destination of your printer backend on your home network. Follow these steps to map the address properly:

### Step 1: Find the IP Address on Your Debian Interface
1. Open your standard web browser and log into your **Mainsail or Fluidd interface dashboard**.
2. Look at your browser's URL address bar. 
3. The group of numbers listed right before the text sections is your printer's network assignment (for example: `192.168.1.29`). Note this down exactly.

### Step 2: Update Your PowerShell Scripts
You must paste this number into **both** script files inside your repository folder so they point to the correct machine:
1. Open your `Jarvis_Repository` folder on your computer.
2. Right-click **`script_unified.ps1`** and choose **Edit** (or Open with Notepad).
3. Find **Line 11** near the top of the file: 
   `$PrinterIP = "192.168.1.29"`
4. Delete the default numbers inside the quotation marks and replace them with your exact network IP address. **Keep the quotation marks intact!** Save and close the file.
5. Repeat this exact process for **Line 11** inside the **`script_sentinel.ps1`** file. Save and close.

---

## 🛠 4. STEP-BY-STEP INSTALLATION FLOW

### Step 1: Configure the Printer Engine (`printer.cfg`)
1. Open your desktop web browser and connect to your printer dashboard web interface (Mainsail or Fluidd).
2. Navigate to the **Machine** or **Configuration Files** folder manager tab on the left sidebar navigation list.
3. Locate and click on the primary configuration layout file named **`printer.cfg`** to open it inside the browser's built-in text editor canvas.
4. Scroll to the absolute bottom of your existing configuration file and completely delete any old custom `[gcode_macro RESPOND]` test blocks left behind from troubleshooting loops to prevent system override structural faults.
5. Highlight, copy, and paste this comprehensive, standardized macro block directly over your existing `[virtual_sdcard]`, homing, mesh, and print control macros. This ensures perfect formatting, lowercase routing tags, and your precise network delays:

```text
[respond]
default_type: echo

[virtual_sdcard]
path: /home/sonic/printer_data/gcodes
on_error_gcode: CANCEL_PRINT

# --- OVERRIDDEN HOMING CORE (ANNOUNCES COMPLETED ROUTINE) ---

[gcode_macro G28]
rename_existing: BASE_G28
gcode:
    # 1. Announce initialization
    RESPOND TYPE=echo MSG="alert: Adjusting positional structural coordinates"
    
    # 2. Execute actual physical axis homing
    BASE_G28 {rawparams}
    
    # 3. Hold network sync and announce completion
    G4 P500                                 ; 0.5-second network buffer clearance pause
    RESPOND TYPE=echo MSG="alert: Positional structural homing completed"

# --- OVERRIDDEN BED MESH CORE (ANNOUNCES COMPLETED ROUTINE) ---

[gcode_macro BED_MESH_CALIBRATE]
rename_existing: BASE_BED_MESH_CALIBRATE
gcode:
    # 1. Announce initialization
    RESPOND TYPE=echo MSG="alert: Structural surface scan initiated"
    
    # 2. Execute multi-point grid calculations
    BASE_BED_MESH_CALIBRATE {rawparams}
    
    # 3. Hold network sync and announce completion
    G4 P500                                 ; 0.5-second network buffer clearance pause
    RESPOND TYPE=echo MSG="alert: Structural surface scan completed. Matrix adjustments applied."

# --- STANDARD PRINT MATRIX CONTROLS ---

[gcode_macro START_PRINT]
description: Code executed immediately upon launching a print job
gcode:
    RESPOND TYPE=echo MSG="alert: Manufacturing sequence initialized"

[gcode_macro END_PRINT]
description: Safe teardown routine when a print finishes successfully
gcode:
    RESPOND TYPE=echo MSG="alert: Manufacturing sequence completed successfully"
