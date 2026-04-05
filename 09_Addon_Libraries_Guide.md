# WoW Addon Libraries - Comprehensive Guide

## Table of Contents
1. [Overview](#overview)
2. [Native API Alternatives](#native-api-alternatives)
3. [LibStub - The Foundation](#libstub---the-foundation)
4. [Ace3 Library Suite](#ace3-library-suite)
5. [Data Broker System](#data-broker-system)
6. [Popular Utility Libraries](#popular-utility-libraries)
7. [UI Enhancement Libraries](#ui-enhancement-libraries)
8. [Specialized Libraries](#specialized-libraries)
9. [Housing Libraries](#housing-libraries)
10. [Embedding Libraries](#embedding-libraries)
11. [Library Compatibility Notes](#library-compatibility-notes)
12. [Library Best Practices](#library-best-practices)
13. [Finding and Using Libraries](#finding-and-using-libraries)

---

## Overview

WoW addon libraries are reusable code modules that provide common functionality. They allow developers to avoid reinventing the wheel and ensure compatibility between addons.

### Why Use Libraries?

**Benefits:**
- Save development time - Pre-built functionality
- Battle-tested code - Used by thousands of addons
- Community support - Well-documented and maintained
- Compatibility - Multiple addons can share same library
- Best practices - Written by experienced developers

**Common Use Cases:**
- Configuration GUI (AceConfig)
- Database management (AceDB)
- Data feeds (LibDataBroker)
- Minimap icons (LibDBIcon)
- Media resources (LibSharedMedia)
- Event handling (AceEvent)
- Localization (AceLocale)

**Important Note for 12.0.0 (Midnight):**
Many library functions now have native API equivalents added in patches 11.1.5 through 12.0.0. For new addons, prefer native APIs when available. Libraries remain valuable for backwards compatibility and advanced features not covered by native APIs.

---

## Native API Alternatives

**New in 11.1.5+ and 12.0.0** - Blizzard has added native APIs that replace common library functionality. For new addons, prefer these native alternatives.

### C_EncodingUtil vs LibSerialize/LibCompress/LibDeflate (11.1.5+)

Native compression and encoding is now available via `C_EncodingUtil`:

```lua
-- Native compression (prefer this for new addons)
local data = "This is a long string that needs compression..."

-- Compress string
local compressed = C_EncodingUtil.CompressString(data)

-- Encode to base64 for safe transmission/storage
local base64 = C_EncodingUtil.EncodeBase64(compressed)

-- Decode and decompress
local decoded = C_EncodingUtil.DecodeBase64(base64)
local original = C_EncodingUtil.DecompressString(decoded)

-- Native JSON serialization (prefer this for new addons)
local myTable = { name = "Test", value = 42, nested = { a = 1, b = 2 } }
local json = C_EncodingUtil.SerializeJSON(myTable)
local restored = C_EncodingUtil.DeserializeJSON(json)
```

**When to still use libraries:**
- LibDeflate/LibCompress: For specific compression algorithms (ZLIB, etc.)
- LibSerialize: For complex table serialization with metatables
- Cross-addon communication requiring specific formats
- Backwards compatibility with existing data formats

### Native Table Functions vs LibTableUtil (11.1.7+)

Native table utilities have been added to the `table` namespace:

```lua
-- Native table preallocation (11.1.7+)
-- Pre-allocate array with 100 slots for better performance
local t = table.create(100)

-- Pre-allocate with both array (100) and hash (50) slots
local t2 = table.create(100, 50)

-- Native table counting (11.2.5+)
-- Count all key-value pairs (not just array portion)
local myTable = { a = 1, b = 2, [1] = "x", [2] = "y" }
local count = table.count(myTable)  -- Returns 4

-- Compare with # operator (only counts array portion)
print(#myTable)  -- Returns 2 (only numeric keys from 1)
print(table.count(myTable))  -- Returns 4 (all keys)
```

**When to still use libraries:**
- Complex table operations (deep copy, merge, diff)
- Table pooling and recycling
- Backwards compatibility with older WoW versions

### Settings API vs AceConfig (11.0+)

The native Settings API is now the standard for addon configuration:

```lua
-- Native Settings API approach
local category = Settings.RegisterVerticalLayoutCategory("MyAddon")

local setting = Settings.RegisterAddOnSetting(category,
    "myAddonEnabled",
    "enabled",
    MyAddonDB,
    Settings.VarType.Boolean,
    "Enable Addon",
    true
)

Settings.CreateCheckbox(category, setting, "Enable or disable the addon")
Settings.RegisterAddOnCategory(category)

-- Open settings
Settings.OpenToCategory(category.ID)
```

**When to still use AceConfig:**
- Complex nested option tables
- Dynamic option generation
- Profile management integration with AceDB
- Existing addons with extensive AceConfig usage
- Rapid prototyping (AceConfig's declarative syntax is faster to write)

### EventRegistry vs AceEvent/CallbackHandler (11.0+)

The native EventRegistry system provides modern callback handling:

```lua
-- Native EventRegistry for frame events
local function OnPlayerLogin()
    print("Player logged in!")
end

EventRegistry:RegisterCallback("PLAYER_LOGIN", OnPlayerLogin, owner)
EventRegistry:UnregisterCallback("PLAYER_LOGIN", owner)

-- For custom addon events, EventRegistry also works
EventRegistry:RegisterCallback("MyAddon.SomeEvent", handler, owner)
EventRegistry:TriggerEvent("MyAddon.SomeEvent", arg1, arg2)
```

**When to still use libraries:**
- AceEvent: For its embedding pattern and integration with AceAddon
- CallbackHandler: For library development requiring callback systems
- Backwards compatibility

### Summary: Native vs Library

| Task | Native API (12.0.0) | Library Alternative | Recommendation |
|------|---------------------|---------------------|----------------|
| Compression | C_EncodingUtil.CompressString | LibDeflate, LibCompress | Native for new code |
| Base64 | C_EncodingUtil.EncodeBase64 | LibBase64 | Native for new code |
| JSON | C_EncodingUtil.SerializeJSON | LibJSON | Native for new code |
| Table prealloc | table.create() | LibTableUtil | Native for new code |
| Table count | table.count() | LibTableUtil | Native for new code |
| Settings UI | Settings API | AceConfig | Native preferred, Ace for complex UIs |
| Events | EventRegistry | AceEvent | Either works well |

---

## LibStub - The Foundation

**The most fundamental library in WoW addon development.**

### What is LibStub?

LibStub is a minimalist library loader that allows multiple addons to share the same library without version conflicts.

**Key Features:**
- Version management (only newest version loads)
- Global library registry
- Embedding support
- Tiny footprint (~100 lines)

### LibStub Source

**File:** `LibStub.lua` (found in nearly every addon's Libs folder)

```lua
-- LibStub.lua
LibStub = LibStub or {}
local LIBSTUB_MAJOR, LIBSTUB_MINOR = "LibStub", 4

function LibStub:NewLibrary(major, minor)
    assert(type(major) == "string", "Bad argument #2 to `NewLibrary' (string expected)")
    minor = assert(tonumber(minor), "Bad argument #2 to `NewLibrary' (number expected)")

    local oldminor = self.minors[major]
    if oldminor and oldminor >= minor then
        return nil  -- Older version, don't load
    end

    self.minors[major] = minor
    self.libs[major] = self.libs[major] or {}

    return self.libs[major]
end

function LibStub:GetLibrary(major, silent)
    if not self.libs[major] and not silent then
        error(("Cannot find a library instance of %q."):format(tostring(major)), 2)
    end
    return self.libs[major], self.minors[major]
end
```

### Using LibStub

**Loading a library:**
```lua
-- Get library (error if not found)
local AceAddon = LibStub("AceAddon-3.0")

-- Get library (silent, returns nil if not found)
local AceAddon = LibStub("AceAddon-3.0", true)

-- Check if library exists
if LibStub:GetLibrary("LibDataBroker-1.1", true) then
    -- Library is available
end
```

**Creating a library:**
```lua
local MAJOR, MINOR = "MyLib-1.0", 1
local lib = LibStub:NewLibrary(MAJOR, MINOR)
if not lib then return end  -- Older version already loaded

-- Define library functions
function lib:DoSomething()
    print("Library function!")
end
```

---

## Ace3 Library Suite

**The most popular addon framework, used by thousands of addons.**

> **Full Ace3 documentation has moved to [09a_Ace3_Library_Guide.md](09a_Ace3_Library_Guide.md).** That guide covers all 14+ Ace3 libraries in detail with complete API references, code examples, a full addon example, troubleshooting, and a comparison table. The summary below is kept for quick reference only.

### Ace3 Overview

Ace3 is a collection of embeddable libraries that handle common addon tasks: lifecycle management, saved variables with profiles, event handling, slash commands, timers, hooking, configuration GUIs, communication, and localization.

**Repository:** [WoWAce Ace3](https://www.wowace.com/projects/ace3)
**Full Guide:** See [09a_Ace3_Library_Guide.md](09a_Ace3_Library_Guide.md) for comprehensive documentation.

**Core libraries:** AceAddon-3.0, AceDB-3.0, AceDBOptions-3.0, AceEvent-3.0, AceConsole-3.0, AceTimer-3.0, AceHook-3.0, AceBucket-3.0, AceComm-3.0, AceSerializer-3.0, AceConfig-3.0, AceConfigDialog-3.0, AceGUI-3.0, AceLocale-3.0

---

## Data Broker System

### LibDataBroker-1.1

**Standard for creating data feeds and displays.**

LibDataBroker allows addons to publish data that can be displayed by various display addons (like Bazooka, ChocolateBar, Titan Panel).

#### Creating a Data Source

```lua
local LDB = LibStub("LibDataBroker-1.1")

-- Create data object
local dataObj = LDB:NewDataObject("MyAddon", {
    type = "data source",
    text = "N/A",
    icon = "Interface\\Icons\\INV_Misc_QuestionMark",
    label = "My Addon",

    OnClick = function(frame, button)
        if button == "LeftButton" then
            print("Left clicked!")
        elseif button == "RightButton" then
            print("Right clicked!")
        end
    end,

    OnTooltipShow = function(tooltip)
        tooltip:AddLine("My Addon")
        tooltip:AddLine("Click for more info")
    end,
})

-- Update data
function UpdateData()
    dataObj.text = format("%d gold", GetMoney() / 10000)
end
```

#### Data Object Types

**data source:**
- Displays text/icon in broker bar
- Used for information display

**launcher:**
- Button to open windows/perform actions
- Used for quick access

#### Consuming Data Sources

```lua
local LDB = LibStub("LibDataBroker-1.1")

-- Get all data objects
for name, obj in LDB:DataObjectIterator() do
    print(name, obj.type, obj.text)
end

-- Register callback for new objects
LDB.RegisterCallback(addon, "LibDataBroker_DataObjectCreated", function(event, name, dataobj)
    print("New data object:", name)
end)

-- Attribute changed callback
LDB.RegisterCallback(addon, "LibDataBroker_AttributeChanged", function(event, name, attr, value)
    print("Data object", name, "changed", attr, "to", value)
end)
```

### LibDBIcon-1.0

**Minimap icon library (uses LibDataBroker).**

```lua
local LDB = LibStub("LibDataBroker-1.1")
local LDBIcon = LibStub("LibDBIcon-1.0")

-- Create data broker object
local dataObj = LDB:NewDataObject("MyAddon", {
    type = "launcher",
    icon = "Interface\\Icons\\INV_Misc_QuestionMark",
    OnClick = function(frame, button)
        if button == "LeftButton" then
            MyAddon:ToggleFrame()
        end
    end,
})

-- Register minimap icon
LDBIcon:Register("MyAddon", dataObj, self.db.profile.minimap)

-- Hide/show minimap icon
LDBIcon:Hide("MyAddon")
LDBIcon:Show("MyAddon")

-- Check if shown
if LDBIcon:IsRegistered("MyAddon") then
    print("Icon is registered")
end
```

**In saved variables:**
```lua
local defaults = {
    profile = {
        minimap = {
            hide = false,
            minimapPos = 220,
            radius = 80,
        },
    },
}
```

---

## Popular Utility Libraries

### LibSharedMedia-3.0

**Shared font, sound, and texture resources.**

```lua
local LSM = LibStub("LibSharedMedia-3.0")

-- Get media
local font = LSM:Fetch("font", "Friz Quadrata TT")
local sound = LSM:Fetch("sound", "Auction House Open")
local statusbar = LSM:Fetch("statusbar", "Blizzard")
local background = LSM:Fetch("background", "Blizzard Dialog Background")
local border = LSM:Fetch("border", "Blizzard Tooltip")

-- List available media
local fonts = LSM:List("font")
for i, fontName in ipairs(fonts) do
    print(fontName)
end

-- Register your own media
LSM:Register("font", "My Custom Font", [[Interface\AddOns\MyAddon\Fonts\MyFont.ttf]])
LSM:Register("statusbar", "My Texture", [[Interface\AddOns\MyAddon\Textures\StatusBar]])

-- Callback when media added
LSM.RegisterCallback(addon, "LibSharedMedia_Registered", function(event, mediaType, key)
    print("New media registered:", mediaType, key)
end)
```

**Media Types:**
- `font` - Font files (.ttf)
- `sound` - Sound files
- `statusbar` - StatusBar textures
- `background` - Background textures
- `border` - Border textures

### LibRangeCheck-2.0 / 3.0

**Accurate range checking.**

```lua
local LRC = LibStub("LibRangeCheck-3.0")

-- Get range to unit
local minRange, maxRange = LRC:GetRange("target")

if maxRange then
    if maxRange < 5 then
        print("Melee range")
    elseif maxRange < 30 then
        print("Medium range")
    else
        print("Long range")
    end
end

-- Check specific range
local inRange = LRC:GetRange("target", 40)  -- Within 40 yards?
```

### LibDeflate

**Compression library.**

**Note (11.1.5+):** For new addons, consider using native `C_EncodingUtil.CompressString()` and `C_EncodingUtil.EncodeBase64()` instead. See Native API Alternatives section.

```lua
local LibDeflate = LibStub:GetLibrary("LibDeflate")

-- Compress data
local original = "This is a long string to compress..."
local compressed = LibDeflate:CompressDeflate(original)
local encoded = LibDeflate:EncodeForPrint(compressed)

-- Decompress data
local decoded = LibDeflate:DecodeForPrint(encoded)
local decompressed = LibDeflate:DecompressDeflate(decoded)

print(original == decompressed)  -- true
```

**Use cases:**
- Sharing addon profiles
- Compressing saved data
- Import/export strings
- Backwards compatibility with existing LibDeflate-encoded data

---

## UI Enhancement Libraries

### LibDialog-1.0

**Better popup dialogs than StaticPopup.**

```lua
local LibDialog = LibStub("LibDialog-1.0")

-- Create dialog
LibDialog:Register("MYADDON_CONFIRM", {
    text = "Are you sure you want to reset?",
    buttons = {
        {
            text = "Yes",
            on_click = function()
                MyAddon:ResetDatabase()
            end,
        },
        {
            text = "No",
        },
    },
})

-- Show dialog
LibDialog:Spawn("MYADDON_CONFIRM")
```

### LibWindow-1.1

**Save window positions.**

```lua
local LibWindow = LibStub("LibWindow-1.1")

-- Save window position
LibWindow.SavePosition(frame)

-- Restore window position
LibWindow.RestorePosition(frame)

-- Make window movable
LibWindow.MakeDraggable(frame)

-- Register for automatic save/restore
LibWindow.RegisterConfig(frame, self.db.profile.windowPos)
```

---

## Specialized Libraries

### LibGroupInSpecT-1.1

**Inspect group member specs.**

```lua
local LGIST = LibStub("LibGroupInSpecT-1.1")

-- Get unit spec
LGIST.RegisterCallback(self, "GroupInSpecT_Update", function(event, guid, unit, info)
    if info then
        local specID = info.global_spec_id
        local specName = info.spec_name_localized
        print(unit, "is", specName)
    end
end)

-- Get current data
local data = LGIST:GetCachedInfo(guid)
```

### LibClassicDurations

**Aura duration tracking (Classic).**

### LibCustomGlow

**Glow effects for action buttons.**

```lua
local LCG = LibStub("LibCustomGlow-1.0")

-- Add glow
LCG.ButtonGlow_Start(button)
LCG.PixelGlow_Start(button)
LCG.AutoCastGlow_Start(button)

-- Remove glow
LCG.ButtonGlow_Stop(button)
LCG.PixelGlow_Stop(button)
LCG.AutoCastGlow_Stop(button)
```

**Note (12.0.0):** LibCustomGlow may need updates due to action bar API changes. Check for updated versions.

---

## Housing Libraries

**New in 12.0.0 (Midnight)** - Player housing introduces extensive APIs but community libraries are still emerging.

### Current Status

No standard housing libraries exist yet, but common patterns are emerging in the community:

```lua
-- Housing API provides extensive functionality via C_Housing
-- Libraries will likely wrap these patterns:

-- Example: Housing data management pattern
local HousingHelper = {}

function HousingHelper:GetPlacedFurniture()
    local furniture = {}
    local plotInfo = C_Housing.GetCurrentPlotInfo()
    if plotInfo then
        -- Gather placed furniture data
        for i, item in ipairs(C_Housing.GetPlacedFurniture()) do
            table.insert(furniture, {
                id = item.furnitureID,
                position = item.position,
                rotation = item.rotation,
            })
        end
    end
    return furniture
end

function HousingHelper:SaveLayout(name)
    local layout = self:GetPlacedFurniture()
    -- Save to addon database
    return layout
end
```

### Expected Library Patterns

As housing matures, expect libraries for:

- **Layout Management** - Save/restore furniture arrangements
- **Furniture Cataloging** - Track owned furniture across characters
- **Positioning Helpers** - Snap-to-grid, alignment tools
- **Housing Events** - Simplified callbacks for housing state changes

### Using Native C_Housing API

For now, use the native C_Housing API directly:

```lua
-- Check if player has housing unlocked
if C_Housing.IsHousingUnlocked() then
    -- Get current plot info
    local plotInfo = C_Housing.GetCurrentPlotInfo()

    -- Enter housing mode
    C_Housing.EnterHousingMode()

    -- Place furniture (when in housing mode)
    C_Housing.PlaceFurniture(furnitureID, position, rotation)
end

-- Register for housing events
local frame = CreateFrame("Frame")
frame:RegisterEvent("HOUSING_FURNITURE_PLACED")
frame:RegisterEvent("HOUSING_FURNITURE_REMOVED")
frame:RegisterEvent("HOUSING_MODE_ENTERED")
frame:RegisterEvent("HOUSING_MODE_EXITED")
```

### Recommendations

1. **Build on native APIs** - C_Housing is comprehensive
2. **Watch community repos** - Libraries will emerge on WoWAce/GitHub
3. **Consider contributing** - Help establish standard patterns
4. **Use AceDB for storage** - Profile-based layout storage works well

---

## Embedding Libraries

### How Embedding Works

Libraries can be "embedded" into your addon, copying their functions into your addon table.

**Without embedding:**
```lua
local AceEvent = LibStub("AceEvent-3.0")
AceEvent:RegisterEvent(MyAddon, "PLAYER_LOGIN", function() ... end)
```

**With embedding:**
```lua
local MyAddon = LibStub("AceAddon-3.0"):NewAddon("MyAddon", "AceEvent-3.0")
MyAddon:RegisterEvent("PLAYER_LOGIN")  -- Method is now part of MyAddon
```

### Manual Embedding

```lua
local lib = LibStub("SomeLibrary-1.0")
lib:Embed(MyAddon)

-- Now MyAddon has all of lib's methods
MyAddon:SomeLibraryMethod()
```

---

## Library Compatibility Notes

**12.0.0 (Midnight) Compatibility Status**

### Libraries Requiring Updates

#### Ace3 Suite
- **Status:** Likely needs updates for Settings API changes
- **Impact:** AceConfigDialog integration with Blizzard options panel
- **Workaround:** Check for 12.0.0-compatible releases on WoWAce
- **Note:** Core functionality (AceAddon, AceDB, AceEvent) should work

```lua
-- Check Ace3 version compatibility
local AceAddon = LibStub("AceAddon-3.0", true)
if AceAddon then
    local _, minor = LibStub:GetLibrary("AceAddon-3.0")
    print("AceAddon version:", minor)
end
```

#### Action Bar Libraries
- **Status:** MAJOR updates required
- **Impact:** Global action bar functions removed in 12.0.0
- **Affected:** LibActionButton, Bartender4 internals, custom action bar code
- **Details:** Functions like `ActionButton_Update()` no longer exist as globals

```lua
-- OLD (broken in 12.0.0):
ActionButton_Update(button)

-- NEW: Use mixin methods on button frames
button:Update()
-- Or hook the mixin directly
```

#### Transmog/Appearance Libraries
- **Status:** Complete rewrites needed
- **Impact:** Wardrobe API overhauled
- **Affected:** LibWardrobe, transmog collection addons
- **Details:** Many C_TransmogCollection functions changed or removed

#### LibCustomGlow
- **Status:** May need updates
- **Impact:** Action button glow effects
- **Workaround:** Test thoroughly; may work with minor fixes

### Libraries Expected to Work

#### LibStub
- **Status:** Works (foundation library, minimal API surface)
- **Note:** Always use latest version

#### LibDataBroker-1.1
- **Status:** Works (self-contained callback system)
- **Note:** No WoW API dependencies

#### LibDBIcon-1.0
- **Status:** Should work
- **Note:** Verify with new addon list changes in 12.0.0
- **Test:** Check minimap icon visibility and positioning

#### CallbackHandler-1.0
- **Status:** Works
- **Note:** Consider EventRegistry for new code, but CallbackHandler remains functional

#### LibSharedMedia-3.0
- **Status:** Works (media registration unchanged)
- **Note:** Standard fonts/textures still function

#### LibRangeCheck-3.0
- **Status:** Likely works
- **Note:** Range-checking APIs stable

### Secret Values Consideration

**12.0.0 introduces secret values for sensitive APIs.** This doesn't directly affect libraries but impacts addon code using libraries:

```lua
-- Libraries that wrap secure functions may need updates
-- Example: If a library caches spell IDs from GetSpellInfo
-- Some spell data may now require secret value handling

-- Your addon code should handle this:
local function SafeGetSpellName(spellID)
    local name = C_Spell.GetSpellName(spellID)
    -- name might be a secret value in some contexts
    return name
end
```

### Version Checking Pattern

```lua
-- Robust library loading with version check
local function LoadLibrarySafely(name, minVersion)
    local lib, version = LibStub:GetLibrary(name, true)
    if not lib then
        print("Library not found:", name)
        return nil
    end
    if minVersion and version < minVersion then
        print("Library too old:", name, "need", minVersion, "have", version)
        return nil
    end
    return lib
end

-- Usage
local AceDB = LoadLibrarySafely("AceDB-3.0", 28)
if AceDB then
    -- Safe to use
end
```

### Migration Recommendations

1. **Test libraries in 12.0.0 PTR** before launch
2. **Check WoWAce/GitHub** for updated library versions
3. **Have fallbacks** for critical functionality
4. **Consider native APIs** for new features (see Native API Alternatives section)
5. **Report issues** to library maintainers with specific error messages

---

## Library Best Practices

### 1. Version Management

**Always specify library version:**
```lua
local lib = LibStub("MyLib-1.0", true)  -- Silent if not found
if not lib then
    -- Library not available, provide fallback
    return
end
```

### 2. Embedding in TOC

**List libraries before your code:**
```
## Interface: 120000
## Title: My Addon

# Libraries
Libs\LibStub\LibStub.lua
Libs\AceAddon-3.0\AceAddon-3.0.xml
Libs\AceDB-3.0\AceDB-3.0.xml
Libs\AceConfig-3.0\AceConfig-3.0.xml

# Your code
Core.lua
Config.lua
```

### 3. Don't Modify Libraries

**Never edit library files!**
- Update to newer versions instead
- Report bugs to library maintainers
- Use hooks if you need to modify behavior

### 4. Use .pkgmeta for Distribution

**Let CurseForge/Wago handle libraries:**
```yaml
externals:
    Libs/LibStub:
        url: https://repos.curseforge.com/wow/libstub/trunk
    Libs/AceAddon-3.0:
        url: https://repos.curseforge.com/wow/ace3/trunk/AceAddon-3.0
```

---

## Finding and Using Libraries

### Where to Find Libraries

**WoWAce:**
- https://www.wowace.com/
- Primary repository for WoW libraries

**CurseForge:**
- https://www.curseforge.com/wow/addons
- Search for "lib" or "library"

**GitHub:**
- Many libraries are on GitHub
- Search for "wow-addon-library"

### Common Library Locations in Addons

```
<WoW Install>\Interface\AddOns\
├── Ace3\                           # Full Ace3 suite
├── SomeAddon\
│   └── Libs\
│       ├── LibStub\
│       ├── AceAddon-3.0\
│       ├── AceDB-3.0\
│       ├── LibDataBroker-1.1\
│       └── LibDBIcon-1.0\
```

### Extracting Libraries from Addons

You can copy library folders from other addons, but it's better to:
1. Download from official source (WoWAce, CurseForge)
2. Use package manager (.pkgmeta)
3. Use git submodules if hosting on GitHub

---

## Quick Reference

### Most Essential Libraries

<!-- CLAUDE_SKIP_START -->

| Library | Purpose | When to Use |
|---------|---------|-------------|
| **LibStub** | Library loader | Always (foundation) |
| **AceAddon-3.0** | Addon framework | Most addons |
| **AceDB-3.0** | Database + profiles | Need saved variables |
| **AceConfig-3.0** | Options GUI | Need config panel |
| **AceEvent-3.0** | Event handling | Simplify events |
| **LibDataBroker-1.1** | Data feeds | Display data in bars |
| **LibDBIcon-1.0** | Minimap icons | Want minimap button |
| **LibSharedMedia-3.0** | Fonts/textures | Custom media |

<!-- CLAUDE_SKIP_END -->

### Library Loading Order

**In TOC file, always load:**

1. LibStub first
2. Other libraries next
3. Your code last

```
Libs\LibStub\LibStub.lua
Libs\CallbackHandler-1.0\CallbackHandler-1.0.xml
Libs\AceAddon-3.0\AceAddon-3.0.xml
Libs\AceDB-3.0\AceDB-3.0.xml
Core.lua
```

<!-- CLAUDE_SKIP_START -->

---

## Summary

### Key Points

1. **LibStub** is the foundation - all libraries use it
2. **Ace3** is the most popular framework - great for beginners
3. **LibDataBroker** is standard for data display - use for broker integration
4. **Embed libraries** to make code cleaner
5. **Use .pkgmeta** for automatic library management
6. **Never modify** library files - update or report bugs instead
7. **Load in correct order** - LibStub then libs then your code
8. **Check versions** - use LibStub:GetLibrary() safely
9. **Consider native APIs** - C_EncodingUtil, table.create(), Settings API
10. **Check 12.0.0 compatibility** - Test libraries before Midnight launch

---

<!-- CLAUDE_SKIP_END -->
