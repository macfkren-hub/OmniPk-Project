# OmniPK Engine

A modular, tick-accurate OSRS PvP intelligence system. Combat HUD + Prediction + Analysis Framework.

## Overview

OmniPK Engine is designed to provide real-time PvP analysis and overlay assistance for Old School RuneScape. It follows strict architectural principles to ensure maintainability, extensibility, and clean separation of concerns.

## Architecture

```
/OmniPK-Engine/
│
├── /core/                    # Logic modules (NO drawing)
│   ├── TickTracker.simba     # Timing source (600ms ticks)
│   ├── FreezeDetection.simba # Freeze status tracking
│   ├── MovementHelper.simba  # Tile safety & recommendations
│   ├── AttackStyleDetector.simba  # Combat style detection
│   ├── PrayerHelper.simba    # Prayer recommendations
│   ├── KOAnalyzer.simba      # KO window detection
│   ├── PatternReader.simba   # Opponent pattern learning
│   ├── PathingPredictor.simba # Movement prediction
│   ├── SplashPredictor.simba # Magic accuracy analysis
│   └── DesyncEngine.simba    # Tick desync advantage
│
├── /overlay/                 # Visual modules (NO logic)
│   ├── HUDOverlay.simba      # Main HUD display
│   ├── TileOverlay.simba     # Tile highlights
│   ├── KOOverlay.simba       # KO warnings
│   ├── PrayerOverlay.simba   # Prayer indicators
│   └── DebugOverlay.simba    # Debug information
│
├── /data/                    # Configuration files
│   ├── weapon_profiles.json  # Weapon stats & max hits
│   ├── spell_profiles.json   # Spell data & freeze durations
│   ├── tile_weights.json     # Movement safety weights
│   ├── pattern_memory.json   # Learned opponent patterns
│   └── config.json           # Global settings
│
├── /utils/                   # Utility modules
│   ├── ColorUtils.simba      # Color manipulation
│   ├── TileUtils.simba       # Tile calculations
│   ├── MathUtils.simba       # Math helpers
│   └── DebugUtils.simba      # Logging utilities
│
├── main.simba                # Entry point & main loop
└── README.md                 # This file
```

## Guardrails

### 1. Module Boundaries
- Each `/core/` file is a self-contained module
- **NO module may draw anything** - drawing is overlay-only
- **NO module may modify another module's state directly**
- Cross-module interaction happens via:
  - Public helper functions (read-only access)
  - Public state records (read-only access)

### 2. Overlay Rules
- Overlays **ONLY read** state from `/core/` modules
- Overlays **NEVER contain logic**
- Overlays **NEVER modify state**
- Overlays must be minimal, high-contrast, non-blocking
- Use 60-80% transparency with neon color palette

### 3. Data Configuration
- **ALL constants, thresholds, and tunables** go in `/data/` JSON files
- **NO hardcoded colors, timings, or weights** inside modules
- PatternReader persists learned data to `pattern_memory.json`

### 4. Timing Rules
- **TickTracker is the ONLY timing source**
- **NO module may use Wait(), Sleep(), or time-based delays**
- All timing must be tick-based via `GetCurrentTick()`
- Only exception: `Wait(15)` in main loop

### 5. Update Order
The main loop updates modules in this **exact order** (do not change):

1. TickTracker
2. FreezeDetection
3. MovementHelper
4. AttackStyleDetector
5. PrayerHelper
6. PatternReader
7. PathingPredictor
8. SplashPredictor
9. DesyncEngine
10. KOAnalyzer

Overlays are drawn after updates:
1. TileOverlay (background)
2. HUDOverlay
3. PrayerOverlay
4. KOOverlay
5. DebugOverlay (topmost)

## Module Interfaces

### Core Module Structure

Every `/core/` module follows this pattern:

```pascal
type
  T<ModuleName>State = record
    // State fields
  end;

var
  <ModuleName>State: T<ModuleName>State;

procedure Init<ModuleName>;    // Initialize module
procedure Update<ModuleName>;  // Update each frame
// Helper functions for external access
```

### Key Helper Functions

**TickTracker:**
- `GetCurrentTick(): Integer` - Current game tick
- `GetTickMs(): Integer` - Tick duration (600ms)
- `GetTickProgress(): Double` - Progress through current tick (0.0-1.0)

**FreezeDetection:**
- `IsPlayerFrozen(): Boolean` - Player freeze status
- `IsOpponentFrozen(): Boolean` - Opponent freeze status
- `GetPlayerFreezeTicksRemaining(): Integer` - Remaining freeze ticks

**MovementHelper:**
- `GetRecommendedTile(): TPoint` - Best tile to move to
- `IsSafeTile(Tile: TPoint): Boolean` - Tile safety check
- `GetPlayerPosition(): TPoint` - Current player position

**PrayerHelper:**
- `RecommendedPrayer(): TPrayerType` - Recommended protection
- `ShouldAutoSwap(): Boolean` - Whether to swap prayers
- `IsPrayerMismatch(): Boolean` - Wrong prayer active

**KOAnalyzer:**
- `IsKOWindowActive(): Boolean` - In KO danger window
- `GetKORiskScore(): Double` - Risk score (0.0-1.0)
- `ShouldEatNow(): Boolean` - Eating recommended

**DesyncEngine:**
- `GetTickOffset(): Integer` - Tick advantage (+) or disadvantage (-)
- `HasDesyncAdvantage(): Boolean` - Has tick advantage

**PatternReader:**
- `GetOpponentEatRhythm(): Integer` - Eating pattern (ticks)
- `GetOpponentPanicThreshold(): Integer` - Panic eat HP

### Overlay Structure

Every `/overlay/` module implements:

```pascal
procedure Draw<OverlayName>;  // Draw the overlay
```

## Visual Design

### Color Palette
- **Neon Cyan**: `#00FFFF` - Primary, tick indicators
- **Electric Purple**: `#A020F0` - Secondary, freeze effects
- **Danger Red**: `#FF4444` - Warnings, KO danger
- **Lime Green**: `#00FF00` - Safe, correct prayer
- **Soft White**: `#FFFFFF` - Text, neutral

### Overlay Layout
- **Top-left**: Tick counter, freeze status, desync indicator
- **Top-right**: KO windows, threat level, prayer status
- **Bottom-left**: Movement tiles, safe/danger zones
- **Bottom-right**: Pattern predictions
- **Center**: Tick pulse animation, KO flash, critical warnings

### Animation Guidelines
- Tick pulse: Subtle expansion on tick transition
- KO flash: Rapid red blink for critical threat
- Prayer mismatch: Pulsing wrong prayer indicator
- Tile highlights: Smooth fade in/out

## Configuration

Edit `/data/config.json` to customize:
- Overlay visibility and positions
- Threshold values for warnings
- Color scheme
- Debug settings
- Keybinds

## Usage

1. Open `main.simba` in Simba
2. Configure settings in `/data/config.json`
3. Run the script
4. Press configured hotkey to toggle overlay
5. Press STOP to terminate

## Extension Guide

### Adding a New Core Module

1. Create `core/NewModule.simba`
2. Define state record: `TNewModuleState`
3. Implement `InitNewModule` and `UpdateNewModule`
4. Add helper functions for external access
5. Include in `main.simba` (in correct dependency order)
6. Add to update sequence in `UpdateAllModules`

### Adding a New Overlay

1. Create `overlay/NewOverlay.simba`
2. Implement `DrawNewOverlay` procedure
3. Include in `main.simba`
4. Add to draw sequence in `DrawAllOverlays`
5. Only read state from core modules - never modify!

## Technical Notes

- **Tick Duration**: 600ms (OSRS game tick)
- **Frame Rate**: ~66 FPS (15ms delay)
- **Coordinate System**: OSRS tile coordinates
- **Color Format**: BGR (Simba standard)

## License

For personal use only. Do not distribute.

---

*OmniPK Engine - Tick-Accurate PvP Intelligence*
