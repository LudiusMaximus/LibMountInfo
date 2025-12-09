# LibMountInfo

A World of Warcraft addon library for tracking mount information and detecting flying mounts.

## Overview

LibMountInfo provides a centralized solution for mount tracking across multiple addons. It eliminates duplicate code by maintaining a single source of truth for:
- Flying mount detection
- Last used mount tracking (flying and non-flying)
- Current mount information
- Mount change callbacks

## Features

- **Flying Mount Detection**: Automatically maintains a list of all flying mounts
- **Auto-Update**: Automatically refreshes flying mount list when new mounts are learned
- **Efficient Caching**: Tracks last used mounts for quick lookups
- **Persistent Storage**: Saves last flying and non-flying mounts per character across sessions
- **Skyriding Detection**: Detect Dynamic Flight vs Steady Flight mode
- **Event Callbacks**: Register callbacks for mount changes
- **API Compatibility**: Clean API that's easy to integrate

## API Reference

### Core Functions

#### `LibMountInfo:GetCurrentMount()`
Returns the currently active mount ID and whether it's a flying mount.

**Returns:**
- `mountID` (number|nil): The active mount's ID, or nil if not mounted
- `isFlying` (boolean): True if the mount can fly

```lua
local mountID, isFlying = LibMountInfo:GetCurrentMount()
if mountID then
    print("Currently mounted on:", mountID)
    print("Can fly:", isFlying)
end
```

#### `LibMountInfo:GetLastFlyingMount()`
Returns the last used flying mount ID.

**Returns:**
- `mountID` (number|nil): The last flying mount ID, or nil if none

```lua
local lastFlyingMount = LibMountInfo:GetLastFlyingMount()
```

#### `LibMountInfo:GetLastNonFlyingMount()`
Returns the last used non-flying mount ID.

**Returns:**
- `mountID` (number|nil): The last non-flying mount ID, or nil if none used yet

```lua
local lastNonFlyingMount = LibMountInfo:GetLastNonFlyingMount()
```

#### `LibMountInfo:GetLastMount()`
Returns the most recently used mount ID, regardless of whether it was flying or non-flying.

**Returns:**
- `mountID` (number|nil): The most recently used mount ID, or nil if none

```lua
local lastMount = LibMountInfo:GetLastMount()
if lastMount then
    C_MountJournal.SummonByID(lastMount)
end
```

#### `LibMountInfo:IsFlyingMount(mountID)`
Check if a specific mount ID is a flying mount.

**Parameters:**
- `mountID` (number): The mount ID to check

**Returns:**
- `isFlying` (boolean): True if the mount can fly

```lua
if LibMountInfo:IsFlyingMount(1234) then
    print("Mount 1234 can fly!")
end
```

#### `LibMountInfo:CurrentMountCanFly()`
Quick check if the current mount can fly.

**Returns:**
- `canFly` (boolean): True if currently on a flying mount

```lua
if LibMountInfo:CurrentMountCanFly() then
    print("I'm on a flying mount!")
end
```

#### `LibMountInfo:IsSkyriding()`
Check if currently using Skyriding (Dynamic Flight) mode vs Steady Flight.

**Returns:**
- `isSkyriding` (boolean): True if using Skyriding, false if Steady Flight or not on flying mount

```lua
if LibMountInfo:IsSkyriding() then
    print("Using Dynamic Flight!")
else
    print("Using Steady Flight or not flying")
end
```

### Advanced Functions

#### `LibMountInfo:UpdateFlyingMounts()`
Manually refresh the flying mount list. This is automatically called on `PLAYER_LOGIN` and `NEW_MOUNT_ADDED`, but can be called manually if needed.

```lua
LibMountInfo:UpdateFlyingMounts()
```

#### `LibMountInfo:SetPersistentStorage(savedVarTable)`
Connect the library to your addon's saved variables for cross-session persistence.

**Parameters:**
- `savedVarTable` (table): A saved variable with structure `[realmName][playerName] = { flying = mountID, notFlying = mountID, last = mountID }`

The library stores your last flying mount, last non-flying mount, and most recently used mount. It will automatically migrate old format (single mountID) to the new format.

```lua
-- In your addon's PLAYER_LOGIN handler
MyAddon_LastMount = MyAddon_LastMount or {}
LibMountInfo:SetPersistentStorage(MyAddon_LastMount)
```

#### `LibMountInfo:RegisterCallback(callbackType, identifier, callback)`
Register a callback for mount events.

**Parameters:**
- `callbackType` (string): "onMountChanged" or "onFlyingMountsUpdated"
- `identifier` (string): Unique identifier for this callback
- `callback` (function): Function to call when event occurs

**Callback Signatures:**
- `onMountChanged(mountID, isFlying)`: Called when mount changes
- `onFlyingMountsUpdated()`: Called when flying mount list is updated

```lua
LibMountInfo:RegisterCallback("onMountChanged", "MyAddon", function(mountID, isFlying)
    print("Mount changed to:", mountID, "Flying:", isFlying)
end)
```

#### `LibMountInfo:UnregisterCallback(callbackType, identifier)`
Unregister a previously registered callback.

**Parameters:**
- `callbackType` (string): "onMountChanged" or "onFlyingMountsUpdated"
- `identifier` (string): The identifier used when registering

```lua
LibMountInfo:UnregisterCallback("onMountChanged", "MyAddon")
```

#### `LibMountInfo:SetIgnoredMounts(mountIDs)`
Set which mounts should be ignored for "last mount" tracking. Useful for utility mounts like Yak or Brutosaur that you use temporarily but don't want to summon with your hotkey.

**Parameters:**
- `mountIDs` (string|table): Either a comma-separated string of mount IDs (e.g., "460, 284") or a table of mount ID numbers

```lua
-- Using comma-separated string
LibMountInfo:SetIgnoredMounts("460, 284, 1817")

-- Using table
LibMountInfo:SetIgnoredMounts({460, 284, 1817})
```

#### `LibMountInfo:GetIgnoredMounts()`
Get the current list of ignored mount IDs.

**Returns:**
- `mountIDs` (table): Array of mount IDs that are being ignored

```lua
local ignored = LibMountInfo:GetIgnoredMounts()
for _, mountID in ipairs(ignored) do
    print("Ignoring mount:", mountID)
end
```

## Integration Examples

### Basic Usage with Persistent Storage
```lua
local LibMountInfo = LibStub("LibMountInfo-1.0")

-- Set up persistent storage on login
local f = CreateFrame("Frame")
f:RegisterEvent("PLAYER_LOGIN")
f:SetScript("OnEvent", function()
    MyAddon_LastMount = MyAddon_LastMount or {}
    LibMountInfo:SetPersistentStorage(MyAddon_LastMount)
end)

-- Library now automatically tracks and persists your last flying mount!
```

### Race on Last Mount
```lua
local LibMountInfo = LibStub("LibMountInfo-1.0")

local function CheckRaceMount()
    local currentMount, isFlying = LibMountInfo:GetCurrentMount()
    local lastFlyingMount = LibMountInfo:GetLastFlyingMount()
    
    if isFlying and lastFlyingMount and currentMount ~= lastFlyingMount then
        -- Switch back to preferred mount
        C_MountJournal.SummonByID(lastFlyingMount)
    end
end
```

### DynamicCam Integration
```lua
local LibMountInfo = LibStub("LibMountInfo-1.0")

function MyAddon:CurrentMountCanFly()
    return LibMountInfo:CurrentMountCanFly()
end
```

## Installation

1. Copy the `LibMountInfo` folder to your addon's `Libs` directory
2. Add to your `embeds.xml`:
```xml
<Script file="Libs\LibMountInfo\LibMountInfo.lua"/>
```
3. Add to your TOC SavedVariables if you want to persist mount data across sessions

## Benefits

- **Performance**: Single mount list maintained instead of multiple per-addon copies
- **Consistency**: All addons see the same mount information
- **Maintainability**: Update mount detection logic in one place
- **Memory Efficient**: Shared data structure across all addons

## Used By

- **LudiusPlus**: DismountToggle and RaceOnLastMount modules
- **DynamicCam**: Situation condition checks for flying mounts

## Technical Notes

The library uses the Mount Journal API to determine which mounts can fly by filtering the journal to show only flying-capable mounts. This is more reliable than checking mount types directly, as Blizzard's mount system has some quirks (e.g., mounts that can fly in some zones but not others).

The flying mount list is generated at login and automatically refreshed when new mounts are learned via the `NEW_MOUNT_ADDED` event. You can also manually refresh it by calling `UpdateFlyingMounts()`.

### Persistent Storage

When `SetPersistentStorage()` is used, the library automatically saves your last flying mount, last non-flying mount, and last used mount (either flying or non-flying) to the provided saved variable table. This allows your mount preferences to be remembered across sessions and reloads.

The library includes automatic migration from the old format (single mountID) to the new format (table with flying/notFlying/last keys), so existing users won't lose their data.

The storage structure is:

```lua
savedVarTable[realmName][playerName] = {
  flying = lastFlyingMountID,
  notFlying = lastNonFlyingMountID,
  last = lastEitherMountID
}
```

The `last` field always contains the most recently used mount, regardless of whether it was flying or non-flying. This is useful for hotkeys that should summon whatever mount you used last.

## License

MIT License - See individual addon licenses for details

## Author

LudiusMaximus