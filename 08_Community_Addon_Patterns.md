# Community Addon Development Patterns

## Table of Contents
1. [Popular Addon Frameworks](#popular-addon-frameworks)
2. [Common Addon Structures](#common-addon-structures)
3. [Configuration and Options](#configuration-and-options)
4. [Slash Commands](#slash-commands)
5. [LibStub and Library Management](#libstub-and-library-management)
6. [Localization Patterns](#localization-patterns)
7. [Profile Systems](#profile-systems)
8. [Update and Notification Systems](#update-and-notification-systems)
9. [Performance Best Practices](#performance-best-practices)
10. [Distribution and Packaging](#distribution-and-packaging)
11. [Addon Categories and 12.0 Compatibility](#addon-categories-and-120-compatibility)
12. [Library Updates for 12.0](#library-updates-for-120)

---

## Popular Addon Frameworks

### Ace3 Framework

**Most popular addon development framework in the WoW community.**

**Core Libraries:**
- **AceAddon-3.0** - Addon management
- **AceConsole-3.0** - Slash commands
- **AceConfig-3.0** - Configuration GUI
- **AceDB-3.0** - Database management with profiles
- **AceEvent-3.0** - Event handling
- **AceHook-3.0** - Function hooking
- **AceGUI-3.0** - GUI widgets
- **AceLocale-3.0** - Localization

> **Note:** The following uses Ace3 libraries (AceAddon, AceEvent, AceConsole, AceDB). These are third-party community libraries, not part of the base WoW addon API. For base API equivalents, see the Single-File Addon pattern below or [04_Addon_Structure.md](04_Addon_Structure.md). For detailed Ace3 documentation, see [09a_Ace3_Library_Guide.md](09a_Ace3_Library_Guide.md).

**Basic Ace3 Addon Structure:**
```lua
-- MyAddon.lua
local AddonName = "MyAddon"
local MyAddon = LibStub("AceAddon-3.0"):NewAddon(AddonName, "AceConsole-3.0", "AceEvent-3.0")

-- Defaults
local defaults = {
    profile = {
        enabled = true,
        message = "Hello, World!",
    },
}

function MyAddon:OnInitialize()
    -- Database
    self.db = LibStub("AceDB-3.0"):New("MyAddonDB", defaults, true)

    -- Register slash command
    self:RegisterChatCommand("myaddon", "ChatCommand")
end

function MyAddon:OnEnable()
    -- Register events
    self:RegisterEvent("PLAYER_LOGIN")
end

function MyAddon:OnDisable()
    -- Cleanup
end

function MyAddon:PLAYER_LOGIN()
    self:Print("MyAddon loaded!")
end

function MyAddon:ChatCommand(input)
    if not input or input:trim() == "" then
        self:Print("Usage: /myaddon <command>")
    else
        self:Print("You said:", input)
    end
end
```

---

## Common Addon Structures

### Module-Based Architecture

**Pattern used by large addons like WeakAuras, DBM, BigWigs:**

```
MyAddon/
├── MyAddon.toc
├── Core.lua              # Main addon initialization
├── Config.lua            # Configuration/defaults
├── Options.lua           # Options GUI
├── Locales/
│   ├── enUS.lua
│   ├── deDE.lua
│   └── frFR.lua
├── Modules/
│   ├── Module1.lua       # Feature module 1
│   ├── Module2.lua       # Feature module 2
│   └── Module3.lua       # Feature module 3
├── UI/
│   ├── MainFrame.xml
│   ├── MainFrame.lua
│   └── Templates.xml
└── Libs/
    ├── LibStub/
    ├── AceAddon-3.0/
    └── AceDB-3.0/
```

**Module Pattern (Base WoW API):**
```lua
-- Core.lua
local ADDON_NAME, ns = ...
ns.modules = {}

function ns:RegisterModule(name, module)
    self.modules[name] = module
end

function ns:InitModules()
    for name, mod in pairs(self.modules) do
        if mod.Init then mod:Init() end
    end
end

-- Module1.lua
local ADDON_NAME, ns = ...
local Module1 = {}
ns:RegisterModule("Module1", Module1)

function Module1:Init()
    print("Module1 initialized")
end
```

> **Library Alternative:** Ace3 provides a built-in module system via `AceAddon:NewModule()`. See [09a_Ace3_Library_Guide.md](09a_Ace3_Library_Guide.md).

### Single-File Addon

**Simple addons can be just one file:**

```
MySimpleAddon/
├── MySimpleAddon.toc
└── MySimpleAddon.lua
```

**MySimpleAddon.toc:**
```
## Interface: 120000
## Title: My Simple Addon
## Author: Your Name
## Version: 1.0.0
## SavedVariables: MySimpleAddonDB

MySimpleAddon.lua
```

**MySimpleAddon.lua:**
```lua
local AddonName = "MySimpleAddon"
local DB

-- Initialize
local frame = CreateFrame("Frame")
frame:RegisterEvent("ADDON_LOADED")
frame:RegisterEvent("PLAYER_LOGIN")

frame:SetScript("OnEvent", function(self, event, ...)
    if event == "ADDON_LOADED" then
        local addonName = ...
        if addonName == AddonName then
            DB = MySimpleAddonDB or {}
            MySimpleAddonDB = DB
        end
    elseif event == "PLAYER_LOGIN" then
        print(AddonName, "loaded!")
    end
end)
```

---

## Configuration and Options

### Blizzard Settings Panel (11.0+)

**Modern WoW settings integration:**

```lua
local category = Settings.RegisterVerticalLayoutCategory("MyAddon")

-- Boolean setting
local variable = "MyAddon_Enabled"
local name = "Enable MyAddon"
local tooltip = "Enable or disable the addon"
local defaultValue = true

local setting = Settings.RegisterAddOnSetting(category, variable, variable, Settings.VarType.Boolean, name, defaultValue)
Settings.CreateCheckbox(category, setting, tooltip)

-- Dropdown setting
local function GetOptions()
    local container = Settings.CreateControlTextContainer()
    container:Add("option1", "Option 1")
    container:Add("option2", "Option 2")
    container:Add("option3", "Option 3")
    return container:GetData()
end

local variable = "MyAddon_DropdownValue"
local name = "Dropdown Setting"
local tooltip = "Select an option"
local defaultValue = "option1"

local setting = Settings.RegisterAddOnSetting(category, variable, variable, Settings.VarType.String, name, defaultValue)
Settings.CreateDropdown(category, setting, GetOptions, tooltip)

-- Register category
Settings.RegisterAddOnCategory(category)
```

### Ace3Config Pattern

> **Library Alternative (requires Ace3):** The following uses AceConfig-3.0 and AceConfigDialog-3.0. See [09a_Ace3_Library_Guide.md](09a_Ace3_Library_Guide.md) for details.

**Many addons use Ace3 for configuration as a library convenience:**

```lua
local options = {
    name = "MyAddon",
    handler = MyAddon,
    type = "group",
    args = {
        enabled = {
            type = "toggle",
            name = "Enable",
            desc = "Enable/disable the addon",
            get = function(info) return MyAddon.db.profile.enabled; end,
            set = function(info, value)
                MyAddon.db.profile.enabled = value
            end,
        },
    },
}

-- Register options
LibStub("AceConfig-3.0"):RegisterOptionsTable("MyAddon", options)
LibStub("AceConfigDialog-3.0"):AddToBlizOptions("MyAddon", "MyAddon")
```

---

## Slash Commands

### Basic Slash Command

```lua
SLASH_MYADDON1 = "/myaddon"
SLASH_MYADDON2 = "/ma"

SlashCmdList["MYADDON"] = function(msg, editBox)
    local command, rest = msg:match("^(%S*)%s*(.-)$")

    if command == "show" then
        MyAddon:ShowFrame()
    elseif command == "hide" then
        MyAddon:HideFrame()
    elseif command == "config" then
        MyAddon:OpenConfig()
    else
        print("MyAddon commands:")
        print("  /myaddon show - Show main frame")
        print("  /myaddon hide - Hide main frame")
        print("  /myaddon config - Open configuration")
    end
end
```

### Advanced Command Parsing

```lua
local commands = {
    ["show"] = function()
        MyAddon:ShowFrame()
    end,
    ["hide"] = function()
        MyAddon:HideFrame()
    end,
    ["set"] = function(rest)
        local key, value = rest:match("^(%S+)%s+(.+)$")
        if key and value then
            MyAddonDB.settings[key] = value
            print("Set", key, "to", value)
        else
            print("Usage: /myaddon set <key> <value>")
        end
    end,
    ["get"] = function(rest)
        local key = rest:match("^(%S+)")
        if key then
            print(key, "=", tostring(MyAddonDB.settings[key]))
        else
            print("Usage: /myaddon get <key>")
        end
    end,
}

SlashCmdList["MYADDON"] = function(msg)
    local command, rest = msg:match("^(%S*)%s*(.-)$")
    command = command:lower()

    local handler = commands[command]
    if handler then
        handler(rest)
    else
        print("Unknown command:", command)
        print("Available commands:")
        for cmd in pairs(commands) do
            print("  /myaddon", cmd)
        end
    end
end
```

---

## LibStub and Library Management

### LibStub Pattern

**Standard library loader:**

```lua
-- Check if library already loaded
local lib = LibStub:GetLibrary("MyLib-1.0", true)
if lib then
    return lib  -- Already loaded
end

-- Create new library
local lib = LibStub:NewLibrary("MyLib-1.0", 1)  -- Name, version
if not lib then
    return  -- Older version already loaded
end

-- Library implementation
function lib:DoSomething()
    print("Library function called!")
end
```

### Embedding Libraries

**Pattern used by Ace3 and other community libraries:**

> The simplest way to use a library is to call it directly: `local MyLib = LibStub("MyLib-1.0"); MyLib:DoSomething()`. The embed/mixin pattern below copies library methods onto your addon object for convenience, and is the approach Ace3 uses internally.

```lua
-- MyLib.lua
local MAJOR, MINOR = "MyLib-1.0", 1
local lib = LibStub:NewLibrary(MAJOR, MINOR)
if not lib then return; end

-- Embed mixin
lib.embeds = lib.embeds or {}

function lib:Embed(target)
    for k, v in pairs(self) do
        if type(v) == "function" and k ~= "Embed" then
            target[k] = v
        end
    end
    self.embeds[target] = true
end

-- Usage
MyAddon = {}
LibStub("MyLib-1.0"):Embed(MyAddon)
MyAddon:DoSomething()  -- Library method now available
```

---

## Localization Patterns

### Simple Localization (No Library)

```lua
local L = {}

if GetLocale() == "deDE" then
    L["WELCOME"] = "Willkommen!"
elseif GetLocale() == "frFR" then
    L["WELCOME"] = "Bienvenue!"
else
    L["WELCOME"] = "Welcome!"
end

print(L["WELCOME"])
```

### AceLocale Localization

> **Library Alternative (requires Ace3):** See [09_Addon_Libraries_Guide.md](09_Addon_Libraries_Guide.md) for detailed AceLocale documentation.

```lua
-- Locales/enUS.lua
local L = LibStub("AceLocale-3.0"):NewLocale("MyAddon", "enUS", true)
if not L then return; end

L["WELCOME_MESSAGE"] = "Welcome to MyAddon!"
L["OPTION_ENABLED"] = "Enabled"

-- Locales/deDE.lua
local L = LibStub("AceLocale-3.0"):NewLocale("MyAddon", "deDE")
if not L then return; end

L["WELCOME_MESSAGE"] = "Willkommen bei MyAddon!"
L["OPTION_ENABLED"] = "Aktiviert"

-- Usage in code
local L = LibStub("AceLocale-3.0"):GetLocale("MyAddon")
print(L["WELCOME_MESSAGE"])
```

---

## Profile Systems

### Base WoW API Profile Pattern

```lua
-- Manual profile system using raw SavedVariables
local ADDON_NAME, ns = ...
local defaultSettings = { fontSize = 12, showTooltips = true }

local frame = CreateFrame("Frame")
frame:RegisterEvent("ADDON_LOADED")
frame:SetScript("OnEvent", function(self, event, addon)
    if addon == ADDON_NAME then
        MyAddonDB = MyAddonDB or { profiles = {}, currentProfile = "Default" }
        local db = MyAddonDB

        -- Ensure current profile exists with defaults
        local profileName = db.currentProfile
        if not db.profiles[profileName] then
            db.profiles[profileName] = {}
        end

        -- Apply defaults to current profile
        for k, v in pairs(defaultSettings) do
            if db.profiles[profileName][k] == nil then
                db.profiles[profileName][k] = v
            end
        end

        ns.settings = db.profiles[profileName]
        self:UnregisterEvent("ADDON_LOADED")
    end
end)

-- Switch profile
function ns:SetProfile(name)
    MyAddonDB.currentProfile = name
    if not MyAddonDB.profiles[name] then
        MyAddonDB.profiles[name] = {}
        for k, v in pairs(defaultSettings) do
            MyAddonDB.profiles[name][k] = v
        end
    end
    ns.settings = MyAddonDB.profiles[name]
end
```

### AceDB-3.0 Profile Pattern

> **Library Alternative (requires Ace3):** See [09_Addon_Libraries_Guide.md](09_Addon_Libraries_Guide.md) for detailed AceDB documentation.

```lua
-- Database with profiles
local defaults = {
    profile = {
        setting1 = true,
        setting2 = "value",
    },
}

MyAddon.db = LibStub("AceDB-3.0"):New("MyAddonDB", defaults, true)

-- Access current profile
local profile = MyAddon.db.profile
print(profile.setting1)

-- Change profile
MyAddon.db:SetProfile("ProfileName")

-- Reset profile
MyAddon.db:ResetProfile()
```

---

## Update and Notification Systems

### Version Check Pattern

```lua
local CURRENT_VERSION = "1.2.3"

-- Parse version string
local function ParseVersion(versionString)
    local major, minor, patch = versionString:match("(%d+)%.(%d+)%.(%d+)")
    return tonumber(major), tonumber(minor), tonumber(patch)
end

-- Compare versions
local function IsNewerVersion(v1, v2)
    local major1, minor1, patch1 = ParseVersion(v1)
    local major2, minor2, patch2 = ParseVersion(v2)

    if major1 > major2 then return true; end
    if major1 < major2 then return false; end
    if minor1 > minor2 then return true; end
    if minor1 < minor2 then return false; end
    return patch1 > patch2
end

-- Check for updates
function MyAddon:CheckVersion()
    if not MyAddonDB.lastVersion then
        -- First install
        MyAddonDB.lastVersion = CURRENT_VERSION
        self:ShowWelcomeMessage()
    elseif IsNewerVersion(CURRENT_VERSION, MyAddonDB.lastVersion) then
        -- Updated
        self:ShowUpdateMessage(MyAddonDB.lastVersion, CURRENT_VERSION)
        MyAddonDB.lastVersion = CURRENT_VERSION
    end
end
```

### Changelog Notification

```lua
local CHANGELOG = {
    ["1.2.3"] = {
        "Added new feature X",
        "Fixed bug Y",
        "Improved performance",
    },
    ["1.2.2"] = {
        "Fixed critical bug",
    },
}

function MyAddon:ShowChangelog(fromVersion, toVersion)
    print(format("|cff00ff00%s updated from %s to %s|r", AddonName, fromVersion, toVersion))

    -- Collect and sort matching versions (pairs order is undefined)
    local versions = {}
    for version in pairs(CHANGELOG) do
        if IsNewerVersion(version, fromVersion) and not IsNewerVersion(version, toVersion) then
            versions[#versions + 1] = version
        end
    end
    table.sort(versions, IsNewerVersion)

    for _, version in ipairs(versions) do
        print(format("|cffFFFF00Version %s:|r", version))
        for _, change in ipairs(CHANGELOG[version]) do
            print(format("  - %s", change))
        end
    end
end
```

---

## Performance Best Practices

### Throttling Updates

```lua
-- Throttle function calls
local function CreateThrottle(delay)
    local lastUpdate = 0
    return function(callback)
        local now = GetTime()
        if now - lastUpdate >= delay then
            lastUpdate = now
            callback()
        end
    end
end

-- Usage
local throttledUpdate = CreateThrottle(0.5)  -- Max once per 0.5 seconds

frame:SetScript("OnUpdate", function()
    throttledUpdate(function()
        -- Expensive update code
        MyAddon:RefreshDisplay()
    end)
end)
```

### Debouncing

```lua
-- Debounce: Only call after activity stops
local function CreateDebounce(delay)
    local timer
    return function(callback)
        if timer then
            timer:Cancel()
        end
        timer = C_Timer.NewTimer(delay, callback)
    end
end

-- Usage
local debouncedSave = CreateDebounce(1.0)  -- Wait 1 second after last change

function MyAddon:OnSettingChanged()
    debouncedSave(function()
        MyAddon:SaveSettings()
    end)
end
```

### Simple Function Profiling

```lua
-- Wrapper to identify performance bottlenecks using debugprofilestop()

local function ProfileFunction(funcName, func)
    return function(...)
        local startTime = debugprofilestop()
        local results = {func(...)}
        local elapsed = debugprofilestop() - startTime

        if elapsed > 1 then  -- Log slow calls (>1ms)
            print(format("%s took %.2fms", funcName, elapsed))
        end

        return unpack(results)
    end
end

-- Wrap expensive functions
MyAddon.ExpensiveFunction = ProfileFunction("ExpensiveFunction", MyAddon.ExpensiveFunction)
```

---

## Distribution and Packaging

### TOC File Best Practices

```
## Interface: 120000
## Title: My Addon
## Notes: Short description of what the addon does
## Author: Your Name
## Version: @project-version@  # Auto-filled by packager
## X-Category: Interface Enhancements
## X-Website: https://github.com/yourname/myaddon
## X-Curse-Project-ID: 12345
## X-Wago-ID: yourwagoid

## SavedVariables: MyAddonDB
## SavedVariablesPerCharacter: MyAddonCharDB

# Libraries
Libs\LibStub\LibStub.lua
Libs\AceAddon-3.0\AceAddon-3.0.lua

# Localization
Locales\enUS.lua
Locales\deDE.lua

# Core
Core.lua
Config.lua

# Modules
Modules\Module1.lua
Modules\Module2.lua

# UI
UI\Templates.xml
UI\MainFrame.lua
UI\MainFrame.xml
```

### .pkgmeta for CurseForge/Wago

```yaml
package-as: MyAddon

enable-nolib-creation: no

externals:
    Libs/LibStub:
        url: https://repos.curseforge.com/wow/libstub/trunk
    Libs/AceAddon-3.0:
        url: https://repos.curseforge.com/wow/ace3/trunk/AceAddon-3.0
    Libs/AceDB-3.0:
        url: https://repos.curseforge.com/wow/ace3/trunk/AceDB-3.0

ignore:
    - README.md
    - .git
    - .github
```

---

## Addon Categories and 12.0 Compatibility

The 12.0.0 (Midnight) expansion introduces significant API changes that affect most addon categories. This section documents the impact and migration strategies for each major category.

### Damage Meters (Details, Skada, Recount) - FUNCTIONAL WITH WORKAROUNDS IN 12.0.0+

> **Updated March 2026:** While combat log parsing is blocked and C_DamageMeter data is secret-protected during combat, **proven workarounds exist** that allow third-party damage meters to function. Recount has demonstrated a successful 12.0.1 update using these techniques.

**12.0 Changes:**
- **Combat log events BLOCKED** - `RegisterEvent("COMBAT_LOG_EVENT_UNFILTERED")` throws `ADDON_ACTION_FORBIDDEN`
- **C_DamageMeter data is SECRET during combat** - Key values (`name`, `totalAmount`, `amountPerSecond`) are secret values
- **Workarounds exist** - Secret values can be extracted for display and become fully readable after combat

**Migration Strategy for Damage Meter Addons:**
```lua
-- Use C_DamageMeter API with secret-value workarounds
local sessionData = C_DamageMeter.GetCombatSessionFromType(sessionType, meterType)
for i, source in ipairs(sessionData.combatSources) do
    -- WORKAROUND 1: pcall(string.format) extracts secret values as text at C++ level
    local ok, dpsText = pcall(string.format, "%.0f", source.amountPerSecond)
    if ok then
        myFontString:SetText(dpsText .. " DPS")
    end

    -- WORKAROUND 2: StatusBar accepts secret values at C++ level
    pcall(myBar.SetMinMaxValues, myBar, 0, sessionData.maxAmount)
    pcall(myBar.SetValue, myBar, source.totalAmount)

    -- WORKAROUND 3: Array index = sort order (i=1 is highest)
    local syntheticRank = 1000 - i

    -- Always accessible (not secret):
    local class = source.classFilename   -- "DEATHKNIGHT"
    local isMe = source.isLocalPlayer    -- true/false
end

-- WORKAROUND 4: Post-combat, all values become fully readable
local reparseFrame = CreateFrame("Frame")
reparseFrame:RegisterEvent("PLAYER_REGEN_ENABLED")
reparseFrame:SetScript("OnEvent", function()
    C_Timer.After(0.5, function()
        -- Full arithmetic, comparisons, string ops now work
        ReparseAllSessions()
    end)
end)
```

**Key Techniques:**
- `pcall(string.format, "%.0f", secretValue)` -- formats secret numbers into normal displayable strings
- `StatusBar:SetValue(secretValue)` / `SetMinMaxValues()` -- proportional bar display with secrets
- Array index from `combatSources` preserves sort order (index 1 = highest)
- `isLocalPlayer` and `classFilename` are always accessible for UI styling
- After `PLAYER_REGEN_ENABLED` + delay, all values become non-secret for post-combat analysis
- See [12a_Secret_Safe_APIs.md](12a_Secret_Safe_APIs.md) for complete documentation of workaround patterns

### Boss Mods (DBM, BigWigs, LittleWigs)

**12.0 Changes:**
- **C_EncounterTimeline API** - Official boss timeline system with phase tracking
- **C_EncounterWarnings API** - Native boss warnings that addons can supplement
- **Secret values** - Unit health in boss encounters may show approximated values

**Migration Strategy:**
```lua
-- Check for official encounter systems
local hasEncounterTimeline = C_EncounterTimeline ~= nil
local hasEncounterWarnings = C_EncounterWarnings ~= nil

-- Modern boss mod initialization
function BossMod:OnEncounterStart(encounterID, encounterName, difficultyID, groupSize)
    -- Check if official timeline is available
    if hasEncounterTimeline then
        local phases = C_EncounterTimeline.GetEncounterPhases(encounterID)
        if phases then
            self:IntegrateOfficialPhases(phases)
        end
    end

    -- Register for official warnings to avoid duplicates
    if hasEncounterWarnings then
        self:RegisterCallback(C_EncounterWarnings, "OnWarningDisplayed", "HandleOfficialWarning")
    end
end

-- Handle secret values for boss health
function BossMod:GetBossHealth(unit)
    -- In 12.0+, UnitHealthPercent(unit) returns a 0-1 float without a curve,
    -- or 0-100 with CurveConstants.ScaleTo100. No issecretvalue guard needed
    -- when called without a curve (SecretWhenCurveSecret only).
    if UnitHealthPercent then
        return UnitHealthPercent(unit, false, CurveConstants.ScaleTo100)  -- Returns 0-100
    end

    -- Fallback for pre-12.0 (UnitHealth/UnitHealthMax are not secret in older clients)
    local health = UnitHealth(unit)
    local maxHealth = UnitHealthMax(unit)
    if maxHealth > 0 then
        return (health / maxHealth) * 100
    end
    return 0
end

-- Supplement rather than replace official warnings
function BossMod:DisplayWarning(warningType, spellID, text)
    -- Check if official warning system will handle this
    if hasEncounterWarnings and C_EncounterWarnings.HasWarningForSpell(spellID) then
        -- Official system handles it - optionally add supplemental info
        return
    end

    -- Display custom warning
    self:ShowWarningFrame(warningType, text)
end
```

### Action Bar Addons (Bartender, ElvUI, Dominos)

**12.0 Changes - BREAKING:**
- **Global action bar functions REMOVED** - Must use C_ActionBar namespace exclusively
- **Major rewrite required** for any addon using legacy action bar APIs
- **New profiler metrics** available for optimization

**Migration Strategy:**
```lua
-- 12.0 REQUIRED: All action bar functions must use C_ActionBar
-- Legacy functions like GetActionInfo(), PickupAction() are REMOVED

-- Modern action bar slot management
local function GetActionSlotInfo(slot)
    -- 12.0: Must use C_ActionBar namespace
    if C_ActionBar.GetActionInfo then
        local actionType, id, subType = C_ActionBar.GetActionInfo(slot)
        return actionType, id, subType
    end
    return nil
end

-- Modern action pickup
local function PickupActionSlot(slot)
    if C_ActionBar.PickupAction then
        C_ActionBar.PickupAction(slot)
    end
end

-- Modern action placement
local function PlaceActionInSlot(slot)
    if C_ActionBar.PlaceAction then
        C_ActionBar.PlaceAction(slot)
    end
end

-- Modern action bar page management
local function GetCurrentActionBarPage()
    if C_ActionBar.GetCurrentPage then
        return C_ActionBar.GetCurrentPage()
    end
    return 1
end

-- Check if action has range
local function IsActionInRange(slot, unit)
    if C_ActionBar.IsActionInRange then
        return C_ActionBar.IsActionInRange(slot, unit)
    end
    return nil
end

-- Get action cooldown
local function GetActionCooldownInfo(slot)
    if C_ActionBar.GetActionCooldown then
        return C_ActionBar.GetActionCooldown(slot)
    end
    return 0, 0, 0
end

-- Full compatibility wrapper for action bar addons
local ActionBarCompat = {}

function ActionBarCompat:Initialize()
    -- Verify required APIs exist
    assert(C_ActionBar, "C_ActionBar namespace required for 12.0+")
    assert(C_ActionBar.GetActionInfo, "C_ActionBar.GetActionInfo required")
    assert(C_ActionBar.HasAction, "C_ActionBar.HasAction required")
end

function ActionBarCompat:HasAction(slot)
    return C_ActionBar.HasAction(slot)
end

function ActionBarCompat:GetActionTexture(slot)
    return C_ActionBar.GetActionTexture(slot)
end

function ActionBarCompat:GetActionText(slot)
    return C_ActionBar.GetActionText(slot)
end

function ActionBarCompat:GetActionCharges(slot)
    return C_ActionBar.GetActionCharges(slot)
end

function ActionBarCompat:IsUsableAction(slot)
    local usable, noMana, noRange = C_ActionBar.IsUsableAction(slot)
    return usable, noMana
end
```

### Unit Frame Addons (ElvUI, Shadowed Unit Frames, Pitbull)

**12.0 Changes:**
- **Secret values system** - Health and power values may be approximated during combat
- **New Unit functions** - UnitHealthPercent(), UnitPowerPercent() provide reliable percentages
- **Health prediction changes** - Must handle secrets in prediction calculations

**Migration Strategy:**
```lua
-- Modern unit health display with secret handling
local function GetUnitHealthDisplay(unit)
    -- 12.0: Use percentage functions for reliable display
    local percent = UnitHealthPercent and UnitHealthPercent(unit)

    if percent then
        -- Reliable percentage even with secrets
        local maxHealth = UnitHealthMax(unit)
        local displayHealth = math.floor(maxHealth * percent / 100)
        return displayHealth, maxHealth, percent
    end

    -- Fallback
    local health = UnitHealth(unit)
    local maxHealth = UnitHealthMax(unit)
    local calcPercent = maxHealth > 0 and (health / maxHealth * 100) or 0
    return health, maxHealth, calcPercent
end

-- Modern power display
local function GetUnitPowerDisplay(unit, powerType)
    local percent = UnitPowerPercent and UnitPowerPercent(unit, powerType)

    if percent then
        local maxPower = UnitPowerMax(unit, powerType)
        local displayPower = math.floor(maxPower * percent / 100)
        return displayPower, maxPower, percent
    end

    local power = UnitPower(unit, powerType)
    local maxPower = UnitPowerMax(unit, powerType)
    local calcPercent = maxPower > 0 and (power / maxPower * 100) or 0
    return power, maxPower, calcPercent
end

-- Health bar update with secrets awareness
function UnitFrameMixin:UpdateHealth()
    local displayHealth, maxHealth, percent = GetUnitHealthDisplay(self.unit)

    self.healthBar:SetMinMaxValues(0, maxHealth)
    self.healthBar:SetValue(displayHealth)

    -- Display percentage for accuracy during secrets
    if self.healthText then
        if self.showPercentage then
            self.healthText:SetText(format("%.0f%%", percent))
        else
            self.healthText:SetText(AbbreviateNumber(displayHealth))
        end
    end
end

-- Incoming heal prediction with secrets handling
function UnitFrameMixin:UpdateHealPrediction()
    local _, maxHealth, percent = GetUnitHealthDisplay(self.unit)
    local incomingHeal = UnitGetIncomingHeals(self.unit) or 0
    local absorb = UnitGetTotalAbsorbs(self.unit) or 0

    -- Calculate based on percentages for consistency
    local currentPercent = percent / 100
    local healPercent = maxHealth > 0 and (incomingHeal / maxHealth) or 0
    local absorbPercent = maxHealth > 0 and (absorb / maxHealth) or 0

    self:SetHealPredictionBars(currentPercent, healPercent, absorbPercent)
end
```

### Transmog Addons (Narcissus, BetterWardrobe, Mog It)

**12.0 Changes - MAJOR OVERHAUL:**
- **C_TransmogOutfitInfo namespace** - Complete new API for outfit management
- **Old outfit APIs removed** - C_TransmogSets heavily restructured
- **Custom sets system** - New player-created outfit functionality

**Migration Strategy:**
```lua
-- Check for 12.0 transmog API
local hasNewTransmogAPI = C_TransmogOutfitInfo ~= nil

-- Modern outfit management
local TransmogCompat = {}

function TransmogCompat:GetOutfits()
    if hasNewTransmogAPI and C_TransmogOutfitInfo.GetOutfits then
        return C_TransmogOutfitInfo.GetOutfits()
    end
    -- Legacy fallback
    return C_TransmogCollection and C_TransmogCollection.GetOutfits() or {}
end

function TransmogCompat:GetOutfitInfo(outfitID)
    if hasNewTransmogAPI and C_TransmogOutfitInfo.GetOutfitInfo then
        return C_TransmogOutfitInfo.GetOutfitInfo(outfitID)
    end
    return nil
end

function TransmogCompat:SaveOutfit(name, sources)
    if hasNewTransmogAPI and C_TransmogOutfitInfo.SaveOutfit then
        return C_TransmogOutfitInfo.SaveOutfit(name, sources)
    end
    -- Legacy
    if C_TransmogCollection and C_TransmogCollection.SaveOutfit then
        return C_TransmogCollection.SaveOutfit(name, sources)
    end
    return false
end

function TransmogCompat:DeleteOutfit(outfitID)
    if hasNewTransmogAPI and C_TransmogOutfitInfo.DeleteOutfit then
        return C_TransmogOutfitInfo.DeleteOutfit(outfitID)
    end
    if C_TransmogCollection and C_TransmogCollection.DeleteOutfit then
        return C_TransmogCollection.DeleteOutfit(outfitID)
    end
end

function TransmogCompat:GetSlotSources(slotID)
    if hasNewTransmogAPI and C_TransmogOutfitInfo.GetSlotSources then
        return C_TransmogOutfitInfo.GetSlotSources(slotID)
    end
    if C_TransmogCollection then
        return C_TransmogCollection.GetAppearanceSources(slotID)
    end
    return {}
end

-- Apply outfit
function TransmogCompat:ApplyOutfit(outfitID)
    if hasNewTransmogAPI and C_TransmogOutfitInfo.ApplyOutfit then
        C_TransmogOutfitInfo.ApplyOutfit(outfitID)
        return true
    end
    return false
end
```

### Bag/Inventory Addons (Bagnon, AdiBags, ArkInventory)

**12.0 Changes:**
- **Void Storage REMOVED** (11.2.0) - Must remove or gracefully handle missing API
- **Socket APIs moved** to C_ItemSocketInfo namespace (11.2.5+)
- **Bank system restructured** - New Warband bank integration

**Migration Strategy:**
```lua
-- Check for removed/moved APIs
local hasVoidStorage = C_VoidStorage ~= nil
local hasNewSocketAPI = C_ItemSocketInfo ~= nil

-- Socket API compatibility
local function GetItemSockets(itemLink)
    if hasNewSocketAPI and C_ItemSocketInfo.GetItemSockets then
        return C_ItemSocketInfo.GetItemSockets(itemLink)
    end
    -- Legacy fallback
    if GetItemStats then
        local stats = GetItemStats(itemLink)
        return stats and stats["EMPTY_SOCKET_RED"] or 0
    end
    return 0
end

function GetSocketInfo(socketIndex)
    if hasNewSocketAPI and C_ItemSocketInfo.GetSocketInfo then
        return C_ItemSocketInfo.GetSocketInfo(socketIndex)
    end
    -- Legacy
    return GetExistingSocketInfo and GetExistingSocketInfo(socketIndex)
end

-- Void Storage handling - gracefully removed
local function HasVoidStorageAccess()
    if not hasVoidStorage then
        return false  -- Feature removed in 11.2.0
    end
    return CanUseVoidStorage and CanUseVoidStorage()
end

-- Bank compatibility with Warband bank
local function GetBankSlotInfo(bagIndex, slotIndex)
    -- Check for Warband bank API
    if C_Bank and C_Bank.GetBankSlotInfo then
        return C_Bank.GetBankSlotInfo(bagIndex, slotIndex)
    end
    -- Legacy
    return GetContainerItemInfo(bagIndex, slotIndex)
end

-- Modern container iteration
local function IterateBagSlots(bagID, callback)
    local numSlots = C_Container.GetContainerNumSlots(bagID)
    for slot = 1, numSlots do
        local itemInfo = C_Container.GetContainerItemInfo(bagID, slot)
        callback(slot, itemInfo)
    end
end

-- Inventory addon initialization
function InventoryAddon:Initialize()
    self.frame = CreateFrame("Frame")
    self.frame:RegisterEvent("BAG_UPDATE")
    self.frame:RegisterEvent("BANKFRAME_OPENED")
    self.frame:RegisterEvent("BANKFRAME_CLOSED")
    self.frame:SetScript("OnEvent", function(_, event, ...)
        if InventoryAddon[event] then
            InventoryAddon[event](InventoryAddon, ...)
        end
    end)

    -- Remove Void Storage tab if feature is gone
    if not hasVoidStorage then
        self:RemoveVoidStorageTab()
    end

    -- Check for Warband bank
    if C_Bank and C_Bank.HasWarbandBank then
        self:SetupWarbandBankTab()
    end
end
```

### Housing Addons (NEW CATEGORY in 12.0)

**12.0 introduces player housing with the C_Housing system. This enables an entirely new addon category.**

**Reference:** See [11_Housing_System_Guide.md](11_Housing_System_Guide.md) for complete API documentation.

```lua
-- Housing addon base structure (base WoW API)
local ADDON_NAME, ns = ...
local HousingAddon = {}
ns.HousingAddon = HousingAddon

-- Check for housing availability
local hasHousingSystem = C_Housing ~= nil

local frame = CreateFrame("Frame")
frame:RegisterEvent("ADDON_LOADED")

frame:SetScript("OnEvent", function(self, event, ...)
    if event == "ADDON_LOADED" and ... == ADDON_NAME then
        if not hasHousingSystem then
            print("Housing system not available - addon disabled")
            return
        end

        -- Initialize saved variables
        MyHousingAddonDB = MyHousingAddonDB or { layouts = {} }
        HousingAddon.db = MyHousingAddonDB

        -- Register housing events
        self:RegisterEvent("HOUSING_ENTERED")
        self:RegisterEvent("HOUSING_EXITED")
        self:RegisterEvent("HOUSING_DECOR_PLACED")
        self:RegisterEvent("HOUSING_DECOR_REMOVED")
        self:RegisterEvent("HOUSING_DECOR_MOVED")
        self:RegisterEvent("HOUSING_LAYOUT_CHANGED")
        self:UnregisterEvent("ADDON_LOADED")
    elseif HousingAddon[event] then
        HousingAddon[event](HousingAddon, ...)
    end
end)

-- Housing state management
function HousingAddon:HOUSING_ENTERED(houseID)
    local houseInfo = C_Housing.GetHouseInfo(houseID)
    self.currentHouse = houseInfo
    self:RefreshDecorList()
end

function HousingAddon:HOUSING_EXITED()
    self.currentHouse = nil
end

-- Decor management
function HousingAddon:GetPlacedDecor()
    if not self.currentHouse then return {} end
    return C_Housing.GetPlacedDecor(self.currentHouse.houseID)
end

function HousingAddon:PlaceDecor(decorID, position, rotation)
    if not C_Housing.CanPlaceDecor(decorID) then
        print("Cannot place this decor item")
        return false
    end

    return C_Housing.PlaceDecor(decorID, position, rotation)
end

-- Layout save/load
function HousingAddon:SaveLayout(layoutName)
    local decor = self:GetPlacedDecor()
    local layout = {
        name = layoutName,
        houseID = self.currentHouse.houseID,
        decor = {},
    }

    for _, item in ipairs(decor) do
        table.insert(layout.decor, {
            decorID = item.decorID,
            position = item.position,
            rotation = item.rotation,
        })
    end

    self.db.layouts[layoutName] = layout
    print("Layout saved:", layoutName)
end

function HousingAddon:LoadLayout(layoutName)
    local layout = self.db.layouts[layoutName]
    if not layout then
        print("Layout not found:", layoutName)
        return
    end

    -- Clear current decor first
    C_Housing.ClearAllDecor()

    -- Place saved decor
    for _, item in ipairs(layout.decor) do
        self:PlaceDecor(item.decorID, item.position, item.rotation)
    end

    print("Layout loaded:", layoutName)
end

-- Neighborhood features
function HousingAddon:GetNeighborhood()
    if C_Housing.GetNeighborhoodInfo then
        return C_Housing.GetNeighborhoodInfo()
    end
    return nil
end

function HousingAddon:VisitNeighbor(playerGUID)
    if C_Housing.CanVisitHouse and C_Housing.CanVisitHouse(playerGUID) then
        C_Housing.VisitHouse(playerGUID)
        return true
    end
    return false
end
```

---

## Library Updates for 12.0

Several community libraries may need updates or have native replacements in 12.0.

### Native Replacements for Common Libraries

**C_EncodingUtil (replaces some LibSerialize/LibCompress functionality):**
```lua
-- 12.0 native encoding utilities
if C_EncodingUtil then
    -- Serialize data to string
    local encoded = C_EncodingUtil.EncodeTable(myTable)

    -- Deserialize string to data
    local decoded = C_EncodingUtil.DecodeTable(encoded)

    -- Compress string
    local compressed = C_EncodingUtil.CompressString(largeString)

    -- Decompress string
    local decompressed = C_EncodingUtil.DecompressString(compressed)
else
    -- Fallback to libraries
    local LibSerialize = LibStub("LibSerialize")
    local LibDeflate = LibStub("LibDeflate")
end
```

**Native Table Functions (reduce utility library needs):**
```lua
-- 12.0 native Lua extensions
-- table.create(narr, nrec) - Pre-allocate table memory
local t = table.create(100, 0)  -- 100 array slots, 0 hash slots

-- table.count(t) - Count all entries (both array and hash)
local count = table.count(myTable)

-- These reduce need for custom implementations
local function CreatePooledTable(size)
    return table.create(size, 0)
end

local function GetTableSize(t)
    if table.count then
        return table.count(t)
    end
    -- Fallback
    local count = 0
    for _ in pairs(t) do count = count + 1; end
    return count
end
```

### Library Compatibility Checklist

**Libraries that need 12.0 updates:**
- **LibDBIcon** - May need updates for minimap changes
- **LibDataBroker** - Check for compatibility with new UI
- **LibActionButton** - CRITICAL: Must update for C_ActionBar changes
- **LibCompress/LibSerialize** - Consider C_EncodingUtil as alternative
- **LibQTip** - Verify tooltip system compatibility
- **LibRangeCheck** - May need updates for range API changes

**Libraries likely still compatible:**
- **LibStub** - Core loader, should work unchanged
- **Ace3 suite** - Community maintains compatibility
- **CallbackHandler** - Pure Lua, no API dependencies
- **AceLocale** - No API dependencies
- **AceDB** - SavedVariables system unchanged

**Checking Library Compatibility:**
```lua
-- Verify libraries load without error
local function CheckLibraryCompat(libName)
    local success, lib = pcall(function()
        return LibStub(libName)
    end)

    if success and lib then
        print(format("%s loaded successfully", libName))
        return true
    else
        print(format("%s FAILED to load: %s", libName, tostring(lib)))
        return false
    end
end

-- Check critical libraries
CheckLibraryCompat("AceAddon-3.0")
CheckLibraryCompat("AceDB-3.0")
CheckLibraryCompat("AceConfig-3.0")
CheckLibraryCompat("LibDBIcon-1.0")
```

### Migration Helper

```lua
-- Helper module for detecting 12.0 API changes
local APICompat = {}

APICompat.features = {
    hasHousing = C_Housing ~= nil,
    hasEncodingUtil = C_EncodingUtil ~= nil,
    hasNewActionBar = C_ActionBar ~= nil and C_ActionBar.GetActionInfo ~= nil,
    hasNewTransmog = C_TransmogOutfitInfo ~= nil,
    hasOfficialDamageMeter = C_DamageMeter ~= nil, -- NOTE: API exists but data is secret-protected!
    hasEncounterTimeline = C_EncounterTimeline ~= nil,
    hasEncounterWarnings = C_EncounterWarnings ~= nil,
    hasVoidStorage = C_VoidStorage ~= nil,
    hasNewSocketAPI = C_ItemSocketInfo ~= nil,
    hasTableCreate = table.create ~= nil,
    hasTableCount = table.count ~= nil,
}

function APICompat:PrintCompatReport()
    print("=== 12.0 API Compatibility Report ===")
    for feature, available in pairs(self.features) do
        local status = available and "|cff00ff00YES|r" or "|cffff0000NO|r"
        print(format("  %s: %s", feature, status))
    end
end

function APICompat:RequiresFeature(featureName)
    if not self.features[featureName] then
        error(format("Required feature '%s' not available in this WoW version", featureName))
    end
end

-- Export for other addons
_G.APICompat = APICompat
```

---

<!-- CLAUDE_SKIP_START -->
## Common Community Patterns Summary

### Do's:
1. Use LibStub for library management
2. Implement profile system for settings
3. Provide localization support
4. Use slash commands for user interaction
5. Version your database for migrations
6. Throttle/debounce expensive operations
7. Follow community naming conventions
8. Provide clear documentation
9. Use semantic versioning (1.2.3)
10. Test with different locales/UI scales

### Don'ts:
1. Don't hardcode English strings
2. Don't embed entire libraries if not needed
3. Don't pollute global namespace
4. Don't break on updates
5. Don't ignore user feedback
6. Don't use deprecated APIs without fallbacks
7. Don't create UI every frame (pool!)
8. Don't save temporary data
9. Don't ignore memory usage
10. Don't ship debug code in release

---

<!-- CLAUDE_SKIP_END -->
