# Ace3 Library Suite -- Comprehensive Guide

> This guide covers the Ace3 framework exclusively. For base WoW API patterns, see [04_Addon_Structure.md](04_Addon_Structure.md). For non-Ace3 community libraries (LibStub, LibDataBroker, LibSharedMedia, etc.), see [09_Addon_Libraries_Guide.md](09_Addon_Libraries_Guide.md).

## Table of Contents

- [Section 1: Overview and When to Use Ace3](#section-1-overview-and-when-to-use-ace3)
  - [What is Ace3?](#what-is-ace3)
  - [Embedding: How It Works](#embedding-how-it-works)
  - [When to Use Ace3 vs. Base WoW API](#when-to-use-ace3-vs-base-wow-api)
- [Section 2: AceAddon-3.0](#section-2-aceaddon-30)
  - [Creating an Addon](#creating-an-addon)
  - [Lifecycle Callbacks](#lifecycle-callbacks)
  - [Module System](#module-system)
- [Section 3: AceDB-3.0](#section-3-acedb-30)
  - [Creating a Database](#creating-a-database)
  - [Storage Scopes](#storage-scopes)
  - [Defaults Wildcards: `["*"]` and `["**"]`](#defaults-wildcards-and)
  - [Profile Management](#profile-management)
  - [Callbacks](#callbacks)
  - [Namespaces](#namespaces)
  - [Default Stripping on Logout](#default-stripping-on-logout)
- [Section 4: AceDBOptions-3.0](#section-4-acedboptions-30)
  - [`LibStub("AceDBOptions-3.0"):GetOptionsTable(db [, noDefaultProfiles])`](#libstubacedboptions-30getoptionstabledb-nodefaultprofiles)
- [Section 5: AceEvent-3.0](#section-5-aceevent-30)
  - [`self:RegisterEvent(event [, callback [, arg]])`](#selfregistereventevent-callback-arg)
  - [`self:UnregisterEvent(event)`](#selfunregistereventevent)
  - [`self:UnregisterAllEvents()`](#selfunregisterallevents)
  - [Custom Messages (Inter-Addon Communication)](#custom-messages-inter-addon-communication)
  - [How It Works Internally](#how-it-works-internally)
  - [AceEvent vs. Base API vs. EventRegistry](#aceevent-vs-base-api-vs-eventregistry)
- [Section 6: AceConsole-3.0](#section-6-aceconsole-30)
  - [`self:RegisterChatCommand(command, func [, persist])`](#selfregisterchatcommandcommand-func-persist)
  - [`self:UnregisterChatCommand(command)`](#selfunregisterchatcommandcommand)
  - [`self:Print(...)`](#selfprint)
  - [`self:Printf(format, ...)`](#selfprintfformat)
  - [`self:GetArgs(str, numargs, startpos)`](#selfgetargsstr-numargs-startpos)
- [Section 7: AceTimer-3.0](#section-7-acetimer-30)
  - [`self:ScheduleTimer(func, delay, ...)`](#selfscheduletimerfunc-delay)
  - [`self:ScheduleRepeatingTimer(func, delay, ...)`](#selfschedulerepeatingtimerfunc-delay)
  - [`self:CancelTimer(handle)`](#selfcanceltimerhandle)
  - [`self:CancelAllTimers()`](#selfcancelalltimers)
  - [`self:TimeLeft(handle)`](#selftimelefthandle)
  - [Internal Implementation](#internal-implementation)
  - [AceTimer vs. C_Timer](#acetimer-vs-c_timer)
- [Section 8: AceHook-3.0](#section-8-acehook-30)
  - [Hook Types Overview](#hook-types-overview)
  - [`self:Hook([object,] method [, handler [, hookSecure]])`](#selfhookobject-method-handler-hooksecure)
  - [`self:SecureHook([object,] method [, handler])`](#selfsecurehookobject-method-handler)
  - [`self:RawHook([object,] method [, handler [, hookSecure]])`](#selfrawhookobject-method-handler-hooksecure)
  - [Script Hooks](#script-hooks)
  - [`self:Unhook([object,] method)`](#selfunhookobject-method)
  - [`self:UnhookAll()`](#selfunhookall)
  - [`self:IsHooked([object,] method)`](#selfishookedobject-method)
  - [The `self.hooks` Table](#the-selfhooks-table)
  - [AceHook vs. `hooksecurefunc()`](#acehook-vs-hooksecurefunc)
- [AceConfig-3.0 -- Declarative Options Tables](#aceconfig-30-declarative-options-tables)
  - [Registering an Options Table](#registering-an-options-table)
  - [Options Table Structure](#options-table-structure)
  - [Common Fields (All Types)](#common-fields-all-types)
  - [Option Type: group](#option-type-group)
  - [Option Type: toggle](#option-type-toggle)
  - [Option Type: range](#option-type-range)
  - [Option Type: input](#option-type-input)
  - [Option Type: select](#option-type-select)
  - [Option Type: multiselect](#option-type-multiselect)
  - [Option Type: color](#option-type-color)
  - [Option Type: execute](#option-type-execute)
  - [Option Type: header](#option-type-header)
  - [Option Type: description](#option-type-description)
  - [Option Type: keybinding](#option-type-keybinding)
  - [Dynamic Options](#dynamic-options)
  - [Forcing a UI Refresh](#forcing-a-ui-refresh)
- [AceConfigDialog-3.0 -- GUI Generation](#aceconfigdialog-30-gui-generation)
  - [AddToBlizOptions](#addtoblizoptions)
  - [Open (Standalone Window)](#open-standalone-window)
  - [Close](#close)
  - [SetDefaultSize](#setdefaultsize)
  - [SelectGroup](#selectgroup)
  - [Pattern: Button in Blizzard Settings That Opens Standalone Dialog](#pattern-button-in-blizzard-settings-that-opens-standalone-dialog)
- [AceGUI-3.0 -- Widget Toolkit](#acegui-30-widget-toolkit)
  - [Creating and Releasing Widgets](#creating-and-releasing-widgets)
  - [Accessing the Underlying WoW Frame](#accessing-the-underlying-wow-frame)
  - [Container Widgets](#container-widgets)
  - [Standard Widgets](#standard-widgets)
  - [Common Widget Methods](#common-widget-methods)
  - [Layouts](#layouts)
  - [Building a Custom UI: Complete Example](#building-a-custom-ui-complete-example)
  - [TabGroup Example](#tabgroup-example)
- [AceComm-3.0 -- Addon Communication](#acecomm-30-addon-communication)
  - [Embedding](#embedding)
  - [Registering for Messages](#registering-for-messages)
  - [Sending Messages](#sending-messages)
  - [Priority Levels](#priority-levels)
  - [Unregistering](#unregistering)
  - [Auto-Splitting and Reassembly](#auto-splitting-and-reassembly)
  - [ChatThrottleLib Integration](#chatthrottlelib-integration)
  - [Comparison with Base WoW API](#comparison-with-base-wow-api)
- [AceSerializer-3.0 -- Data Serialization](#aceserializer-30-data-serialization)
  - [Embedding](#embedding)
  - [Serialize](#serialize)
  - [Deserialize](#deserialize)
  - [Supported Types](#supported-types)
  - [Common Pattern: Serialize + AceComm](#common-pattern-serialize-acecomm)
  - [Common Pattern: Serialize + LibDeflate for Compressed Storage](#common-pattern-serialize-libdeflate-for-compressed-storage)
- [AceBucket-3.0 -- Event Throttling](#acebucket-30-event-throttling)
  - [Embedding](#embedding)
  - [RegisterBucketEvent](#registerbucketevent)
  - [RegisterBucketMessage](#registerbucketmessage)
  - [Unregistering](#unregistering)
  - [How the Bucket Works Internally](#how-the-bucket-works-internally)
  - [When to Use AceBucket](#when-to-use-acebucket)
  - [Comparison with Dirty-Flag + C_Timer Pattern](#comparison-with-dirty-flag-c_timer-pattern)
- [AceLocale-3.0 -- Localization](#acelocale-30-localization)
  - [Concepts](#concepts)
  - [NewLocale](#newlocale)
  - [GetLocale](#getlocale)
  - [Complete Localization Workflow](#complete-localization-workflow)
  - [Silent Mode vs Raw Mode](#silent-mode-vs-raw-mode)
  - [Gotchas](#gotchas)
- [Complete Ace3 Addon Example](#complete-ace3-addon-example)
  - [TOC File](#toc-file)
  - [Localization: Locales/enUS.lua](#localization-localesenuslua)
  - [Localization: Locales/deDE.lua](#localization-localesdedelua)
  - [Core.lua](#corelua)
  - [Options.lua](#optionslua)
  - [Module: Modules/SessionLog.lua](#module-modulessessionloglua)
- [Ace3 Addon File Structure](#ace3-addon-file-structure)
  - [Small Addon (single purpose, <1000 lines)](#small-addon-single-purpose-1000-lines)
  - [Medium Addon (multiple features, 1000-5000 lines)](#medium-addon-multiple-features-1000-5000-lines)
  - [Large Addon (framework-level, 5000+ lines)](#large-addon-framework-level-5000-lines)
- [Embedding & Distribution](#embedding-distribution)
  - [How to Embed Ace3](#how-to-embed-ace3)
  - [Load Order Requirements](#load-order-requirements)
  - [.pkgmeta for CurseForge/WowInterface Packaging](#pkgmeta-for-curseforgewowinterface-packaging)
  - [External vs Embedded Dependencies](#external-vs-embedded-dependencies)
- [Common Patterns & Recipes](#common-patterns-recipes)
  - [Profile Change Handling](#profile-change-handling)
  - [Cross-Addon Communication with AceComm + AceSerializer](#cross-addon-communication-with-acecomm-aceserializer)
  - [Slash Command with Subcommands Using GetArgs](#slash-command-with-subcommands-using-getargs)
  - [Creating a Standalone Options Window](#creating-a-standalone-options-window)
  - [Event Bucketing for Bag/Inventory Updates](#event-bucketing-for-baginventory-updates)
  - [Timer-Based Polling](#timer-based-polling)
  - [Hooking a Blizzard Function Safely](#hooking-a-blizzard-function-safely)
- [Troubleshooting & Common Mistakes](#troubleshooting-common-mistakes)
  - [1. Missing CallbackHandler-1.0](#1-missing-callbackhandler-10)
  - [2. Wrong TOC Load Order](#2-wrong-toc-load-order)
  - [3. Registering Events in OnInitialize](#3-registering-events-in-oninitialize)
  - [4. AceDB Defaults Stripped on Logout](#4-acedb-defaults-stripped-on-logout)
  - [5. AceConfig Options in Random Order](#5-aceconfig-options-in-random-order)
  - [6. AceGUI Widgets Must Be Released](#6-acegui-widgets-must-be-released)
  - [7. AceComm Prefix Length Limit](#7-acecomm-prefix-length-limit)
  - [8. AceTimer Minimum Delay](#8-acetimer-minimum-delay)
  - [9. Printing in OnInitialize](#9-printing-in-oninitialize)
  - [10. Module OnEnable Not Firing](#10-module-onenable-not-firing)
- [Ace3 vs Base WoW API Quick Reference](#ace3-vs-base-wow-api-quick-reference)

---

## Section 1: Overview and When to Use Ace3

### What is Ace3?

Ace3 is a collection of embeddable Lua libraries for World of Warcraft addon development. It provides standardized solutions for common addon tasks: addon lifecycle management, saved variable handling, event dispatching, slash commands, timers, and function hooking. Each library is independent -- you can use only the ones you need.

Ace3 is distributed via [CurseForge](https://www.curseforge.com/wow/addons/ace3) and [WoWAce](https://www.wowace.com/projects/ace3). Most addon authors **embed** Ace3 libraries directly into their addon's folder rather than depending on a standalone Ace3 install. This ensures your addon works regardless of whether the user has Ace3 installed separately.

### Embedding: How It Works

Ace3 uses a pattern called **embedding** (also known as **mixins**). When you create an addon with embedded libraries, those libraries copy their methods directly onto your addon object. This means you call `self:RegisterEvent(...)` rather than `AceEvent:RegisterEvent(self, ...)`.

The embedding happens through LibStub, a minimal library versioning system. LibStub ensures only one copy of each library (the highest version) is active, even if multiple addons bundle different versions.

```lua
-- LibStub retrieves AceAddon-3.0, then NewAddon creates your addon object
-- and embeds AceEvent-3.0 and AceConsole-3.0 methods onto it
local MyAddon = LibStub("AceAddon-3.0"):NewAddon("MyAddon", "AceEvent-3.0", "AceConsole-3.0")

-- Now you can call AceEvent methods directly on your addon:
function MyAddon:OnEnable()
    self:RegisterEvent("PLAYER_REGEN_DISABLED")  -- from AceEvent-3.0
    self:RegisterChatCommand("myaddon", "SlashCommand")  -- from AceConsole-3.0
end
```

### When to Use Ace3 vs. Base WoW API

**Use Ace3 when:**
- You want structured addon lifecycle management (OnInitialize/OnEnable/OnDisable)
- You need profile-based saved variables with copy/delete/reset support
- You're building a modular addon with independently toggleable features
- You want auto-cleanup on disable (events unregister, timers cancel, hooks remove)
- You need inter-addon messaging (SendMessage/RegisterMessage)
- You want safe hook management with proper unhooking

**Use base WoW API when:**
- You're writing a lightweight addon (under 200 lines) where the overhead isn't justified
- You need `EventRegistry` callbacks (Ace3 doesn't wrap these)
- You need `C_Timer.NewTicker()` with specific tick-count limits
- You want frame-level event registration for unit events with unit filters
- You need the simplicity of a single-file addon without library dependencies

**Honest trade-offs:**

| Ace3 | Base WoW API |
|------|-------------|
| Structured lifecycle, module system | Simpler, no dependencies |
| Auto-cleanup on disable | Manual cleanup required |
| Profile management built-in | Must build your own |
| Adds ~100-200KB to addon size | Zero overhead |
| One shared event frame for all registrations | Per-frame event registration, supports unit filters |
| Community standard, familiar to other devs | No learning curve for Lua devs |

---

## Section 2: AceAddon-3.0

AceAddon-3.0 provides addon object creation, lifecycle callbacks, and a module system. It is the foundation that all other Ace3 libraries build on.

### Creating an Addon

```lua
`LibStub("AceAddon-3.0"):NewAddon([object,] name [, lib, ...])`
```

- `object` (table, optional) -- An existing table (e.g., a frame) to use as the addon base
- `name` (string) -- Unique name for this addon. Errors if an addon with this name already exists
- `lib` (strings) -- Library names to embed into the addon

```lua
-- Simple addon with embedded libraries
local MyAddon = LibStub("AceAddon-3.0"):NewAddon("MyAddon", "AceEvent-3.0", "AceDB-3.0")

-- Addon based on an existing frame
local myFrame = CreateFrame("Frame")
local MyAddon = LibStub("AceAddon-3.0"):NewAddon(myFrame, "MyAddon", "AceEvent-3.0")
```

### Lifecycle Callbacks

AceAddon provides three lifecycle callbacks. You define these as methods on your addon object; AceAddon calls them automatically via `safecall` (errors in one addon don't break others).

#### `OnInitialize()`

Called during the `ADDON_LOADED` event, after your addon's files have finished loading. This is the first callback to fire. At this point, saved variables are available but the player may not be fully logged in yet.

**Use for:** Initializing your AceDB database, registering slash commands, setting up addon structure.

```lua
function MyAddon:OnInitialize()
    -- SavedVariables are available now
    self.db = LibStub("AceDB-3.0"):New("MyAddonDB", defaults, true)
    self:RegisterChatCommand("myaddon", "SlashHandler")
end
```

#### `OnEnable()`

Called during `PLAYER_LOGIN`, when the game world is loaded and most game data is available. If your addon loads after the player is already logged in (e.g., LoadOnDemand addons), OnEnable fires immediately after OnInitialize during the next `ADDON_LOADED` event.

**Use for:** Registering events, hooking functions, creating UI frames, starting timers.

```lua
function MyAddon:OnEnable()
    self:RegisterEvent("UNIT_HEALTH")
    self:RegisterEvent("PLAYER_REGEN_DISABLED", "OnEnterCombat")
    self:ScheduleRepeatingTimer("UpdateDisplay", 1.0)
end
```

**Important:** OnEnable will NOT fire if the addon's `enabledState` is false.

#### `OnDisable()`

Called only when your addon is **manually disabled** via `addon:Disable()`. This is NOT called on logout or UI reload.

When an addon is disabled, all embedded libraries also run their disable logic: AceEvent unregisters all events and messages, AceTimer cancels all timers, and AceHook unhooks all hooks. This happens automatically.

```lua
function MyAddon:OnDisable()
    -- Custom cleanup beyond what the libraries handle automatically
    if self.overlayFrame then
        self.overlayFrame:Hide()
    end
end
```

#### Lifecycle Summary

| Callback | Fires When | Game State |
|----------|-----------|------------|
| OnInitialize | ADDON_LOADED | SavedVars available, may not be logged in |
| OnEnable | PLAYER_LOGIN (or immediately if already logged in) | World loaded, game data available |
| OnDisable | Manual `addon:Disable()` call only | Any time |

**Gotcha:** OnInitialize fires for *every* ADDON_LOADED event, but AceAddon only calls your OnInitialize once -- it tracks which addons have been initialized via an internal queue. Modules are initialized in the same ADDON_LOADED pass as their parent, after the parent's OnInitialize.

### Module System

Modules let you break an addon into independently toggleable features. Each module is itself a full AceAddon object with its own lifecycle callbacks and embedded libraries.

#### `addon:NewModule(name [, prototype|lib, ...])`

- `name` (string) -- Unique name for this module (within the parent addon)
- `prototype` (table, optional) -- A table whose methods serve as a base class for the module
- `lib` (strings) -- Libraries to embed into the module

Internally, the module is registered with AceAddon as `"ParentName_ModuleName"`, but `module:GetName()` returns just `"ModuleName"`.

```lua
-- Create a module with its own embedded libraries
local CombatModule = MyAddon:NewModule("Combat", "AceEvent-3.0")

function CombatModule:OnEnable()
    self:RegisterEvent("PLAYER_REGEN_DISABLED")
    self:RegisterEvent("PLAYER_REGEN_ENABLED")
end

function CombatModule:PLAYER_REGEN_DISABLED()
    -- Handle entering combat
end

function CombatModule:PLAYER_REGEN_ENABLED()
    -- Handle leaving combat
end
```

#### `addon:GetModule(name [, silent])`

Returns the module object. Errors if the module doesn't exist, unless `silent` is true (then returns nil).

```lua
local combat = MyAddon:GetModule("Combat")
local optional = MyAddon:GetModule("MaybeExists", true)  -- returns nil if missing
```

#### Module Lifecycle

Modules follow the same OnInitialize/OnEnable/OnDisable pattern. Their lifecycle is linked to their parent:

1. Parent `OnInitialize` fires first
2. Module `OnInitialize` fires next (during the same ADDON_LOADED processing)
3. Parent `OnEnable` fires at PLAYER_LOGIN
4. Each module's `OnEnable` fires after the parent's (in creation order), if the module's `enabledState` is true
5. When the parent is disabled, all modules are disabled automatically

#### `addon:SetDefaultModuleState(state)`

Sets whether new modules start enabled (true) or disabled (false). Default is `true`. Must be called **before** any modules are created.

```lua
MyAddon:SetDefaultModuleState(false)  -- All new modules start disabled

local Combat = MyAddon:NewModule("Combat", "AceEvent-3.0")
-- Combat.enabledState is false; OnEnable won't fire until you call Combat:Enable()
```

#### `addon:SetDefaultModuleLibraries(lib, ...)`

Sets libraries that will be automatically embedded into every new module. Must be called **before** any modules are created.

```lua
MyAddon:SetDefaultModuleLibraries("AceEvent-3.0", "AceTimer-3.0")

-- This module automatically gets AceEvent and AceTimer, plus AceHook from its own args
local UI = MyAddon:NewModule("UI", "AceHook-3.0")
```

#### `addon:SetDefaultModulePrototype(prototype)`

Sets a table that serves as a base class (via `__index` metatable) for all new modules. Must be called **before** any modules are created.

```lua
local modulePrototype = {}

function modulePrototype:OnEnable()
    -- Default behavior for all modules
    if self.RegisterEvents then
        self:RegisterEvents()
    end
end

function modulePrototype:DebugPrint(msg)
    -- Shared utility available to all modules
end

MyAddon:SetDefaultModulePrototype(modulePrototype)
```

#### `addon:Enable()` / `addon:Disable()` / `addon:IsEnabled()`

Manually enable or disable an addon/module. `Enable()` sets `enabledState = true` and fires OnEnable. `Disable()` sets `enabledState = false` and fires OnDisable. Enabling a parent also enables all its modules (that have `enabledState = true`). Disabling a parent disables all modules.

```lua
-- Toggle a module at runtime
if MyAddon:GetModule("Combat"):IsEnabled() then
    MyAddon:GetModule("Combat"):Disable()
else
    MyAddon:GetModule("Combat"):Enable()
end
```

#### `addon:EnableModule(name)` / `addon:DisableModule(name)`

Convenience shortcuts for `addon:GetModule(name):Enable()` and `addon:GetModule(name):Disable()`.

#### `addon:GetName()`

Returns the "display name" of the addon or module. For modules, this returns the module name (e.g., `"Combat"`), not the internal registered name (e.g., `"MyAddon_Combat"`).

#### `addon:IterateModules()`

Returns a `pairs()` iterator over all modules: `name, moduleObject`.

```lua
for name, module in MyAddon:IterateModules() do
    if module:IsEnabled() then
        -- do something with enabled modules
    end
end
```

#### `AceAddon:GetAddon(name [, silent])`

Retrieve any AceAddon-registered addon by name. Useful for inter-addon communication.

```lua
local otherAddon = LibStub("AceAddon-3.0"):GetAddon("SomeOtherAddon", true)
if otherAddon then
    -- interact with it
end
```

---

## Section 3: AceDB-3.0

AceDB-3.0 manages saved variables with profile support, smart defaults, and multiple storage scopes.

### Creating a Database

```lua
`LibStub("AceDB-3.0"):New(tbl, defaults, defaultProfile)`
```

- `tbl` (string or table) -- Either the SavedVariables name (string, looked up in `_G`) or a direct table reference
- `defaults` (table, optional) -- Default values for the database
- `defaultProfile` (string or true, optional) -- The default profile name. Pass `true` to use a shared profile called `"Default"`. If omitted, the default profile is character-specific (`"CharName - RealmName"`)

```lua
-- In your .toc file:
-- ## SavedVariables: MyAddonDB

local defaults = {
    profile = {
        minimap = { hide = false },
        opacity = 1.0,
    },
    global = {
        totalLogins = 0,
    },
    char = {
        lastZone = "",
    },
}

function MyAddon:OnInitialize()
    -- "true" means all characters share a "Default" profile by default
    self.db = LibStub("AceDB-3.0"):New("MyAddonDB", defaults, true)

    -- Access data through the scope tables:
    self.db.profile.opacity = 0.8        -- profile-specific
    self.db.global.totalLogins = self.db.global.totalLogins + 1  -- account-wide
    self.db.char.lastZone = GetZoneText() -- character-specific
end
```

### Storage Scopes

Every scope creates a separate section in your saved variables, keyed by the appropriate identifier.

| Scope | Key | Shared Between |
|-------|-----|----------------|
| profile | User-chosen name | All chars using that profile |
| global | (single table) | All characters, account-wide |
| char | "Name - Realm" | Only this character |
| realm | "Realm" | All chars on this realm |
| class | "WARRIOR", etc. | All chars of this class |
| race | "Human", etc. | All chars of this race |
| faction | "Alliance"/"Horde" | All chars of this faction |
| factionrealm | "Faction - Realm" | Same faction + realm |
| factionrealmregion | "Faction - Realm - Region" | Same faction + realm + region |
| locale | "enus", etc. | All chars using this locale |

```lua
local defaults = {
    profile = { showTooltips = true },
    char = { lastPosition = {} },
    global = { debugMode = false },
    factionrealm = { auctionCache = {} },
}
```

### Defaults Wildcards: `["*"]` and `["**"]`

These are powerful features for providing defaults to dynamically-keyed tables.

#### `["*"]` -- Default for any new key at this level

When you access a key that doesn't exist in a `["*"]` section, AceDB automatically creates a new table with the `["*"]` defaults copied into it.

```lua
local defaults = {
    profile = {
        -- Every bar[N] gets these defaults automatically
        bars = {
            ["*"] = {
                enabled = true,
                width = 200,
                height = 20,
                color = { r = 1, g = 1, b = 1 },
            },
        },
    },
}

self.db = LibStub("AceDB-3.0"):New("MyDB", defaults)

-- Accessing any key auto-creates it with defaults:
local w = self.db.profile.bars.someNewBar.width  -- 200
local e = self.db.profile.bars[1].enabled        -- true
local h = self.db.profile.bars["special"].height -- 20
```

For non-table values, `["*"]` provides a default return value:

```lua
local defaults = {
    profile = {
        enabledModules = {
            ["*"] = true,  -- Any unknown key returns true
        },
    },
}
-- self.db.profile.enabledModules["SomeModule"] returns true by default
```

#### `["**"]` -- Recursive default for all sub-tables at any depth

`["**"]` works like `["*"]` but also applies its values to **explicitly named** sub-tables at the same level. It is inherited downward.

```lua
local defaults = {
    profile = {
        units = {
            -- These defaults apply to ALL unit entries, even named ones
            ["**"] = {
                showHealth = true,
                showPower = true,
                scale = 1.0,
            },
            -- "player" also gets showHealth, showPower, scale from ["**"]
            -- plus its own specific overrides
            player = {
                showPower = false,  -- override the ** default
                showCastBar = true, -- additional setting
            },
            target = {
                showBuffs = true,
            },
        },
    },
}

-- self.db.profile.units.player.showHealth  => true  (from **)
-- self.db.profile.units.player.showPower   => false (explicit override)
-- self.db.profile.units.player.showCastBar => true  (explicit)
-- self.db.profile.units.target.scale       => 1.0   (from **)
-- self.db.profile.units.focus.showHealth   => true   (from ** via dynamic creation)
```

### Profile Management

#### `db:GetCurrentProfile()`

Returns the name of the currently active profile.

```lua
local currentProfile = self.db:GetCurrentProfile()
self:Print("Active profile: " .. currentProfile)
```

#### `db:SetProfile(name)`

Switches to a different profile. Creates it if it doesn't exist. Fires `OnProfileShutdown` (old profile), then `OnProfileChanged`.

```lua
self.db:SetProfile("PvP")
```

#### `db:GetProfiles(tbl)`

Returns a table of all profile names. Optionally reuses an existing table to avoid garbage.

```lua
local profiles = self.db:GetProfiles()
for i, name in ipairs(profiles) do
    -- list all profile names
end
```

Note: The return is `tbl, count` -- the second return value is the number of profiles.

#### `db:DeleteProfile(name [, silent])`

Deletes a profile. Cannot delete the active profile (errors). If `silent` is true, won't error when the profile doesn't exist. Switches any characters using the deleted profile back to the default.

```lua
self.db:DeleteProfile("OldProfile")
```

#### `db:CopyProfile(name [, silent])`

Copies all settings from the named profile into the **current** profile, overwriting existing values. Resets the current profile first, then copies.

```lua
-- Copy settings from "Healer" into the current profile
self.db:CopyProfile("Healer")
```

#### `db:ResetProfile([noChildren [, noCallbacks]])`

Resets the current profile to default values. Wipes all stored data and re-applies defaults.

- `noChildren` (boolean) -- If true, doesn't propagate the reset to namespace databases
- `noCallbacks` (boolean) -- If true, doesn't fire OnProfileReset

```lua
self.db:ResetProfile()
```

#### `db:ResetDB([defaultProfile])`

**Nuclear option.** Wipes the ENTIRE database (all profiles, all scopes, all namespaces) and re-initializes from scratch.

```lua
self.db:ResetDB("Default")
```

### Callbacks

AceDB fires callbacks when profile-related events occur. Register for them to update your UI or internal state when the user changes profiles.

```lua
function MyAddon:OnInitialize()
    self.db = LibStub("AceDB-3.0"):New("MyAddonDB", defaults, true)

    -- Register callbacks
    self.db.RegisterCallback(self, "OnProfileChanged", "RefreshConfig")
    self.db.RegisterCallback(self, "OnProfileCopied", "RefreshConfig")
    self.db.RegisterCallback(self, "OnProfileReset", "RefreshConfig")
end

function MyAddon:RefreshConfig()
    -- Re-read all settings from self.db.profile and update UI
    self:UpdateFramePositions()
    self:UpdateBarColors()
end
```

| Callback | Args (after self) | Fires When |
|----------|------------------|------------|
| OnProfileChanged | db, newProfileKey | Profile switched |
| OnProfileDeleted | db, profileKey | Profile deleted |
| OnProfileCopied | db, sourceProfileKey | Profile copied from another |
| OnProfileReset | db | Current profile reset to defaults |
| OnNewProfile | db, profileKey | New profile created (first access) |
| OnProfileShutdown | db | About to leave current profile |
| OnDatabaseShutdown | db | PLAYER_LOGOUT, before defaults stripped |
| OnDatabaseReset | db | Entire database wiped via ResetDB |

### Namespaces

Namespaces let modules have their own independent data section within the parent addon's saved variable. A namespace is a full AceDB object tied to the parent's profile -- when the parent switches profiles, all namespaces switch too.

```lua
function MyAddon:OnInitialize()
    self.db = LibStub("AceDB-3.0"):New("MyAddonDB", defaults, true)
end

-- In a module:
function CombatModule:OnInitialize()
    local db = MyAddon.db:RegisterNamespace("Combat", {
        profile = {
            showDamageNumbers = true,
            fontSize = 14,
        },
    })
    self.db = db
    -- Access via self.db.profile.showDamageNumbers
end
```

#### `db:RegisterNamespace(name, defaults)`

Creates and returns a namespace database object. The namespace shares the parent's profile selection. You can only call `RegisterDefaults` and `ResetProfile` on namespace objects -- profile management methods like `SetProfile` are only available on the parent.

#### `db:GetNamespace(name [, silent])`

Returns a previously registered namespace.

```lua
local ns = self.db:GetNamespace("Combat", true)  -- nil if not registered
```

### Default Stripping on Logout

**Important:** AceDB removes all default values from saved variables during `PLAYER_LOGOUT`. This minimizes the size of your SavedVariables file by only saving values the user has actually changed. When the addon loads again, defaults are re-applied via metatables.

This means you should never store references to subtables across sessions, as the table structure is rebuilt on each load. Always access data through `self.db.profile.xxx`.

---

## Section 4: AceDBOptions-3.0

AceDBOptions-3.0 generates a ready-made AceConfig options table for profile management. It provides a UI panel where users can create, switch, copy, delete, and reset profiles.

### `LibStub("AceDBOptions-3.0"):GetOptionsTable(db [, noDefaultProfiles])`

- `db` -- Your AceDB database object
- `noDefaultProfiles` (boolean, optional) -- If true, don't show pre-generated profile suggestions (character name, realm, class, "Default")

Returns an AceConfig-compatible options table (type = "group").

```lua
function MyAddon:OnInitialize()
    self.db = LibStub("AceDB-3.0"):New("MyAddonDB", defaults, true)

    -- Your addon's main options
    local options = {
        name = "MyAddon",
        type = "group",
        args = {
            general = {
                name = "General",
                type = "group",
                args = {
                    opacity = {
                        name = "Opacity",
                        type = "range",
                        min = 0, max = 1, step = 0.05,
                        get = function() return self.db.profile.opacity end,
                        set = function(_, val) self.db.profile.opacity = val end,
                    },
                },
            },
            -- Add the profiles panel as a sub-group
            profiles = LibStub("AceDBOptions-3.0"):GetOptionsTable(self.db),
        },
    }

    LibStub("AceConfig-3.0"):RegisterOptionsTable("MyAddon", options)
    LibStub("AceConfigDialog-3.0"):AddToBlizOptions("MyAddon", "MyAddon")
end
```

This gives users a panel with:
- Current profile display
- Profile selection dropdown (with suggestions for character, realm, class, "Default")
- Text box to create a new profile
- Copy from another profile
- Delete unused profiles
- Reset current profile to defaults

---

## Section 5: AceEvent-3.0

AceEvent-3.0 provides event registration and inter-addon messaging using a single shared frame and CallbackHandler-1.0 for dispatch.

### `self:RegisterEvent(event [, callback [, arg]])`

Registers for a Blizzard event.

- `event` (string) -- The event name (e.g., `"PLAYER_REGEN_DISABLED"`)
- `callback` (string or function, optional) -- A method name on `self`, or a function reference. If omitted, defaults to a method named the same as the event
- `arg` (any, optional) -- If provided, this is passed as the first argument to the callback instead of `self`

```lua
function MyAddon:OnEnable()
    -- Default callback: calls self:UNIT_HEALTH(event, ...)
    self:RegisterEvent("UNIT_HEALTH")

    -- Named method callback: calls self:OnCombatStart(event, ...)
    self:RegisterEvent("PLAYER_REGEN_DISABLED", "OnCombatStart")

    -- Function reference callback
    self:RegisterEvent("PLAYER_REGEN_ENABLED", function(event)
        -- Note: when using function refs, 'self' is not automatically passed
        -- unless you capture it via closure
    end)
end

function MyAddon:UNIT_HEALTH(event, unit)
    if unit == "player" then
        -- update player health display
    end
end

function MyAddon:OnCombatStart(event)
    -- entering combat
end
```

**Gotcha:** When you use the default callback (method name = event name), the first argument to the callback is `event` (the event name), followed by event-specific arguments. The `self` parameter is implicit because it's a method call.

### `self:UnregisterEvent(event)`

Stops receiving the specified event.

```lua
self:UnregisterEvent("UNIT_HEALTH")
```

### `self:UnregisterAllEvents()`

Unregisters all events for this addon/module. Called automatically when the addon is disabled.

### Custom Messages (Inter-Addon Communication)

AceEvent provides a message bus for addon-to-addon communication within the same client. These are **not** sent over the network.

#### `self:RegisterMessage(message [, callback [, arg]])`

Same signature as `RegisterEvent`, but for custom message names.

```lua
-- Addon A: listen for a message
function AddonA:OnEnable()
    self:RegisterMessage("MyAddon_SettingsChanged", "OnSettingsChanged")
end

function AddonA:OnSettingsChanged(message, ...)
    -- react to settings change
end
```

#### `self:SendMessage(message, ...)`

Broadcasts a message to all registered listeners.

```lua
-- Addon B: send a message
function AddonB:UpdateSettings()
    -- ... change settings ...
    self:SendMessage("MyAddon_SettingsChanged", "opacity", 0.8)
end
```

#### `self:UnregisterMessage(message)` / `self:UnregisterAllMessages()`

Stop listening for a specific message, or all messages. `UnregisterAllMessages()` is called automatically on disable.

### How It Works Internally

AceEvent uses a **single shared frame** (`AceEvent30Frame`) to register for all Blizzard events. When any event fires, the frame's OnEvent handler calls `events:Fire(event, ...)` which dispatches to all registered callbacks via CallbackHandler-1.0. This means:

- All AceEvent registrations across all addons share one frame
- The dispatch overhead is a function call per registered handler per event
- Events are only registered on the frame when at least one handler exists, and unregistered when the last handler is removed

### AceEvent vs. Base API vs. EventRegistry

| Feature | AceEvent | frame:RegisterEvent | EventRegistry |
|---------|----------|-------------------|---------------|
| Auto-cleanup on disable | Yes | No (manual) | No (manual) |
| Unit event filters | No | Yes | Depends on event |
| Inter-addon messages | Yes (SendMessage) | No | No |
| Per-frame registration | No (shared frame) | Yes | N/A |
| 12.0+ modern events | No | N/A | Yes |

**Use AceEvent when** you want the auto-cleanup and inter-addon messaging features, and don't need unit event filters.

**Use `frame:RegisterUnitEvent()` when** you need to filter events to specific units (e.g., only "player" and "target" for UNIT_HEALTH). AceEvent cannot do this.

**Use EventRegistry when** you need callbacks for newer Blizzard systems that fire through EventRegistry rather than the frame event system (e.g., `EventRegistry:RegisterCallback("EditMode.Enter", ...)`), OR when you want to register for plain frame events without managing your own frame. The latter is the modern idiomatic pattern in 12.0.0+ core UI (used throughout `Blizzard_ActionBar`, `Blizzard_DamageMeter`, `Blizzard_CooldownViewer`, etc.):

```lua
-- Register a frame event and get a callback fired through EventRegistry.
-- No user-supplied frame needed; EventRegistry owns its internal frame.
EventRegistry:RegisterFrameEventAndCallback("PLAYER_REGEN_DISABLED", self.OnCombatStart, self)
EventRegistry:RegisterFrameEventAndCallback("SPELLS_CHANGED", MyAddon.OnSpellsChanged, MyAddon)

-- A plain anonymous callback (no owner) is also supported:
EventRegistry:RegisterFrameEventAndCallback("PLAYER_ENTERING_WORLD", function()
    -- handler body
end)
```

---

## Section 6: AceConsole-3.0

AceConsole-3.0 provides slash command registration, chat output, and argument parsing.

### `self:RegisterChatCommand(command, func [, persist])`

Registers a slash command.

- `command` (string) -- The command name **without** the leading `/`
- `func` (string or function) -- A method name on `self`, or a function reference
- `persist` (boolean, optional) -- If false, the command is unregistered when the addon is disabled and re-registered when enabled. Default is true (persists across enable/disable cycles)

```lua
function MyAddon:OnInitialize()
    self:RegisterChatCommand("myaddon", "SlashCommand")
    self:RegisterChatCommand("ma", "SlashCommand")  -- alias
end

function MyAddon:SlashCommand(input)
    if input == "config" then
        Settings.OpenToCategory("MyAddon")
    elseif input == "toggle" then
        self.db.profile.enabled = not self.db.profile.enabled
    else
        self:Print("Usage: /myaddon config | toggle")
    end
end
```

**Note:** The slash command is stored globally as `SLASH_ACECONSOLE_MYADDON1 = "/myaddon"`. The handler is stored in `SlashCmdList["ACECONSOLE_MYADDON"]`.

### `self:UnregisterChatCommand(command)`

Unregisters a slash command.

```lua
self:UnregisterChatCommand("myaddon")
```

### `self:Print(...)`

Prints to `DEFAULT_CHAT_FRAME` with the addon name prefixed in green. You can optionally pass a specific chat frame as the first argument.

```lua
self:Print("Addon loaded!")
-- Output: |cff33ff99MyAddon|r: Addon loaded!

-- Print to a specific chat frame
self:Print(ChatFrame2, "This goes to chat tab 2")
```

### `self:Printf(format, ...)`

Same as `Print` but accepts `string.format` arguments.

```lua
self:Printf("Loaded %d profiles in %.2f seconds", count, elapsed)
```

### `self:GetArgs(str, numargs, startpos)`

Parses space-separated arguments from a string, correctly handling quoted strings and item links (which contain spaces and pipe characters).

- `str` (string) -- The input string (typically the slash command input)
- `numargs` (number, optional) -- How many arguments to extract. Default 1
- `startpos` (number, optional) -- Starting position in the string. Default 1
- **Returns:** `arg1, arg2, ..., nextpos` -- The extracted arguments plus the position after the last argument. Returns nil for missing arguments and `1e9` when the end of string is reached.

```lua
function MyAddon:SlashCommand(input)
    local cmd, next = self:GetArgs(input, 1)

    if cmd == "set" then
        local key, value, _ = self:GetArgs(input, 2, next)
        self:Printf("Setting %s = %s", key or "nil", value or "nil")
    elseif cmd == "note" then
        -- Get everything after "note" as one argument
        local text = self:GetArgs(input, 1, next)
        self.db.profile.note = text
    end
end

-- /myaddon set opacity 0.8
-- cmd = "set", key = "opacity", value = "0.8"

-- /myaddon note "this is a quoted string"
-- cmd = "note", text = "this is a quoted string"
```

**Gotcha:** `GetArgs` handles both single-quoted and double-quoted strings. It also correctly skips past `|Hitem:...|h[Item Name]|h` hyperlinks without splitting on the spaces inside them.

---

## Section 7: AceTimer-3.0

AceTimer-3.0 provides one-shot and repeating timers with automatic cancellation on addon disable.

### `self:ScheduleTimer(func, delay, ...)`

Schedules a one-shot timer that fires once after `delay` seconds.

- `func` (string or function) -- Method name on `self`, or a function reference
- `delay` (number) -- Delay in seconds (minimum 0.01)
- `...` -- Optional arguments passed to the callback
- **Returns:** A timer handle (table) used for cancellation and time queries

```lua
function MyAddon:OnEnable()
    -- Using a method name
    self:ScheduleTimer("DelayedInit", 2.0)

    -- Using a function reference with extra arguments
    self:ScheduleTimer(function(greeting)
        -- Note: 'self' is not passed automatically for function refs
        -- 'greeting' receives "Hello after 5 seconds!" at fire time
    end, 5.0, "Hello after 5 seconds!")
end

function MyAddon:DelayedInit()
    -- runs once, 2 seconds after OnEnable
end
```

### `self:ScheduleRepeatingTimer(func, delay, ...)`

Schedules a repeating timer that fires every `delay` seconds until canceled.

- Same arguments and return value as `ScheduleTimer`
- The timer compensates for frame rate variations to maintain an accurate average interval

```lua
function MyAddon:OnEnable()
    self.updateTimer = self:ScheduleRepeatingTimer("PeriodicUpdate", 1.0)
end

function MyAddon:PeriodicUpdate()
    -- runs every ~1 second
    self:UpdateDisplay()
end
```

### `self:CancelTimer(handle)`

Cancels a timer. Returns `true` if the timer was found and canceled, `false` if the handle was invalid or already expired/canceled.

```lua
function MyAddon:OnDisable()
    -- Explicit cancel (though AceTimer auto-cancels all timers on disable)
    if self.updateTimer then
        self:CancelTimer(self.updateTimer)
        self.updateTimer = nil
    end
end
```

### `self:CancelAllTimers()`

Cancels all timers registered by this addon/module. Called automatically when the addon is disabled.

### `self:TimeLeft(handle)`

Returns the seconds remaining on a timer. Returns 0 for invalid or expired handles.

```lua
local remaining = self:TimeLeft(self.updateTimer)
self:Printf("Next update in %.1f seconds", remaining)
```

### Internal Implementation

Since version 17, AceTimer uses `C_Timer.After()` internally (not OnUpdate frames or animation groups). Each timer creates a closure that wraps `C_Timer.After()`. Repeating timers re-schedule themselves in the callback with delay compensation to maintain accuracy.

The minimum delay is **0.01 seconds** (enforced by the C_Timer API).

### AceTimer vs. C_Timer

| Feature | AceTimer | C_Timer.After | C_Timer.NewTicker |
|---------|----------|--------------|-------------------|
| Auto-cancel on disable | Yes | No | No (must call `:Cancel()`) |
| Cancel by handle | Yes | No (no handle returned) | Yes |
| Time remaining query | Yes | No | No |
| Repeating with compensation | Yes | No (one-shot only) | Yes |
| Tick count limit | No | N/A | Yes (N iterations then stop) |
| Pass extra arguments | Yes | No (use closure) | No (use closure) |
| Method name callbacks | Yes ("MethodName") | No | No |

**Use AceTimer when** you want auto-cleanup, time-remaining queries, or method-name callbacks.

**Use `C_Timer.After()` when** you need a simple one-shot delay with no cleanup concerns.

**Use `C_Timer.NewTicker()` when** you need a timer that runs exactly N times, or when you're not using AceAddon.

---

## Section 8: AceHook-3.0

AceHook-3.0 provides safe hooking and unhooking of global functions, object methods, and frame scripts. All hooks are automatically removed when the addon is disabled.

### Hook Types Overview

AceHook offers three types of function/method hooks, plus script equivalents:

| Type | When Called | Can Modify Args/Return? | Can Block Original? | Stores Original? |
|------|-----------|------------------------|--------------------|-----------------| 
| Hook (safe) | BEFORE original | No | No | No |
| SecureHook | AFTER original | No | No | No |
| RawHook | REPLACES original | Yes | Yes | Yes, in self.hooks |

### `self:Hook([object,] method [, handler [, hookSecure]])`

Creates a **pre-hook** (safe hook). Your handler is called BEFORE the original function, then the original function runs automatically. You cannot modify arguments, return values, or prevent the original from executing.

- `object` (table, optional) -- The object whose method to hook. Omit for global functions
- `method` (string) -- The method/function name
- `handler` (string or function, optional) -- Callback. Defaults to a method named the same as `method`
- `hookSecure` (boolean, optional) -- If true, allows hooking secure functions (use with caution)

```lua
function MyAddon:OnEnable()
    -- Hook a global function
    self:Hook("ChatFrame_OnHyperlinkShow", "OnHyperlinkClick")

    -- Hook an object method
    self:Hook(GameTooltip, "SetUnit", "OnTooltipSetUnit")
end

function MyAddon:OnHyperlinkClick(chatFrame, link, text, button)
    -- Called BEFORE the original ChatFrame_OnHyperlinkShow
    -- Original runs automatically afterward
end

function MyAddon:OnTooltipSetUnit(tooltip, ...)
    -- Called BEFORE GameTooltip:SetUnit()
end
```

**Important:** Hook (safe hook) errors if the target is a secure function. Pass `true` as the last argument to override this check, or use `SecureHook` instead.

### `self:SecureHook([object,] method [, handler])`

Creates a **post-hook** using the WoW API's `hooksecurefunc()`. Your handler is called AFTER the original function. You cannot modify arguments, return values, or prevent execution. Safe to use on secure functions without causing taint.

```lua
function MyAddon:OnEnable()
    -- Hook after a secure function runs
    self:SecureHook("UseAction", "OnUseAction")

    -- Hook after an object method
    self:SecureHook(SpellFlyout, "Toggle", "OnSpellFlyoutToggle")
end

function MyAddon:OnUseAction(slot, cursor, selfCast)
    -- Called AFTER UseAction() has already run
    self:UpdateCooldownDisplay(slot)
end
```

### `self:RawHook([object,] method [, handler [, hookSecure]])`

Creates a **raw hook** that completely replaces the original function. The original is stored in `self.hooks[method]` (for globals) or `self.hooks[object][method]` (for object methods). **You are responsible for calling the original** if you want it to run.

```lua
function MyAddon:OnEnable()
    -- RawHook a global function
    self:RawHook("GetQuestReward", "OnGetQuestReward")

    -- RawHook an object method
    self:RawHook(GameTooltip, "SetUnit", "OnTooltipSetUnit")
end

function MyAddon:OnGetQuestReward(choice)
    -- Do something before the original
    self:LogQuestCompletion(choice)

    -- Call the original (REQUIRED if you want normal behavior)
    return self.hooks.GetQuestReward(choice)
end

function MyAddon:OnTooltipSetUnit(tooltip, ...)
    -- Conditionally call the original
    if self.db.profile.showTooltips then
        self.hooks[tooltip].SetUnit(tooltip, ...)
    end
    -- Add custom tooltip content
end
```

**Critical:** In `self.hooks[object][method]`, you must pass the object as the first argument yourself when calling the original. AceHook stores the raw function reference, not a bound method.

### Script Hooks

Frame scripts have equivalent hook methods. The `frame` argument is required (not optional like `object` for function hooks).

#### `self:HookScript(frame, script [, handler])`

Pre-hook for frame scripts. Your handler fires BEFORE the original script.

```lua
self:HookScript(PlayerFrame, "OnEnter", "OnPlayerFrameEnter")
```

#### `self:SecureHookScript(frame, script [, handler])`

Post-hook for frame scripts (uses `frame:HookScript()` internally). Fires AFTER the original.

```lua
self:SecureHookScript(PlayerFrame, "OnEnter", "OnPlayerFrameEnter")
```

#### `self:RawHookScript(frame, script [, handler])`

Replaces the frame script entirely. Original stored in `self.hooks[frame][script]`.

```lua
function MyAddon:OnEnable()
    self:RawHookScript(PlayerFrame, "OnEnter", "OnPlayerFrameEnter")
end

function MyAddon:OnPlayerFrameEnter(frame, motion)
    -- Custom behavior
    if self.db.profile.customTooltip then
        self:ShowCustomTooltip(frame)
    else
        -- Call original script
        self.hooks[frame].OnEnter(frame, motion)
    end
end
```

**Note:** `HookScript` and `RawHookScript` refuse to hook the `OnClick` script of protected frames. Use `SecureHookScript` for secure script handlers.

### `self:Unhook([object,] method)`

Removes a hook and restores the original function/script. For secure hooks (SecureHook/SecureHookScript), the hook is deactivated but cannot truly be removed due to how `hooksecurefunc` works -- it simply stops calling your handler.

```lua
self:Unhook("GetQuestReward")
self:Unhook(GameTooltip, "SetUnit")
self:Unhook(PlayerFrame, "OnEnter")
```

### `self:UnhookAll()`

Removes all hooks for this addon/module. Called automatically on disable.

### `self:IsHooked([object,] method)`

Returns `isHooked, handler` -- whether the function is currently hooked, and the handler being used.

```lua
local hooked, handler = self:IsHooked(GameTooltip, "SetUnit")
if not hooked then
    self:SecureHook(GameTooltip, "SetUnit", "OnSetUnit")
end
```

### The `self.hooks` Table

When using `RawHook` or `RawHookScript`, the original function/script is stored in `self.hooks`:

```lua
-- For global functions:
self.hooks["FunctionName"]          -- the original function

-- For object methods:
self.hooks[object]["MethodName"]    -- the original function

-- For frame scripts:
self.hooks[frame]["ScriptName"]     -- the original script handler
```

This table is created by AceHook when it embeds into your addon. The entries are cleaned up when you unhook.

### AceHook vs. `hooksecurefunc()`

| Feature | AceHook | hooksecurefunc |
|---------|---------|---------------|
| Auto-unhook on disable | Yes | No (permanent) |
| Can unhook at will | Yes (non-secure hooks) | No |
| Pre-hooks | Yes (Hook, HookScript) | No |
| Raw replacement hooks | Yes (RawHook) | No |
| Taint-safe post-hooks | Yes (SecureHook) | Yes |
| Tracks hook state | Yes (IsHooked) | No |
| Stores original for you | Yes (self.hooks) | No |

**Use AceHook when** you need managed hooks with cleanup, especially with RawHook for argument/return modification.

**Use `hooksecurefunc()` directly when** you need a permanent post-hook on a secure function and don't need AceAddon lifecycle integration, or when hooking from code that doesn't use AceAddon.

**Warning about SecureHook/SecureHookScript:** While `Unhook()` stops your handler from being called, the underlying `hooksecurefunc` hook remains permanently. This is a WoW API limitation, not an AceHook limitation. The residual hook has negligible performance impact (it calls an empty check function).

---

## AceConfig-3.0 -- Declarative Options Tables

AceConfig-3.0 is a thin wrapper that ties together AceConfigRegistry (stores options tables) and AceConfigCmd (slash command handling). You declare your addon's settings as a single Lua table, and AceConfig generates both a GUI dialog and slash command interface from that declaration.

### Registering an Options Table

```lua
local AceConfig = LibStub("AceConfig-3.0")

AceConfig:RegisterOptionsTable("MyAddon", optionsTable, {"/myaddon", "/ma"})
```

**Parameters:**
- `appName` (string) -- Unique identifier for this options table. Used by AceConfigDialog to reference it.
- `options` (table or function) -- The options table, or a function that returns one. Functions receive `(uiType, uiName, appName)` where uiType is `"cmd"`, `"dropdown"`, or `"dialog"`.
- `slashcmd` (string or table or nil) -- Slash command(s) to register. Pass `nil` to skip slash command creation.

The root of every options table must be `type = "group"`.

### Options Table Structure

Every node in an options table requires at minimum a `type` and a `name`. The root must be a group.

```lua
local options = {
    type = "group",
    name = "My Addon Settings",
    args = {
        -- child options go here, keyed by unique string IDs
    },
}
```

### Common Fields (All Types)

These fields are valid on every option type:

| Field | Type | Description |
|-------|------|-------------|
| type | string | **Required.** The option type. |
| name | string or function | **Required.** Display name. Functions receive `(info)`. |
| desc | string or function | Tooltip description. |
| order | number or function | Sort order (lower = earlier). Defaults to alphabetical by key. |
| get | string or function | Getter. String = method name on handler. Function receives `(info)`. |
| set | string or function | Setter. String = method name on handler. Function receives `(info, value)`. |
| func | string or function | For `execute` type. The function to call when clicked. |
| validate | string or function or false | Validation before set. Return string for error message, `true` for OK. `false` disables. |
| confirm | string or function or boolean | Show confirmation dialog before set. `true` uses default text. |
| confirmText | string | Custom confirmation dialog text. |
| disabled | string or function or boolean | Grays out the option. |
| hidden | string or function or boolean | Hides from all UIs. |
| guiHidden | boolean | Hides from dialog GUI only. |
| dialogHidden | boolean | Hides from dialog GUI only (same effect as guiHidden). |
| cmdHidden | boolean | Hides from slash command only. |
| dropdownHidden | boolean | Hides from dropdown UI only. |
| width | string or number | Control width: `"half"`, `"double"`, `"full"`, or a numeric multiplier. Default is `"normal"` (1x = 170px). |
| handler | table | Object to call string-type get/set/func methods on. Inherited from parent groups. |
| arg | any | Arbitrary user data stored in `info.arg`. |
| icon | string, number, or function | Icon texture path or fileID. |
| iconCoords | table or function | Texture coordinates for icon `{left, right, top, bottom}`. |
| tooltipHyperlink | string or function | A hyperlink to attach to the tooltip. |

**The `info` table:** Every get/set/validate/confirm/func callback receives an `info` table as its first argument. Key fields:

- `info[1], info[2], ...` -- The path of option keys from root to this node.
- `info.options` -- The root options table.
- `info.option` -- The specific option node.
- `info.arg` -- The `arg` field from this option.
- `info.handler` -- The resolved handler object.
- `info.type` -- The option type string.
- `info.uiType` -- `"dialog"`, `"cmd"`, or `"dropdown"`.
- `info.appName` -- The appName this table was registered with.

### Option Type: group

Groups are containers. The `args` table holds child options keyed by string identifiers.

```lua
general = {
    type = "group",
    name = "General",
    order = 1,
    args = {
        enable = {
            type = "toggle",
            name = "Enable",
            order = 1,
            get = function(info) return MyAddonDB.enabled end,
            set = function(info, val) MyAddonDB.enabled = val end,
        },
    },
},
```

**Group-specific fields:**

- `args` (table) -- **Required.** Child option nodes.
- `childGroups` (string) -- How child groups render: `"tree"` (default, tree view), `"tab"` (tab bar), or `"select"` (dropdown selector).
- `inline` (boolean) -- Renders the group inline (bordered section) rather than as a navigable node. Also available as `dialogInline`, `cmdInline`, `guiInline`, `dropdownInline` for per-UI control.
- `plugins` (table) -- Additional args injected by other modules: `plugins = { ["moduleName"] = { key = optionNode, ... } }`.

**childGroups examples:**

```lua
-- Tab-style child groups
settings = {
    type = "group",
    name = "Settings",
    childGroups = "tab",
    args = {
        display = {
            type = "group",
            name = "Display",
            order = 1,
            args = { --[[ ... ]] },
        },
        behavior = {
            type = "group",
            name = "Behavior",
            order = 2,
            args = { --[[ ... ]] },
        },
    },
},

-- Inline group (rendered as a bordered section within its parent)
appearance = {
    type = "group",
    name = "Appearance",
    inline = true,
    order = 3,
    args = {
        scale = {
            type = "range",
            name = "Scale",
            min = 0.5, max = 2.0, step = 0.1,
            order = 1,
        },
    },
},
```

**Tree vs Tab vs Select behavior:**
- `"tree"` (default) -- All descendant groups become nodes in a tree view on the left side. Groups with `inline = true` are rendered inline within their parent instead of becoming tree nodes.
- `"tab"` -- Direct child groups become tabs. Grandchild groups default to inline unless they specify otherwise.
- `"select"` -- Same as tab, but uses a dropdown selector instead of tab buttons.

### Option Type: toggle

A boolean checkbox.

```lua
showMinimap = {
    type = "toggle",
    name = "Show Minimap Icon",
    desc = "Display the addon icon on the minimap.",
    order = 1,
    get = function(info) return MyAddonDB.minimap end,
    set = function(info, val) MyAddonDB.minimap = val end,
},
```

**Toggle-specific fields:**
- `tristate` (boolean) -- Cycles through `true` / `nil` / `false` (three states).
- `image` (string, number, or function) -- Icon displayed beside the checkbox.
- `imageCoords` (table or function) -- Texture coords for the image.

**Tristate example:**

```lua
filterMode = {
    type = "toggle",
    name = "Filter Mode",
    desc = "Check = show, X = hide, empty = default.",
    tristate = true,
    get = function(info) return MyAddonDB.filter end,  -- true, false, or nil
    set = function(info, val) MyAddonDB.filter = val end,
},
```

### Option Type: range

A slider for numeric values.

```lua
opacity = {
    type = "range",
    name = "Opacity",
    desc = "Set the frame opacity.",
    min = 0,
    max = 1,
    step = 0.05,
    isPercent = true,
    order = 2,
    get = function(info) return MyAddonDB.opacity end,
    set = function(info, val) MyAddonDB.opacity = val end,
},
```

**Range-specific fields:**

| Field | Type | Description |
|-------|------|-------------|
| min | number | Hard minimum value. |
| max | number | Hard maximum value. |
| softMin | number | Slider minimum (user can type below down to `min`). |
| softMax | number | Slider maximum (user can type above up to `max`). |
| step | number | Increment per step. |
| bigStep | number | Increment when holding modifier key. |
| isPercent | boolean | Displays values as percentages (0.5 shows as "50%"). |

**softMin/softMax example:**

```lua
fontSize = {
    type = "range",
    name = "Font Size",
    min = 4,       -- absolute minimum (manual entry)
    max = 72,      -- absolute maximum (manual entry)
    softMin = 8,   -- slider minimum
    softMax = 32,  -- slider maximum
    step = 1,
    order = 3,
    get = function(info) return MyAddonDB.fontSize end,
    set = function(info, val) MyAddonDB.fontSize = val end,
},
```

### Option Type: input

A text input field.

```lua
customText = {
    type = "input",
    name = "Custom Text",
    desc = "Enter the text to display on the frame.",
    order = 4,
    get = function(info) return MyAddonDB.customText end,
    set = function(info, val) MyAddonDB.customText = val end,
},
```

**Input-specific fields:**
- `pattern` (string) -- Lua pattern to validate input. Input is rejected if it does not match.
- `usage` (string) -- Usage hint displayed when pattern validation fails.
- `multiline` (boolean or number) -- `true` for a multi-line text area. A number sets the approximate height in lines (e.g., `multiline = 5`).

**Multi-line with validation example:**

```lua
luaCode = {
    type = "input",
    name = "Custom Lua Expression",
    desc = "Enter a Lua expression that returns a string.",
    multiline = 8,
    width = "full",
    order = 5,
    pattern = "^[%w%s%p]+$",
    usage = "Only printable characters are allowed.",
    get = function(info) return MyAddonDB.luaExpr end,
    set = function(info, val) MyAddonDB.luaExpr = val end,
},
```

### Option Type: select

A dropdown selection from a list of values.

```lua
frameAnchor = {
    type = "select",
    name = "Anchor Point",
    desc = "Where the frame anchors on screen.",
    values = {
        ["TOPLEFT"] = "Top Left",
        ["TOP"] = "Top",
        ["TOPRIGHT"] = "Top Right",
        ["LEFT"] = "Left",
        ["CENTER"] = "Center",
        ["RIGHT"] = "Right",
        ["BOTTOMLEFT"] = "Bottom Left",
        ["BOTTOM"] = "Bottom",
        ["BOTTOMRIGHT"] = "Bottom Right",
    },
    sorting = {"TOPLEFT", "TOP", "TOPRIGHT", "LEFT", "CENTER", "RIGHT", "BOTTOMLEFT", "BOTTOM", "BOTTOMRIGHT"},
    order = 6,
    get = function(info) return MyAddonDB.anchor end,
    set = function(info, val) MyAddonDB.anchor = val end,
},
```

**Select-specific fields:**
- `values` (table or function) -- Key-value pairs where keys are the stored values and values are display strings. Can also be a function returning a table: `function(info) return tbl end`.
- `sorting` (table or function) -- Array of keys defining display order. Without this, order is undefined (Lua table iteration).
- `style` (string) -- `"dropdown"` (default) or `"radio"` (renders as radio buttons instead of a dropdown).
- `itemControl` (string) -- Override the AceGUI widget type used for individual items.

**Radio style example:**

```lua
difficulty = {
    type = "select",
    name = "Difficulty",
    style = "radio",
    values = {
        easy = "Easy",
        normal = "Normal",
        hard = "Hard",
    },
    sorting = {"easy", "normal", "hard"},
    order = 7,
    get = function(info) return MyAddonDB.difficulty end,
    set = function(info, val) MyAddonDB.difficulty = val end,
},
```

### Option Type: multiselect

Multiple checkboxes from a values table. The get callback receives the key as a second argument; set receives key and state.

```lua
enabledModules = {
    type = "multiselect",
    name = "Enabled Modules",
    desc = "Toggle individual modules on/off.",
    values = {
        nameplates = "Nameplates",
        castbars = "Cast Bars",
        buffs = "Buff Tracking",
        threat = "Threat Indicators",
    },
    order = 8,
    get = function(info, key) return MyAddonDB.modules[key] end,
    set = function(info, key, val) MyAddonDB.modules[key] = val end,
},
```

**Multiselect-specific fields:**
- `values` (table or function) -- Same as `select`.
- `tristate` (boolean) -- Three-state checkboxes (`true`/`nil`/`false`).

**Gotcha:** The get/set signatures differ from other types. `get(info, key)` not `get(info)`. `set(info, key, value)` not `set(info, value)`.

### Option Type: color

A color picker. The get callback must return `r, g, b` or `r, g, b, a`. The set callback receives `(info, r, g, b, a)`.

```lua
barColor = {
    type = "color",
    name = "Health Bar Color",
    desc = "Color of the health bar.",
    hasAlpha = true,
    order = 9,
    get = function(info)
        local c = MyAddonDB.barColor
        return c.r, c.g, c.b, c.a
    end,
    set = function(info, r, g, b, a)
        MyAddonDB.barColor = { r = r, g = g, b = b, a = a }
    end,
},
```

**Color-specific fields:**
- `hasAlpha` (boolean or function) -- Show the alpha slider in the color picker.

### Option Type: execute

A clickable button. Uses `func` instead of get/set.

```lua
resetDefaults = {
    type = "execute",
    name = "Reset to Defaults",
    desc = "Reset all settings to their default values.",
    confirm = true,
    confirmText = "Are you sure you want to reset all settings?",
    order = 10,
    func = function(info)
        wipe(MyAddonDB)
        MyAddon:ApplyDefaults()
        LibStub("AceConfigRegistry-3.0"):NotifyChange("MyAddon")
    end,
},
```

**Execute-specific fields:**
- `image` (string, number, or function) -- Icon displayed on the button.
- `imageCoords` (table or function) -- Texture coords for the image.
- `imageWidth` (number) -- Width of the image.
- `imageHeight` (number) -- Height of the image.

### Option Type: header

A horizontal separator line with optional text.

```lua
spacer = {
    type = "header",
    name = "Advanced Options",
    order = 20,
},
```

Pass `name = ""` for a plain separator line with no text.

### Option Type: description

A block of read-only text. Useful for instructions or notes.

```lua
instructions = {
    type = "description",
    name = "Configure the addon below. Changes take effect immediately.\nUse /myaddon to open this panel.",
    fontSize = "medium",
    order = 0,
},
```

**Description-specific fields:**
- `fontSize` (string or function) -- `"small"`, `"medium"`, or `"large"`.
- `image` (string, number, or function) -- Image displayed beside the text.
- `imageWidth` (number) -- Width of the image.
- `imageHeight` (number) -- Height of the image.
- `imageCoords` (table or function) -- Texture coords for the image.

### Option Type: keybinding

A keybinding capture widget.

```lua
toggleKey = {
    type = "keybinding",
    name = "Toggle Window",
    desc = "Keybind to toggle the main window.",
    order = 11,
    get = function(info) return MyAddonDB.toggleKey end,
    set = function(info, val) MyAddonDB.toggleKey = val end,
},
```

The value stored is a WoW key string like `"CTRL-SHIFT-F"`, or `false`/`nil` for unbound.

### Dynamic Options

Any field that accepts a function is re-evaluated each time the GUI is drawn. This enables dynamic UI.

```lua
targetName = {
    type = "input",
    name = function(info)
        return "Current Target: " .. (UnitName("target") or "None")
    end,
    desc = "Dynamically updated name.",
    hidden = function(info) return not UnitExists("target") end,
    disabled = function(info) return InCombatLockdown() end,
    order = 12,
    get = function(info) return MyAddonDB.targetOverride end,
    set = function(info, val) MyAddonDB.targetOverride = val end,
},
```

**Dynamic values in select:**

```lua
profileSelect = {
    type = "select",
    name = "Active Profile",
    values = function(info)
        local profiles = {}
        for _, name in ipairs(MyAddonDB.profileList) do
            profiles[name] = name
        end
        return profiles
    end,
    order = 13,
    get = function(info) return MyAddonDB.activeProfile end,
    set = function(info, val)
        MyAddonDB.activeProfile = val
        MyAddon:LoadProfile(val)
    end,
},
```

### Forcing a UI Refresh

When your options table changes due to an external event (a game event, timer, data load), call `NotifyChange` to tell all listening config GUIs to redraw.

```lua
local AceConfigRegistry = LibStub("AceConfigRegistry-3.0")
AceConfigRegistry:NotifyChange("MyAddon")
```

This fires a `"ConfigTableChange"` callback internally. AceConfigDialog listens for this and refreshes any open panels.

---

## AceConfigDialog-3.0 -- GUI Generation

AceConfigDialog renders AceConfig options tables as AceGUI-based dialogs. It can embed options in the Blizzard Settings panel or open standalone windows.

### AddToBlizOptions

```lua
AceConfigDialog:AddToBlizOptions(appName [, name] [, parent] [, ...])
```

Registers your options table as a panel in the Blizzard Interface Options (Settings) panel.

**Parameters:**
- `appName` (string) -- The name used in `RegisterOptionsTable`.
- `name` (string, optional) -- Display name in the settings tree. Defaults to `appName`.
- `parent` (string, optional) -- Name of an existing parent category to nest under (one level of nesting supported).
- `...` (strings, optional) -- Path into your options table. If provided, only that sub-group is rendered in this panel.

**Returns:**
- `frame` -- The container frame registered with Interface Options.
- `categoryID` -- The ID to pass to `Settings.OpenToCategory()`.

**Basic example -- single panel:**

```lua
local MyAddon = LibStub("AceAddon-3.0"):NewAddon("MyAddon", "AceConsole-3.0")
local AceConfig = LibStub("AceConfig-3.0")
local AceConfigDialog = LibStub("AceConfigDialog-3.0")

function MyAddon:OnInitialize()
    self.db = LibStub("AceDB-3.0"):New("MyAddonDB", defaults, true)

    AceConfig:RegisterOptionsTable("MyAddon", options)
    self.optionsFrame, self.optionsCategoryID = AceConfigDialog:AddToBlizOptions("MyAddon", "My Addon")

    self:RegisterChatCommand("myaddon", function()
        Settings.OpenToCategory(self.optionsCategoryID)
    end)
end
```

**Multi-panel example with parent/child:**

```lua
function MyAddon:OnInitialize()
    AceConfig:RegisterOptionsTable("MyAddon", mainOptions)
    AceConfig:RegisterOptionsTable("MyAddon_Profiles", profileOptions)

    -- Main category
    local _, mainID = AceConfigDialog:AddToBlizOptions("MyAddon", "My Addon")

    -- Sub-categories under "My Addon"
    AceConfigDialog:AddToBlizOptions("MyAddon", "General", "My Addon", "general")
    AceConfigDialog:AddToBlizOptions("MyAddon", "Display", "My Addon", "display")
    AceConfigDialog:AddToBlizOptions("MyAddon_Profiles", "Profiles", "My Addon")
end
```

**12.0 note:** On 12.0+ clients, the category ID override (`category.ID = categoryName`) is no longer applied. The `categoryID` returned from `AddToBlizOptions` is the correct value to pass to `Settings.OpenToCategory()`. Store it.

### Open (Standalone Window)

```lua
AceConfigDialog:Open(appName [, container] [, ...])
```

Opens a standalone AceGUI `Frame` window for the given options table. If a window for this appName is already open, it refreshes it.

**Parameters:**
- `appName` (string) -- The registered options table name.
- `container` (AceGUI container, optional) -- Feed the options into an existing container instead of creating a new Frame.
- `...` (strings, optional) -- Path to a specific group to open to.

```lua
-- Open a standalone config window
AceConfigDialog:Open("MyAddon")

-- Open directly to a sub-group
AceConfigDialog:Open("MyAddon", "display", "fonts")
```

**Important:** `Open` hooks `CloseSpecialWindows` so that pressing Escape closes the config frame.

### Close

```lua
AceConfigDialog:Close(appName)
```

Closes the standalone window for the given appName. Returns `true` if a window was open.

```lua
AceConfigDialog:Close("MyAddon")
```

### SetDefaultSize

```lua
AceConfigDialog:SetDefaultSize(appName, width, height)
```

Sets the default dimensions for the standalone window. Call this before the first `Open`. The default (if not set) is 700x500.

```lua
AceConfigDialog:SetDefaultSize("MyAddon", 800, 600)
```

### SelectGroup

```lua
AceConfigDialog:SelectGroup(appName, ...)
```

Navigates an already-open options window to a specific group. The path must match the keys in your args tables.

```lua
-- Navigate to the "display" group, then into its "fonts" subgroup
AceConfigDialog:SelectGroup("MyAddon", "display", "fonts")
```

This works with all `childGroups` modes -- it selects the correct tree node, tab, or dropdown entry.

### Pattern: Button in Blizzard Settings That Opens Standalone Dialog

The Blizzard Settings panel has limited width. For complex options with tree navigation, register a minimal panel in Blizzard Settings that contains just a button to open a full standalone dialog.

```lua
local buttonOptions = {
    type = "group",
    name = "My Addon",
    args = {
        openConfig = {
            type = "execute",
            name = "Open My Addon Configuration",
            desc = "Opens the full configuration window.",
            width = "full",
            func = function()
                -- Close the Blizzard settings panel first
                if Settings and Settings.ClosePanel then
                    Settings.ClosePanel()
                elseif SettingsPanel then
                    HideUIPanel(SettingsPanel)
                end
                -- Open our standalone dialog
                AceConfigDialog:SetDefaultSize("MyAddon_Full", 850, 600)
                AceConfigDialog:Open("MyAddon_Full")
            end,
            order = 1,
        },
        info = {
            type = "description",
            name = "\nThe full configuration window provides tree-based navigation for all settings.",
            order = 2,
        },
    },
}

AceConfig:RegisterOptionsTable("MyAddon_Button", buttonOptions)
AceConfig:RegisterOptionsTable("MyAddon_Full", fullOptions)
AceConfigDialog:AddToBlizOptions("MyAddon_Button", "My Addon")
```

---

## AceGUI-3.0 -- Widget Toolkit

AceGUI is the widget framework underlying AceConfigDialog. You can also use it directly to build custom GUIs unrelated to config tables. AceGUI widgets are **not** raw WoW frames -- they are wrapper objects that manage an underlying frame.

### Creating and Releasing Widgets

```lua
local AceGUI = LibStub("AceGUI-3.0")

-- Create a widget
local button = AceGUI:Create("Button")
button:SetText("Click Me")
button:SetWidth(200)
button:SetCallback("OnClick", function(widget)
    -- handle click
end)

-- When done, release it back to the pool
AceGUI:Release(button)
```

**Key concept: widget pooling.** AceGUI maintains an internal pool. `Create` returns a widget from the pool if available, otherwise constructs a new one. `Release` hides the widget, clears its data, and returns it to the pool. Always release widgets you no longer need. Never destroy the underlying frame directly.

### Accessing the Underlying WoW Frame

```lua
local widget = AceGUI:Create("Button")
local frame = widget.frame  -- the actual WoW Frame object

-- Only use this for things AceGUI doesn't expose (e.g., SetFrameStrata)
-- Do NOT resize or reposition via widget.frame -- use widget:SetWidth(), etc.
widget.frame:SetFrameStrata("HIGH")
```

**Warning from the AceGUI source:** "Please do not modify the frames of the widgets directly, as any unknown change to the widgets will cause addons that get your widget out of the widget pool to misbehave."

### Container Widgets

Containers hold child widgets and manage their layout.

| Widget Type | Description |
|-------------|-------------|
| Frame | Standalone movable/resizable window with title bar, status bar, and close button. |
| Window | Simpler movable window without the status bar. |
| InlineGroup | Bordered group with a title, used within other containers (similar to `inline = true` groups). |
| SimpleGroup | Invisible grouping container, no border or title. |
| TabGroup | Container with tab buttons for switching between content pages. |
| TreeGroup | Container with a tree view on the left and content area on the right. |
| DropDownGroup | Container with a dropdown selector for switching content pages. |
| ScrollFrame | Scrollable container for content that exceeds available height. |
| BlizOptionsGroup | Special container for Blizzard Interface Options integration (used internally by AceConfigDialog). |

### Standard Widgets

| Widget Type | Description |
|-------------|-------------|
| Button | Clickable button. Callback: `OnClick`. |
| CheckBox | Toggle checkbox. Callbacks: `OnValueChanged`. |
| ColorPicker | Color selection with optional alpha. Callback: `OnValueConfirmed`, `OnValueChanged`. |
| Dropdown | Dropdown selection list. Callback: `OnValueChanged`. |
| EditBox | Single-line text input. Callbacks: `OnEnterPressed`, `OnTextChanged`. |
| Heading | Horizontal separator line with optional label text. |
| Icon | Clickable icon with optional label. Callbacks: `OnClick`, `OnEnter`, `OnLeave`. |
| InteractiveLabel | Clickable text label. Callbacks: `OnClick`, `OnEnter`, `OnLeave`. |
| Keybinding | Key binding capture widget. Callback: `OnKeyChanged`. |
| Label | Static text display (non-interactive). |
| MultiLineEditBox | Multi-line text editor with accept/cancel buttons. Callback: `OnEnterPressed`. |
| Slider | Numeric slider. Callback: `OnValueChanged`, `OnMouseUp`. |

### Common Widget Methods

These are available on all widgets via the WidgetBase metatable:

```lua
widget:SetCallback(name, func)     -- Register callback: func(widget, event, ...)
widget:SetWidth(width)             -- Set absolute width in pixels
widget:SetHeight(height)           -- Set absolute height in pixels
widget:SetRelativeWidth(fraction)  -- Set width as fraction of container (0.0-1.0)
widget:SetFullWidth(bool)          -- Fill entire container width
widget:SetFullHeight(bool)         -- Fill entire container height
widget:SetPoint(...)               -- Set anchor point (pass-through to frame)
widget:SetUserData(key, value)     -- Store arbitrary data on the widget
widget:GetUserData(key)            -- Retrieve stored data
widget:GetUserDataTable()          -- Get the entire userdata table
widget:IsVisible()                 -- Check visibility
widget:Release()                   -- Shortcut for AceGUI:Release(self)
widget:Fire(name, ...)             -- Manually fire a callback
```

**Container-specific methods:**

```lua
container:AddChild(widget)              -- Add a child widget
container:AddChildren(w1, w2, w3, ...)  -- Add multiple children at once (single layout pass)
container:ReleaseChildren()             -- Release all child widgets
container:SetLayout(layoutName)         -- Set the layout engine: "Flow", "Fill", "List", or "Table"
container:PauseLayout()                 -- Temporarily prevent relayout
container:ResumeLayout()                -- Resume layout (does not trigger relayout)
container:DoLayout()                    -- Force an immediate relayout
container:SetAutoAdjustHeight(bool)     -- Auto-adjust container height to content
```

### Layouts

Layouts control how child widgets are arranged within a container.

**"List"** -- Children are stacked vertically, top-to-bottom, each on its own row. This is the default layout.

```lua
container:SetLayout("List")
```

**"Flow"** -- Children flow left-to-right, wrapping to the next row when the container width is exceeded. Similar to CSS flexbox with `flex-wrap: wrap`.

```lua
container:SetLayout("Flow")
-- Two widgets side by side (each takes ~half the width)
local left = AceGUI:Create("CheckBox")
left:SetRelativeWidth(0.5)
local right = AceGUI:Create("CheckBox")
right:SetRelativeWidth(0.5)
container:AddChild(left)
container:AddChild(right)
```

**"Fill"** -- A single child fills the entire container. Only the first child is used.

```lua
container:SetLayout("Fill")
local editBox = AceGUI:Create("MultiLineEditBox")
container:AddChild(editBox)  -- fills the entire container
```

**"Table"** -- Grid layout. Set column definitions via UserData before adding children.

```lua
container:SetLayout("Table")
container:SetUserData("table", {
    columns = {
        { weight = 1 },   -- column 1: flexible width
        { width = 100 },  -- column 2: fixed 100px
        { width = 0.3 },  -- column 3: 30% of available width
    },
    space = 5,             -- spacing between cells
    -- spaceH, spaceV for separate horizontal/vertical spacing
    -- align = "CENTERLEFT" (default alignment)
})
```

### Building a Custom UI: Complete Example

```lua
local AceGUI = LibStub("AceGUI-3.0")

local function CreateConfigWindow()
    -- Main frame
    local frame = AceGUI:Create("Frame")
    frame:SetTitle("My Addon Configuration")
    frame:SetStatusText("v1.0.0")
    frame:SetLayout("Flow")
    frame:SetWidth(600)
    frame:SetHeight(450)
    frame:SetCallback("OnClose", function(widget)
        AceGUI:Release(widget)
    end)

    -- Heading
    local heading = AceGUI:Create("Heading")
    heading:SetText("General Settings")
    heading:SetFullWidth(true)
    frame:AddChild(heading)

    -- Checkbox
    local enableCheck = AceGUI:Create("CheckBox")
    enableCheck:SetLabel("Enable Addon")
    enableCheck:SetValue(MyAddonDB.enabled)
    enableCheck:SetRelativeWidth(0.5)
    enableCheck:SetCallback("OnValueChanged", function(widget, event, val)
        MyAddonDB.enabled = val
    end)
    frame:AddChild(enableCheck)

    -- Dropdown
    local modeDropdown = AceGUI:Create("Dropdown")
    modeDropdown:SetLabel("Display Mode")
    modeDropdown:SetList({
        simple = "Simple",
        detailed = "Detailed",
        minimal = "Minimal",
    }, {"simple", "detailed", "minimal"})  -- second arg = sort order
    modeDropdown:SetValue(MyAddonDB.mode)
    modeDropdown:SetRelativeWidth(0.5)
    modeDropdown:SetCallback("OnValueChanged", function(widget, event, val)
        MyAddonDB.mode = val
    end)
    frame:AddChild(modeDropdown)

    -- Slider
    local scaleSlider = AceGUI:Create("Slider")
    scaleSlider:SetLabel("UI Scale")
    scaleSlider:SetSliderValues(0.5, 2.0, 0.1)  -- min, max, step
    scaleSlider:SetValue(MyAddonDB.scale)
    scaleSlider:SetFullWidth(true)
    scaleSlider:SetCallback("OnValueChanged", function(widget, event, val)
        MyAddonDB.scale = val
    end)
    frame:AddChild(scaleSlider)

    -- Inline group with a scroll frame
    local scrollContainer = AceGUI:Create("InlineGroup")
    scrollContainer:SetTitle("Log Output")
    scrollContainer:SetFullWidth(true)
    scrollContainer:SetFullHeight(true)  -- fill remaining vertical space
    scrollContainer:SetLayout("Fill")
    frame:AddChild(scrollContainer)

    local scroll = AceGUI:Create("ScrollFrame")
    scroll:SetLayout("List")
    scrollContainer:AddChild(scroll)

    -- Add labels to the scroll frame
    for i, entry in ipairs(MyAddonDB.log or {}) do
        local label = AceGUI:Create("Label")
        label:SetText(entry)
        label:SetFullWidth(true)
        scroll:AddChild(label)
    end

    return frame
end
```

### TabGroup Example

```lua
local function CreateTabbedWindow()
    local frame = AceGUI:Create("Frame")
    frame:SetTitle("Settings")
    frame:SetLayout("Fill")
    frame:SetWidth(500)
    frame:SetHeight(400)
    frame:SetCallback("OnClose", function(w) AceGUI:Release(w) end)

    local tabGroup = AceGUI:Create("TabGroup")
    tabGroup:SetLayout("Flow")
    tabGroup:SetTabs({
        { value = "general", text = "General" },
        { value = "display", text = "Display" },
        { value = "about",   text = "About" },
    })
    tabGroup:SetCallback("OnGroupSelected", function(container, event, group)
        container:ReleaseChildren()
        if group == "general" then
            local cb = AceGUI:Create("CheckBox")
            cb:SetLabel("Enable Feature")
            cb:SetFullWidth(true)
            container:AddChild(cb)
        elseif group == "display" then
            local sl = AceGUI:Create("Slider")
            sl:SetLabel("Scale")
            sl:SetSliderValues(0.5, 2, 0.1)
            sl:SetFullWidth(true)
            container:AddChild(sl)
        elseif group == "about" then
            local lbl = AceGUI:Create("Label")
            lbl:SetText("My Addon v1.0\nAuthor: You")
            lbl:SetFullWidth(true)
            container:AddChild(lbl)
        end
    end)
    tabGroup:SelectTab("general")
    frame:AddChild(tabGroup)

    return frame
end
```

---

## AceComm-3.0 -- Addon Communication

AceComm handles addon-to-addon communication over WoW's addon message channels. It automatically splits messages longer than 255 bytes and reassembles them on the receiving end. It uses ChatThrottleLib to avoid disconnections from send rate limits.

### Embedding

AceComm is designed to be embedded into your addon object:

```lua
local MyAddon = LibStub("AceAddon-3.0"):NewAddon("MyAddon", "AceComm-3.0")
```

### Registering for Messages

```lua
self:RegisterComm(prefix [, method])
```

- `prefix` (string) -- A message classification string, max 16 printable characters. Typically your addon name.
- `method` (string or function, optional) -- Callback method name or function. Defaults to `"OnCommReceived"`.

The callback receives `(prefix, message, distribution, sender)`.

```lua
function MyAddon:OnEnable()
    self:RegisterComm("MyAddonSync")
end

function MyAddon:OnCommReceived(prefix, message, distribution, sender)
    if prefix == "MyAddonSync" then
        -- message is the full reassembled string (even if it was multi-part)
        -- distribution is "WHISPER", "PARTY", "RAID", "GUILD", etc.
        -- sender is the player name (already passed through Ambiguate)
    end
end
```

**Multiple prefixes:**

```lua
function MyAddon:OnEnable()
    self:RegisterComm("MyAddonSync", "HandleSync")
    self:RegisterComm("MyAddonVer", "HandleVersionCheck")
end

function MyAddon:HandleSync(prefix, message, distribution, sender)
    -- handle sync data
end

function MyAddon:HandleVersionCheck(prefix, message, distribution, sender)
    -- handle version check
end
```

### Sending Messages

```lua
self:SendCommMessage(prefix, text, distribution [, target] [, prio] [, callbackFn] [, callbackArg])
```

- `prefix` (string) -- Must match what recipients registered with `RegisterComm`.
- `text` (string) -- The message data. Nils (`\000`) not allowed. No length limit (auto-split at 255 bytes).
- `distribution` (string) -- Channel: `"WHISPER"`, `"PARTY"`, `"RAID"`, `"GUILD"`, `"INSTANCE_CHAT"`.
- `target` (string or nil) -- Required for `"WHISPER"`, ignored for others.
- `prio` (string) -- `"ALERT"`, `"NORMAL"` (default), or `"BULK"`.
- `callbackFn` (function) -- Progress callback: `function(callbackArg, bytesSent, bytesTotal, sendResult)`.
- `callbackArg` (any) -- First argument passed to callbackFn.

```lua
-- Simple message to guild
self:SendCommMessage("MyAddonSync", "HELLO", "GUILD")

-- Whisper to a specific player
self:SendCommMessage("MyAddonSync", serializedData, "WHISPER", "Targetplayer", "BULK")

-- With progress callback for large messages
self:SendCommMessage("MyAddonSync", largePayload, "RAID", nil, "BULK",
    function(arg, sent, total)
        local pct = math.floor((sent / total) * 100)
        -- update a progress bar, etc.
    end
)
```

### Priority Levels

- `"ALERT"` -- Highest priority. Use for time-sensitive data (ready checks, pull timers). Sends immediately.
- `"NORMAL"` -- Default. Standard priority for regular addon data.
- `"BULK"` -- Lowest priority. Use for large data transfers (profile sync, export). Yields to higher priorities.

**Important:** A single multi-part message always uses the same priority for all chunks. AceComm enforces this to prevent out-of-sequence delivery.

### Unregistering

```lua
self:UnregisterComm("MyAddonSync")   -- Unregister a specific prefix
self:UnregisterAllComm()             -- Unregister all prefixes
```

### Auto-Splitting and Reassembly

AceComm transparently handles messages exceeding 255 bytes:

1. The sender splits the message into 254-byte chunks (1 byte reserved for the multi-part control character).
2. Each chunk is prefixed with `\001` (first), `\002` (continuation), or `\003` (last).
3. The receiver buffers chunks keyed by `prefix + distribution + sender` and concatenates them when the last chunk arrives.
4. Your `OnCommReceived` callback only fires once with the fully reassembled message.

You never need to handle splitting yourself.

### ChatThrottleLib Integration

AceComm requires ChatThrottleLib (bundled with Ace3). CTL manages send rate to avoid disconnection from Blizzard's throttling. It queues messages and drains them at a safe rate. The `prio` parameter controls queue ordering within CTL.

### Comparison with Base WoW API

| Feature | Base API | AceComm |
|---------|----------|---------|
| Register prefix | `C_ChatInfo.RegisterAddonMessagePrefix(prefix)` | `self:RegisterComm(prefix)` |
| Send message | `C_ChatInfo.SendAddonMessage(prefix, msg, type, target)` | `self:SendCommMessage(prefix, msg, dist, target)` |
| Max message size | 255 bytes (silently truncated) | Unlimited (auto-split) |
| Receive event | `CHAT_MSG_ADDON` | Callback method |
| Throttle protection | None (manual) | ChatThrottleLib built-in |
| Multi-part reassembly | Manual | Automatic |

---

## AceSerializer-3.0 -- Data Serialization

AceSerializer converts Lua values into a string representation suitable for transmission via AceComm or storage.

### Embedding

```lua
local MyAddon = LibStub("AceAddon-3.0"):NewAddon("MyAddon", "AceSerializer-3.0", "AceComm-3.0")
```

### Serialize

```lua
local str = self:Serialize(...)
```

Accepts any number of arguments. Returns a single string containing all serialized values.

```lua
local data = {
    name = "TestProfile",
    settings = { scale = 1.5, enabled = true },
    tags = { "pvp", "raid" },
}
local serialized = self:Serialize(data)
-- serialized is now a string like "^1^T^Sname^STestProfile^Ssettings^T..."
```

**Multiple values:**

```lua
local serialized = self:Serialize("header", 42, true, someTable)
-- All four values are packed into one string
```

### Deserialize

```lua
local success, ... = self:Deserialize(str)
```

Returns `true` followed by the deserialized values, or `false` followed by an error message.

```lua
local success, data = self:Deserialize(serialized)
if success then
    -- data is the original table — e.g. data.name == "TestProfile"
    local name = data.name
else
    -- data is the error message string
    geterrorhandler()(data)
end
```

**Multiple values:**

```lua
local success, header, num, flag, tbl = self:Deserialize(serialized)
if success then
    -- header = "header", num = 42, flag = true, tbl = someTable
end
```

### Supported Types

| Type | Supported | Notes |
|------|-----------|-------|
| string | Yes | All characters preserved, including control chars. |
| number | Yes | Full precision for floats via mantissa/exponent encoding. Handles inf, -inf. |
| boolean | Yes | Both true and false. |
| nil | Yes | Explicit nils are preserved. |
| table | Yes | Both hash and array parts. Nested tables supported. |
| function | **No** | Throws an error. |
| userdata | **No** | Throws an error. |

**Gotcha:** Multiple references to the same table are serialized as independent copies. After deserialization, they will be separate table instances.

### Common Pattern: Serialize + AceComm

The most common use is serializing data for transmission to other players.

```lua
local MyAddon = LibStub("AceAddon-3.0"):NewAddon("MyAddon", "AceSerializer-3.0", "AceComm-3.0")

function MyAddon:SendProfile(target)
    local data = {
        version = 3,
        profile = self.db.profile,
    }
    local serialized = self:Serialize(data)
    self:SendCommMessage("MyAddonProf", serialized, "WHISPER", target, "BULK")
end

function MyAddon:OnEnable()
    self:RegisterComm("MyAddonProf")
end

function MyAddon:OnCommReceived(prefix, message, distribution, sender)
    if prefix == "MyAddonProf" then
        local success, data = self:Deserialize(message)
        if not success then return end

        if data.version ~= 3 then
            -- version mismatch, ignore or migrate
            return
        end
        -- data.profile contains the sender's profile settings
        self:ImportProfile(data.profile, sender)
    end
end
```

### Common Pattern: Serialize + LibDeflate for Compressed Storage

For SavedVariables compression or compact export strings:

```lua
local LibDeflate = LibStub("LibDeflate")

-- Compress for storage
function MyAddon:ExportData(data)
    local serialized = self:Serialize(data)
    local compressed = LibDeflate:CompressDeflate(serialized)
    local encoded = LibDeflate:EncodeForPrint(compressed)
    return encoded  -- safe for copy/paste in edit boxes
end

-- Decompress from storage
function MyAddon:ImportData(encoded)
    local compressed = LibDeflate:DecodeForPrint(encoded)
    if not compressed then return nil, "Invalid encoded string" end

    local serialized = LibDeflate:DecompressDeflate(compressed)
    if not serialized then return nil, "Decompression failed" end

    local success, data = self:Deserialize(serialized)
    if not success then return nil, "Deserialization failed: " .. data end

    return data
end
```

---

## AceBucket-3.0 -- Event Throttling

AceBucket collects events that fire in rapid bursts and delivers them as a single batch callback at a configurable interval. This is essential for events like `BAG_UPDATE`, `UNIT_AURA`, or `UNIT_HEALTH` that can fire dozens of times per frame.

### Embedding

```lua
local MyAddon = LibStub("AceAddon-3.0"):NewAddon("MyAddon", "AceBucket-3.0")
```

AceBucket depends on AceEvent-3.0 and AceTimer-3.0 internally (loaded lazily on first use).

### RegisterBucketEvent

```lua
local handle = self:RegisterBucketEvent(event, interval, callback)
```

- `event` (string or table) -- A single event name, or a table of event names.
- `interval` (number) -- Seconds between bucket fires. The bucket waits this long after the first event before calling back.
- `callback` (string or function) -- Method name on self, or a function reference. If omitted and `event` is a string, defaults to the event name as the method name.

**Returns:** A handle (string) for later unregistration.

**The callback receives a table** mapping each event's first argument (arg1) to the number of times it appeared:

```lua
function MyAddon:OnEnable()
    self:RegisterBucketEvent("UNIT_AURA", 0.3, "HandleAuraChanges")
end

function MyAddon:HandleAuraChanges(units)
    -- units = { ["player"] = 5, ["target"] = 2, ["party1"] = 1 }
    -- The keys are the first argument of each UNIT_AURA event that fired
    -- The values are how many times that arg1 appeared

    if units.player then
        self:UpdatePlayerBuffs()
    end
    if units.target then
        self:UpdateTargetDebuffs()
    end
end
```

**Multiple events in one bucket:**

```lua
function MyAddon:OnEnable()
    self:RegisterBucketEvent(
        {"UNIT_HEALTH", "UNIT_MAXHEALTH", "UNIT_ABSORB_AMOUNT_CHANGED"},
        0.2,
        "UpdateHealthDisplay"
    )
end

function MyAddon:UpdateHealthDisplay(units)
    -- units contains the unitIDs from ALL three events combined
    for unitID, count in pairs(units) do
        self:RefreshUnitFrame(unitID)
    end
end
```

> **12.0.0+ Secret Values caveat.** `UnitHealth(unit)`, `UnitHealthMax(unit)`, and `UnitGetTotalAbsorbs(unit)` return SECRET values during combat (and in any tainted execution context). If `RefreshUnitFrame` uses those results for arithmetic, comparisons, or `string.format("%d", ...)`, it will error. For a plain health bar, use `UnitHealthPercent(unit, false, CurveConstants.ScaleTo100)` which returns a non-secret 0–100 percentage. For raw-value display paths, guard with `issecretvalue()` first. See [Secret Safe APIs](12a_Secret_Safe_APIs.md) for the full pattern.

### RegisterBucketMessage

```lua
local handle = self:RegisterBucketMessage(message, interval, callback)
```

Same as `RegisterBucketEvent` but listens for AceEvent messages (fired via `self:SendMessage()`) instead of WoW game events.

```lua
function MyAddon:OnEnable()
    self:RegisterBucketMessage("MyAddon_DataUpdated", 0.5, "ProcessBatchUpdate")
end

function MyAddon:ProcessBatchUpdate(sources)
    -- sources contains the arg1 values from all MyAddon_DataUpdated messages
end
```

### Unregistering

```lua
self:UnregisterBucket(handle)  -- Unregister a specific bucket by handle
self:UnregisterAllBuckets()    -- Unregister all buckets for this addon
```

### How the Bucket Works Internally

1. **Idle state:** The bucket is listening but has no active timer.
2. **First event arrives:** The arg1 is recorded in the `received` table. A timer is started for `interval` seconds.
3. **More events arrive during the interval:** Each arg1 is counted in the `received` table. No new timer is started.
4. **Timer fires:** If the `received` table has any entries, the callback fires with the table. The table is then wiped, and a new timer starts.
5. **Timer fires with empty table:** The timer stops. The bucket returns to idle, waiting for the next event.

**Key behavior:** The bucket only fires if at least one event was received. Empty buckets do not produce callbacks.

### When to Use AceBucket

**Good candidates for bucketing:**
- `BAG_UPDATE` -- Fires per-slot during mass loot, vendor purchases, mail.
- `UNIT_AURA` -- Fires per-aura change; opening a raid frame triggers many at once.
- `UNIT_HEALTH` / `UNIT_POWER_UPDATE` -- Frequent during combat for multiple units.
- `CHAT_MSG_*` events -- Chat spam scenarios.
- `SPELL_UPDATE_COOLDOWN` -- Fires for every cooldown change, often multiple per GCD.
- Custom AceEvent messages from high-frequency data sources.

**Not appropriate for bucketing:**
- Events where you need immediate response (combat state changes, targeting).
- Events with unique payloads beyond arg1 that you need individually (the bucket only tracks arg1 counts).
- Events that fire rarely (no benefit from bucketing).

### Comparison with Dirty-Flag + C_Timer Pattern

The manual alternative to AceBucket:

```lua
-- Manual approach (without AceBucket)
local dirty = false
function MyAddon:UNIT_AURA(event, unit)
    if unit == "player" then
        dirty = true
        if not self.updatePending then
            self.updatePending = true
            C_Timer.After(0.3, function()
                self.updatePending = false
                if dirty then
                    dirty = false
                    self:UpdateBuffDisplay()
                end
            end)
        end
    end
end
```

AceBucket encapsulates this pattern with less boilerplate, tracks which arg1 values fired, and handles multiple events in a single bucket. The manual approach is lighter (no library dependency) and allows tracking more than just arg1, but requires more code per event.

---

## AceLocale-3.0 -- Localization

AceLocale manages translation strings for addons. It supports a default locale (typically enUS) with optional overrides for other languages, and provides fallback behavior when translations are missing.

### Concepts

- **Default locale:** The language your addon is written in. This is the fallback for any missing translations.
- **Active locale:** The user's game client language (`GetLocale()`).
- **Non-matching locales return nil from NewLocale**, so translation files for other languages safely skip all their assignments.
- **The `L["key"] = true` pattern:** In the default locale, assigning `true` sets the value to the key string itself, avoiding redundant `L["Enable"] = "Enable"`.

### NewLocale

```lua
local L = LibStub("AceLocale-3.0"):NewLocale(appName, locale [, isDefault] [, silent])
```

- `appName` (string) -- Unique name for your addon (typically the addon folder name).
- `locale` (string) -- The locale code: `"enUS"`, `"deDE"`, `"frFR"`, `"esES"`, `"esMX"`, `"ptBR"`, `"ruRU"`, `"koKR"`, `"zhCN"`, `"zhTW"`, `"itIT"`.
- `isDefault` (boolean) -- `true` if this is the default/fallback locale.
- `silent` (boolean or string) -- Controls behavior for missing keys:
  - `nil`/`false` (default) -- Logs an error on first access to a missing key, then caches the key string as the value.
  - `true` -- Silently returns the key string for missing keys (no error).
  - `"raw"` -- Returns `nil` for missing keys (no metatable fallback at all).

**Returns:** A table to fill with translations, or `nil` if this locale is not needed (not default and not the active game locale).

**Important:** `silent` must be set on the **first** locale registered (the default). Setting it later throws an error.

### GetLocale

```lua
local L = LibStub("AceLocale-3.0"):GetLocale(appName [, silent])
```

- `appName` (string) -- The same name used in `NewLocale`.
- `silent` (boolean, optional) -- If `true`, returns `nil` instead of erroring when no locale exists for this appName.

**Returns:** The locale table with all translations merged (default + current language overrides).

### Complete Localization Workflow

**File structure in your addon:**

```
MyAddon/
  MyAddon.toc
  Locales/
    enUS.lua    -- default locale (loaded first)
    deDE.lua
    frFR.lua
    zhCN.lua
  Core.lua
```

**TOC file (load order matters):**

```
## Interface: 120000
## Title: My Addon
## Notes: Example addon with localization

Locales\enUS.lua
Locales\deDE.lua
Locales\frFR.lua
Locales\zhCN.lua
Core.lua
```

**Locales/enUS.lua (default locale):**

```lua
local L = LibStub("AceLocale-3.0"):NewLocale("MyAddon", "enUS", true)

-- Using true sets the value to the key string itself
L["Enable"] = true                              -- value becomes "Enable"
L["Show Minimap Icon"] = true                   -- value becomes "Show Minimap Icon"
L["Health Bar Color"] = true                    -- value becomes "Health Bar Color"

-- You can also set explicit values (useful when the key is a code-style identifier)
L["TOOLTIP_LINE1"] = "Left-click to open settings"
L["TOOLTIP_LINE2"] = "Right-click to toggle display"
L["WELCOME_MSG"] = "My Addon loaded. Type /myaddon for options."
```

**Locales/deDE.lua (German):**

```lua
local L = LibStub("AceLocale-3.0"):NewLocale("MyAddon", "deDE")
if not L then return end  -- not German client, skip entirely

L["Enable"] = "Aktivieren"
L["Show Minimap Icon"] = "Minimap-Symbol anzeigen"
L["Health Bar Color"] = "Farbe der Lebensleiste"
L["TOOLTIP_LINE1"] = "Linksklick zum Oeffnen der Einstellungen"
L["TOOLTIP_LINE2"] = "Rechtsklick zum Umschalten der Anzeige"
L["WELCOME_MSG"] = "My Addon geladen. /myaddon fuer Optionen."
```

**Locales/frFR.lua (French):**

```lua
local L = LibStub("AceLocale-3.0"):NewLocale("MyAddon", "frFR")
if not L then return end

L["Enable"] = "Activer"
L["Show Minimap Icon"] = "Afficher l'icone de la minicarte"
-- Missing keys (Health Bar Color, TOOLTIP_*, WELCOME_MSG) will fall back to enUS values
```

**Core.lua (using the locale):**

```lua
local MyAddon = LibStub("AceAddon-3.0"):NewAddon("MyAddon", "AceConsole-3.0")
local L = LibStub("AceLocale-3.0"):GetLocale("MyAddon")

local options = {
    type = "group",
    name = "My Addon",
    args = {
        enable = {
            type = "toggle",
            name = L["Enable"],
            order = 1,
            get = function(info) return MyAddonDB.enabled end,
            set = function(info, val) MyAddonDB.enabled = val end,
        },
        minimap = {
            type = "toggle",
            name = L["Show Minimap Icon"],
            order = 2,
            get = function(info) return MyAddonDB.minimap end,
            set = function(info, val) MyAddonDB.minimap = val end,
        },
        barColor = {
            type = "color",
            name = L["Health Bar Color"],
            order = 3,
            get = function(info)
                local c = MyAddonDB.barColor
                return c.r, c.g, c.b
            end,
            set = function(info, r, g, b)
                MyAddonDB.barColor = { r = r, g = g, b = b }
            end,
        },
    },
}

function MyAddon:OnInitialize()
    self:Print(L["WELCOME_MSG"])
end
```

### Silent Mode vs Raw Mode

**Default (no silent flag):** On a French client, accessing `L["TOOLTIP_LINE1"]` (which was not translated in frFR.lua) triggers a one-time error message in the error handler: `"AceLocale-3.0: MyAddon: Missing entry for 'TOOLTIP_LINE1'"`. Then it caches and returns `"TOOLTIP_LINE1"` (the key string). This is useful during development to find missing translations.

**silent = true:** Same fallback behavior (returns the key string), but no error is logged. Use this for production addons where missing translations are acceptable.

**silent = "raw":** Returns `nil` for unknown keys. No metatable is set. Use this when you need to test whether a key exists:

```lua
local L = LibStub("AceLocale-3.0"):NewLocale("MyAddon", "enUS", true, "raw")
-- ...
local L = LibStub("AceLocale-3.0"):GetLocale("MyAddon")
if L["OPTIONAL_FEATURE"] then
    -- key exists, use it
end
-- With "raw", L["NONEXISTENT"] returns nil instead of "NONEXISTENT"
```

### Gotchas

- **Load order matters.** The default locale must be loaded first (listed first in the TOC). Non-default locales can be in any order after that.
- **`enGB` is treated as `enUS`.** AceLocale maps `"enGB"` to `"enUS"` internally.
- **NewLocale returns a proxy, not the real table.** The proxy handles the `true`-to-key-string conversion. You cannot iterate the proxy with `pairs()`. Use `GetLocale()` to get the real table.
- **The `GAME_LOCALE` global** (if set) overrides `GetLocale()`. Translators use this to test translations without changing their WoW client language.
- **Default locale uses `writedefaultproxy`** which refuses to overwrite existing values. If the German locale loaded first and set `L["Enable"] = "Aktivieren"`, then the enUS default `L["Enable"] = true` will NOT overwrite it. This allows locales to load in any order safely.

---

## Complete Ace3 Addon Example

This section builds a complete, working addon called **GoldTracker** -- a simple gold-per-session tracker that records how much gold you've earned or spent since login. It demonstrates all the major Ace3 libraries working together.

### TOC File

```toc
## Interface: 120001
## Title: GoldTracker
## Notes: Tracks gold earned and spent per session
## Author: YourName
## Version: 1.0.0
## SavedVariables: GoldTrackerDB
## OptionalDeps: Ace3
## Category: Economy

# Libraries -- order matters!
# LibStub must load first, then CallbackHandler, then everything else
Libs\LibStub\LibStub.lua
Libs\CallbackHandler-1.0\CallbackHandler-1.0.lua
Libs\AceAddon-3.0\AceAddon-3.0.lua
Libs\AceEvent-3.0\AceEvent-3.0.lua
Libs\AceTimer-3.0\AceTimer-3.0.lua
Libs\AceBucket-3.0\AceBucket-3.0.lua
Libs\AceDB-3.0\AceDB-3.0.lua
Libs\AceDBOptions-3.0\AceDBOptions-3.0.lua
Libs\AceConsole-3.0\AceConsole-3.0.lua
Libs\AceConfig-3.0\AceConfig-3.0.lua
Libs\AceLocale-3.0\AceLocale-3.0.lua

# Localization files -- default locale MUST load first
Locales\enUS.lua
Locales\deDE.lua

# Addon files
Core.lua
Options.lua
Modules\SessionLog.lua
```

### Localization: Locales/enUS.lua

```lua
-- Default locale (enUS). This MUST load before any other locale file.
-- The third argument 'true' marks this as the default/fallback locale.
local L = LibStub("AceLocale-3.0"):NewLocale("GoldTracker", "enUS", true)

-- When a key is assigned 'true', the key string itself becomes the value.
-- This is a convenience so you don't have to write L["Gold Tracker"] = "Gold Tracker"
L["Gold Tracker"] = true
L["Session Summary"] = true
L["gold_earned"] = "Gold earned this session: %s"
L["gold_spent"] = "Gold spent this session: %s"
L["gold_net"] = "Net change: %s"
L["session_start"] = "Session started. Current gold: %s"
L["opt_general"] = "General"
L["opt_show_login"] = "Show on Login"
L["opt_show_login_desc"] = "Display your current gold when you log in."
L["opt_show_changes"] = "Show Gold Changes"
L["opt_show_changes_desc"] = "Print a message when your gold changes."
L["opt_minimap"] = "Minimap"
L["opt_threshold"] = "Change Threshold"
L["opt_threshold_desc"] = "Minimum gold change (in copper) to trigger a notification."
```

### Localization: Locales/deDE.lua

```lua
-- German locale. NewLocale returns nil if the player's client isn't deDE,
-- so we bail out early to avoid loading unnecessary strings.
local L = LibStub("AceLocale-3.0"):NewLocale("GoldTracker", "deDE")
if not L then return end

L["Gold Tracker"] = "Gold Tracker"
L["Session Summary"] = "Sitzungsübersicht"
L["gold_earned"] = "Gold verdient in dieser Sitzung: %s"
L["gold_spent"] = "Gold ausgegeben in dieser Sitzung: %s"
L["gold_net"] = "Nettoveränderung: %s"
L["session_start"] = "Sitzung gestartet. Aktuelles Gold: %s"
L["opt_general"] = "Allgemein"
L["opt_show_login"] = "Beim Login anzeigen"
L["opt_show_login_desc"] = "Zeigt dein aktuelles Gold beim Einloggen an."
L["opt_show_changes"] = "Goldänderungen anzeigen"
L["opt_show_changes_desc"] = "Zeigt eine Nachricht, wenn sich dein Gold ändert."
L["opt_minimap"] = "Minikarte"
L["opt_threshold"] = "Änderungsschwelle"
L["opt_threshold_desc"] = "Mindestgoldänderung (in Kupfer), um eine Benachrichtigung auszulösen."
```

### Core.lua

```lua
-- Grab the locale table (returns the correct locale for the player's client)
local L = LibStub("AceLocale-3.0"):GetLocale("GoldTracker")

-- Create the addon object, embedding the libraries we need.
-- The first argument is the addon name (must match the TOC filename).
-- Subsequent arguments are libraries to embed as mixins.
local GoldTracker = LibStub("AceAddon-3.0"):NewAddon(
	"GoldTracker",
	"AceEvent-3.0",    -- RegisterEvent, UnregisterEvent, SendMessage
	"AceConsole-3.0",  -- RegisterChatCommand, Print, GetArgs
	"AceBucket-3.0",   -- RegisterBucketEvent (throttled events)
	"AceTimer-3.0"     -- ScheduleTimer, ScheduleRepeatingTimer
)

-- Store a reference to the locale on the addon for modules to access
GoldTracker.L = L

-- Localise frequently used globals for performance
local GetMoney = GetMoney
local GetCoinTextureString = GetCoinTextureString

----------------------------------------------------------------------
-- OnInitialize: Called once, immediately after ADDON_LOADED.
-- Use this for one-time setup: database, options, slash commands.
-- Do NOT register events here -- the game world may not be ready yet.
----------------------------------------------------------------------
function GoldTracker:OnInitialize()
	-- Set up saved variables with AceDB.
	-- "GoldTrackerDB" must match the ## SavedVariables line in the TOC.
	-- The second argument is the defaults table.
	-- The third argument 'true' means use a shared "Default" profile.
	local defaults = {
		profile = {
			showOnLogin = true,
			showChanges = true,
			changeThreshold = 0, -- in copper; 0 means show all changes
		},
		char = {
			-- Per-character data: track the last known gold value
			lastKnownGold = 0,
		},
		global = {
			-- Account-wide lifetime stats
			totalEarned = 0,
			totalSpent = 0,
		},
	}
	self.db = LibStub("AceDB-3.0"):New("GoldTrackerDB", defaults, true)

	-- Register a callback for when the user switches profiles.
	-- The DB fires "OnProfileChanged", "OnProfileCopied", "OnProfileReset"
	-- when profiles change. We handle all three the same way.
	self.db.RegisterCallback(self, "OnProfileChanged", "RefreshConfig")
	self.db.RegisterCallback(self, "OnProfileCopied", "RefreshConfig")
	self.db.RegisterCallback(self, "OnProfileReset", "RefreshConfig")

	-- Register the options table (defined in Options.lua)
	-- This makes it available to AceConfigDialog and AceConfigCmd
	local AceConfig = LibStub("AceConfig-3.0")
	AceConfig:RegisterOptionsTable("GoldTracker", self:GetOptionsTable())

	-- Add to Blizzard's Settings panel (ESC -> Options -> AddOns)
	local AceConfigDialog = LibStub("AceConfigDialog-3.0")
	AceConfigDialog:AddToBlizOptions("GoldTracker", L["Gold Tracker"])

	-- Add the AceDBOptions profile management panel as a sub-category
	local profileOptions = LibStub("AceDBOptions-3.0"):GetOptionsTable(self.db)
	AceConfig:RegisterOptionsTable("GoldTracker_Profiles", profileOptions)
	AceConfigDialog:AddToBlizOptions("GoldTracker_Profiles", "Profiles", L["Gold Tracker"])

	-- Set the default size for the standalone options window
	AceConfigDialog:SetDefaultSize("GoldTracker", 600, 400)

	-- Register slash commands. The second argument is a method name on self.
	self:RegisterChatCommand("goldtracker", "SlashCommand")
	self:RegisterChatCommand("gt", "SlashCommand")

	-- Session tracking variables (not saved -- reset every login)
	self.sessionStart = 0
	self.sessionEarned = 0
	self.sessionSpent = 0
end

----------------------------------------------------------------------
-- OnEnable: Called at PLAYER_LOGIN (or immediately if already logged in).
-- The game world is ready. Register events and start doing things.
----------------------------------------------------------------------
function GoldTracker:OnEnable()
	-- Record starting gold for the session
	self.sessionStart = GetMoney()
	self.db.char.lastKnownGold = self.sessionStart

	-- Register for the PLAYER_MONEY event using the method-name pattern.
	-- AceEvent will call self:PLAYER_MONEY(event) when money changes.
	self:RegisterEvent("PLAYER_MONEY")

	-- Use a bucket event for BAG_UPDATE. Bag events fire in rapid bursts
	-- (e.g., when looting multiple items). The bucket collects all events
	-- in a 0.3-second window, then fires once with a table of arg1 values.
	self:RegisterBucketEvent("BAG_UPDATE", 0.3, "OnBagUpdate")

	-- Show login message if the option is enabled
	if self.db.profile.showOnLogin then
		self:Print(L["session_start"]:format(GetCoinTextureString(self.sessionStart)))
	end
end

----------------------------------------------------------------------
-- OnDisable: Called when the addon is manually disabled.
-- Clean up events, timers, hooks, etc.
----------------------------------------------------------------------
function GoldTracker:OnDisable()
	-- AceEvent, AceTimer, AceBucket, and AceHook all automatically
	-- unregister everything in OnDisable via their OnEmbedDisable callbacks.
	-- You only need explicit cleanup for things NOT managed by Ace3.
end

----------------------------------------------------------------------
-- Event Handlers
----------------------------------------------------------------------

-- Method-name event handler: AceEvent calls self:PLAYER_MONEY("PLAYER_MONEY")
function GoldTracker:PLAYER_MONEY(event)
	local currentGold = GetMoney()
	local previousGold = self.db.char.lastKnownGold
	local diff = currentGold - previousGold

	-- Check threshold before reporting
	local threshold = self.db.profile.changeThreshold
	if math.abs(diff) < threshold then
		self.db.char.lastKnownGold = currentGold
		return
	end

	-- Track earned vs spent
	if diff > 0 then
		self.sessionEarned = self.sessionEarned + diff
		self.db.global.totalEarned = self.db.global.totalEarned + diff
	elseif diff < 0 then
		self.sessionSpent = self.sessionSpent + math.abs(diff)
		self.db.global.totalSpent = self.db.global.totalSpent + math.abs(diff)
	end

	self.db.char.lastKnownGold = currentGold

	-- Notify user if the option is enabled
	if self.db.profile.showChanges and diff ~= 0 then
		local sign = diff > 0 and "|cFF00FF00+" or "|cFFFF0000"
		self:Print(sign .. GetCoinTextureString(math.abs(diff)) .. "|r")
	end

	-- Fire an AceEvent message so other addons/modules can react
	self:SendMessage("GoldTracker_GoldChanged", currentGold, diff)
end

-- Bucket event callback for BAG_UPDATE.
-- 'bagIDs' is a table: keys are the bag slot IDs that changed,
-- values are the number of times that bag fired the event.
function GoldTracker:OnBagUpdate(bagIDs)
	-- Example: you could scan bags here for vendor trash totals, etc.
	-- For this addon, we just use it as a demonstration of bucket events.
	-- if bagIDs[0] then ... end  -- backpack changed
end

----------------------------------------------------------------------
-- Slash Command Handler
----------------------------------------------------------------------

function GoldTracker:SlashCommand(input)
	-- GetArgs parses space-separated arguments, respecting quotes and item links.
	-- Returns arg1, arg2, ..., nextPosition. Returns 1e9 for nextPosition at end.
	local cmd, nextPos = self:GetArgs(input, 1)

	if not cmd or cmd == "" or cmd == "summary" then
		-- Show session summary
		self:Print(L["gold_earned"]:format(GetCoinTextureString(self.sessionEarned)))
		self:Print(L["gold_spent"]:format(GetCoinTextureString(self.sessionSpent)))
		local net = self.sessionEarned - self.sessionSpent
		local color = net >= 0 and "|cFF00FF00" or "|cFFFF0000"
		self:Print(L["gold_net"]:format(color .. GetCoinTextureString(math.abs(net)) .. "|r"))
	elseif cmd == "config" or cmd == "options" then
		-- Open the standalone options window (not the Blizzard Settings panel)
		LibStub("AceConfigDialog-3.0"):Open("GoldTracker")
	elseif cmd == "reset" then
		-- Reset session counters
		self.sessionStart = GetMoney()
		self.sessionEarned = 0
		self.sessionSpent = 0
		self:Print("Session counters reset.")
	else
		-- Unknown command: show usage
		self:Print("Usage: /gt [summary|config|reset]")
	end
end

----------------------------------------------------------------------
-- Profile Change Handler
----------------------------------------------------------------------

function GoldTracker:RefreshConfig()
	-- Called when the user switches, copies, or resets a profile.
	-- Re-apply any settings that need immediate effect.
	-- For this addon, most settings are checked on-the-fly,
	-- so there's nothing to refresh -- but this is the pattern.
end
```

### Options.lua

```lua
local GoldTracker = LibStub("AceAddon-3.0"):GetAddon("GoldTracker")
local L = LibStub("AceLocale-3.0"):GetLocale("GoldTracker")

function GoldTracker:GetOptionsTable()
	return {
		type = "group",          -- Root of every AceConfig table must be type="group"
		name = L["Gold Tracker"],

		-- childGroups controls how direct child groups are displayed:
		-- "tree" = tree view on the left (default)
		-- "tab"  = tabs across the top
		-- "select" = dropdown to pick a group
		childGroups = "tab",

		args = {
			general = {
				type = "group",
				name = L["opt_general"],
				order = 1,        -- Without explicit order, groups appear in random order!

				args = {
					showOnLogin = {
						type = "toggle",
						name = L["opt_show_login"],
						desc = L["opt_show_login_desc"],
						order = 1,
						width = "full",   -- widget takes full row width
						get = function(info)
							return self.db.profile.showOnLogin
						end,
						set = function(info, value)
							self.db.profile.showOnLogin = value
						end,
					},

					showChanges = {
						type = "toggle",
						name = L["opt_show_changes"],
						desc = L["opt_show_changes_desc"],
						order = 2,
						width = "full",
						get = function(info)
							return self.db.profile.showChanges
						end,
						set = function(info, value)
							self.db.profile.showChanges = value
						end,
					},

					changeThreshold = {
						type = "range",
						name = L["opt_threshold"],
						desc = L["opt_threshold_desc"],
						order = 3,
						min = 0,
						max = 10000,      -- 1 gold = 10000 copper
						softMax = 10000,  -- slider max (user can type higher)
						step = 100,       -- snap to silver increments
						bigStep = 1000,   -- shift-click step
						get = function(info)
							return self.db.profile.changeThreshold
						end,
						set = function(info, value)
							self.db.profile.changeThreshold = value
						end,
					},

					spacer1 = {
						type = "description",
						name = "\n",
						order = 4,
						-- 'description' type renders text. Use it for spacing or instructions.
					},

					resetSession = {
						type = "execute",
						name = "Reset Session",
						desc = "Reset the session gold counters to zero.",
						order = 5,
						func = function()
							self.sessionStart = GetMoney()
							self.sessionEarned = 0
							self.sessionSpent = 0
							self:Print("Session counters reset.")
						end,
						-- 'confirm' can be a boolean or string. If true, shows a generic
						-- confirmation dialog. If a string, uses that as the prompt.
						confirm = true,
					},
				},
			},

			stats = {
				type = "group",
				name = "Statistics",
				order = 2,

				args = {
					sessionHeader = {
						type = "header",
						name = "Session Statistics",
						order = 1,
					},

					sessionEarned = {
						type = "description",
						name = function()
							return "Earned: " .. GetCoinTextureString(self.sessionEarned)
						end,
						order = 2,
						fontSize = "medium",
					},

					sessionSpent = {
						type = "description",
						name = function()
							return "Spent: " .. GetCoinTextureString(self.sessionSpent)
						end,
						order = 3,
						fontSize = "medium",
					},

					lifetimeHeader = {
						type = "header",
						name = "Lifetime Statistics (Account-Wide)",
						order = 10,
					},

					lifetimeEarned = {
						type = "description",
						name = function()
							return "Total Earned: " .. GetCoinTextureString(self.db.global.totalEarned)
						end,
						order = 11,
						fontSize = "medium",
					},

					lifetimeSpent = {
						type = "description",
						name = function()
							return "Total Spent: " .. GetCoinTextureString(self.db.global.totalSpent)
						end,
						order = 12,
						fontSize = "medium",
					},
				},
			},
		},
	}
end
```

### Module: Modules/SessionLog.lua

```lua
-- Modules are mini-addons that belong to a parent addon.
-- They get their own OnInitialize/OnEnable/OnDisable lifecycle.
-- They can embed libraries independently of the parent.

local GoldTracker = LibStub("AceAddon-3.0"):GetAddon("GoldTracker")
local L = GoldTracker.L

-- Create a module. The second argument onwards are libraries to embed.
local SessionLog = GoldTracker:NewModule("SessionLog", "AceEvent-3.0")

-- Module-level storage
SessionLog.log = {}

function SessionLog:OnInitialize()
	-- Module OnInitialize runs after the parent's OnInitialize.
	-- Access parent addon data via the parent reference:
	-- local parentDB = GoldTracker.db
end

function SessionLog:OnEnable()
	-- Module OnEnable runs after the parent's OnEnable.
	-- Register for the custom message that Core.lua fires:
	self:RegisterMessage("GoldTracker_GoldChanged", "OnGoldChanged")
end

function SessionLog:OnDisable()
	-- Automatically unregisters all events/messages (AceEvent:OnEmbedDisable)
end

function SessionLog:OnGoldChanged(event, currentGold, diff)
	-- Record the change with a timestamp
	local entry = {
		time = time(),
		amount = diff,
		total = currentGold,
	}
	table.insert(self.log, entry)

	-- Keep the log from growing unbounded (last 200 entries)
	while #self.log > 200 do
		table.remove(self.log, 1)
	end
end

-- Public API for other modules or external addons
function SessionLog:GetLog()
	return self.log
end

function SessionLog:GetEntryCount()
	return #self.log
end
```

---

## Ace3 Addon File Structure

### Small Addon (single purpose, <1000 lines)

```
MyAddon/
  MyAddon.toc
  Libs/                  -- Embedded Ace3 libraries
    LibStub/
    CallbackHandler-1.0/
    AceAddon-3.0/
    AceDB-3.0/
    AceEvent-3.0/
    AceConsole-3.0/
  Locales/
    enUS.lua
  Core.lua               -- All logic in one file
```

### Medium Addon (multiple features, 1000-5000 lines)

```
MyAddon/
  MyAddon.toc
  Libs/
    LibStub/
    CallbackHandler-1.0/
    AceAddon-3.0/
    AceDB-3.0/
    AceDBOptions-3.0/
    AceEvent-3.0/
    AceConsole-3.0/
    AceConfig-3.0/       -- Contains AceConfigRegistry, AceConfigDialog, AceConfigCmd
    AceGUI-3.0/
  Locales/
    enUS.lua
    deDE.lua
    zhCN.lua
  Core.lua               -- Addon object, initialization, events
  Options.lua            -- AceConfig options table
  UI.lua                 -- Frame creation and layout
```

### Large Addon (framework-level, 5000+ lines)

```
MyAddon/
  MyAddon.toc
  Libs/
    LibStub/
    CallbackHandler-1.0/
    AceAddon-3.0/
    AceBucket-3.0/
    AceComm-3.0/
    AceConfig-3.0/
    AceConsole-3.0/
    AceDB-3.0/
    AceDBOptions-3.0/
    AceEvent-3.0/
    AceGUI-3.0/
    AceHook-3.0/
    AceLocale-3.0/
    AceSerializer-3.0/
    AceTimer-3.0/
    LibDataBroker-1.1/
    LibDBIcon-1.0/
  Locales/
    enUS.lua
    deDE.lua
    frFR.lua
    koKR.lua
    zhCN.lua
    zhTW.lua
  Core.lua               -- Addon object, slash commands
  Config.lua             -- AceConfig registration
  Options/
    General.lua          -- General options group
    Display.lua          -- Display options group
    Profiles.lua         -- Profile panel setup
  Modules/
    FeatureA.lua         -- NewModule("FeatureA")
    FeatureB.lua         -- NewModule("FeatureB")
    FeatureC.lua         -- NewModule("FeatureC")
  UI/
    MainFrame.lua
    MinimapButton.lua
    Widgets/
      CustomWidget.lua
  Utils.lua              -- Shared utility functions
```

---

## Embedding & Distribution

### How to Embed Ace3

Your addon should include its own copy of each Ace3 library it uses, inside a `Libs/` directory. This ensures your addon works even if the standalone Ace3 addon is not installed.

Each library is a directory containing a `.lua` file and an `.xml` file. You only need the `.lua` file when embedding -- the `.xml` files are for the standalone Ace3 addon's own loading.

**Directory structure for embedding:**

```
MyAddon/
  Libs/
    LibStub/
      LibStub.lua
    CallbackHandler-1.0/
      CallbackHandler-1.0.lua
    AceAddon-3.0/
      AceAddon-3.0.lua
    AceEvent-3.0/
      AceEvent-3.0.lua
    AceDB-3.0/
      AceDB-3.0.lua
    ... (only the libs you actually use)
```

**TOC file references each .lua directly:**

```toc
Libs\LibStub\LibStub.lua
Libs\CallbackHandler-1.0\CallbackHandler-1.0.lua
Libs\AceAddon-3.0\AceAddon-3.0.lua
Libs\AceEvent-3.0\AceEvent-3.0.lua
Libs\AceDB-3.0\AceDB-3.0.lua
```

### Load Order Requirements

The load order in your TOC file is critical. Libraries that depend on others must come after their dependencies:

1. **LibStub** -- Always first. Every library uses LibStub to register itself.
2. **CallbackHandler-1.0** -- Second. Required by AceEvent, AceComm, AceConfigRegistry, and AceDB's callback system.
3. **AceAddon-3.0** -- The addon framework. Load before any library that will be embedded into your addon.
4. **AceEvent-3.0** -- Required by AceBucket (AceBucket lazily binds to AceEvent and AceTimer).
5. **AceTimer-3.0** -- Required by AceBucket.
6. **AceBucket-3.0** -- Depends on AceEvent + AceTimer.
7. **All other Ace3 libs** -- AceDB, AceConsole, AceHook, AceLocale, AceSerializer, AceComm, AceGUI, AceConfig, AceDBOptions -- can load in any order after the above.
8. **Your locale files** -- Default locale first, then translations.
9. **Your addon files** -- Core.lua, Options.lua, modules, etc.

### .pkgmeta for CurseForge/WowInterface Packaging

When you upload to CurseForge, the packager can automatically fetch Ace3 libraries so you don't have to check them into your repository. Create a `.pkgmeta` file in your addon root:

```yaml
package-as: GoldTracker

externals:
  Libs/LibStub:
    url: https://repos.wowace.com/wow/libstub/trunk
    tag: latest
  Libs/CallbackHandler-1.0:
    url: https://repos.wowace.com/wow/callbackhandler/trunk/CallbackHandler-1.0
    tag: latest
  Libs/AceAddon-3.0:
    url: https://repos.wowace.com/wow/ace3/trunk/AceAddon-3.0
    tag: latest
  Libs/AceDB-3.0:
    url: https://repos.wowace.com/wow/ace3/trunk/AceDB-3.0
    tag: latest
  Libs/AceDBOptions-3.0:
    url: https://repos.wowace.com/wow/ace3/trunk/AceDBOptions-3.0
    tag: latest
  Libs/AceEvent-3.0:
    url: https://repos.wowace.com/wow/ace3/trunk/AceEvent-3.0
    tag: latest
  Libs/AceConsole-3.0:
    url: https://repos.wowace.com/wow/ace3/trunk/AceConsole-3.0
    tag: latest
  Libs/AceBucket-3.0:
    url: https://repos.wowace.com/wow/ace3/trunk/AceBucket-3.0
    tag: latest
  Libs/AceTimer-3.0:
    url: https://repos.wowace.com/wow/ace3/trunk/AceTimer-3.0
    tag: latest
  Libs/AceConfig-3.0:
    url: https://repos.wowace.com/wow/ace3/trunk/AceConfig-3.0
    tag: latest
  Libs/AceGUI-3.0:
    url: https://repos.wowace.com/wow/ace3/trunk/AceGUI-3.0
    tag: latest
  Libs/AceLocale-3.0:
    url: https://repos.wowace.com/wow/ace3/trunk/AceLocale-3.0
    tag: latest

ignore:
  - .git
  - .github
  - README.md
  - .pkgmeta
```

### External vs Embedded Dependencies

**Embedded** (recommended): Your addon bundles its own copy of each library. LibStub ensures only one copy runs -- the highest version among all addons wins. This is the standard approach.

**External** (via `## OptionalDeps: Ace3`): Your addon lists Ace3 as an optional dependency, which makes the WoW client load the standalone Ace3 addon before yours. This guarantees the libraries exist by the time your files load. However, if the user doesn't have standalone Ace3 installed, your embedded copies still work as a fallback.

Always embed AND list as OptionalDeps:
```toc
## OptionalDeps: Ace3
```

This gives you the best of both worlds: if the user has a newer standalone Ace3, its higher-version libraries take precedence. If they don't have standalone Ace3, your embedded copies work fine.

---

## Common Patterns & Recipes

### Profile Change Handling

When the user switches, copies, or resets a profile, you need to refresh any cached settings or UI elements that read from the profile.

```lua
function MyAddon:OnInitialize()
	self.db = LibStub("AceDB-3.0"):New("MyAddonDB", defaults, true)

	-- Register for ALL profile-change callbacks
	self.db.RegisterCallback(self, "OnProfileChanged", "RefreshConfig")
	self.db.RegisterCallback(self, "OnProfileCopied", "RefreshConfig")
	self.db.RegisterCallback(self, "OnProfileReset", "RefreshConfig")
end

function MyAddon:RefreshConfig()
	-- Re-read settings from self.db.profile and apply them
	-- e.g., update frame positions, toggle features, recolor elements
	if self.mainFrame then
		self.mainFrame:SetAlpha(self.db.profile.alpha)
		self.mainFrame:ClearAllPoints()
		self.mainFrame:SetPoint(unpack(self.db.profile.position))
	end

	-- Notify modules to refresh too
	self:SendMessage("MyAddon_ConfigRefreshed")
end
```

### Cross-Addon Communication with AceComm + AceSerializer

```lua
local MyAddon = LibStub("AceAddon-3.0"):NewAddon(
	"MyAddon", "AceComm-3.0", "AceSerializer-3.0", "AceEvent-3.0"
)

local COMM_PREFIX = "MyAdnSync" -- Max 16 characters!

function MyAddon:OnEnable()
	-- Register to receive messages on our prefix
	self:RegisterComm(COMM_PREFIX)
end

-- Send data to the raid/party
function MyAddon:BroadcastData(data)
	-- Serialize the Lua table into a string
	local serialized = self:Serialize(data)

	-- Send via addon channel. Distribution can be:
	-- "PARTY", "RAID", "GUILD", "WHISPER" (requires target), etc.
	-- Priority: "ALERT" (immediate), "NORMAL" (default), "BULK" (lowest)
	self:SendCommMessage(COMM_PREFIX, serialized, "RAID", nil, "NORMAL")
end

-- Callback when a message is received. AceComm calls OnCommReceived by default.
-- Arguments: prefix, message, distribution, sender
function MyAddon:OnCommReceived(prefix, message, distribution, sender)
	-- Don't process our own messages
	if sender == UnitName("player") then return end

	-- Deserialize the string back into a Lua table
	local success, data = self:Deserialize(message)
	if not success then
		-- data contains the error message on failure
		return
	end

	-- Process the received data
	-- data is now the original Lua table
end
```

### Slash Command with Subcommands Using GetArgs

```lua
local MyAddon = LibStub("AceAddon-3.0"):NewAddon("MyAddon", "AceConsole-3.0")

function MyAddon:OnInitialize()
	self:RegisterChatCommand("myaddon", "HandleSlash")
	self:RegisterChatCommand("ma", "HandleSlash")
end

function MyAddon:HandleSlash(input)
	-- GetArgs(string, numArgs, startPos) -> arg1, ..., nextPos
	-- nextPos is 1e9 when at end of string
	local cmd, nextPos = self:GetArgs(input, 1)

	if not cmd or cmd == "" then
		-- No argument: open config
		LibStub("AceConfigDialog-3.0"):Open("MyAddon")
		return
	end

	cmd = cmd:lower()

	if cmd == "show" then
		self:ShowMainFrame()
	elseif cmd == "hide" then
		self:HideMainFrame()
	elseif cmd == "set" then
		-- Parse two more arguments starting from nextPos
		local key, value, _ = self:GetArgs(input, 2, nextPos)
		if key and value then
			self:Print(("Setting %s = %s"):format(key, value))
		else
			self:Print("Usage: /ma set <key> <value>")
		end
	elseif cmd == "help" then
		self:Print("Commands: show, hide, set <key> <value>, help")
	else
		self:Print("Unknown command: " .. cmd .. ". Type /ma help for usage.")
	end
end
```

### Creating a Standalone Options Window

```lua
-- Open a standalone AceConfigDialog window (NOT embedded in Blizzard Settings)
function MyAddon:OpenConfig()
	-- Open() creates a movable, resizable window with the options tree
	LibStub("AceConfigDialog-3.0"):Open("MyAddon")
end

-- Close it programmatically
function MyAddon:CloseConfig()
	LibStub("AceConfigDialog-3.0"):Close("MyAddon")
end

-- Set the default size before the first Open()
function MyAddon:OnInitialize()
	-- ...
	LibStub("AceConfigDialog-3.0"):SetDefaultSize("MyAddon", 700, 500)
end
```

### Event Bucketing for Bag/Inventory Updates

```lua
local MyAddon = LibStub("AceAddon-3.0"):NewAddon("MyAddon", "AceBucket-3.0")

function MyAddon:OnEnable()
	-- Single event bucket: fires callback at most once per 0.2 seconds
	self:RegisterBucketEvent("BAG_UPDATE", 0.2, "ScanBags")

	-- Multiple events in one bucket: any of these events trigger the same
	-- 1-second collection window, then fires the callback once
	self:RegisterBucketEvent(
		{"UNIT_HEALTH", "UNIT_MAXHEALTH", "UNIT_ABSORB_AMOUNT_CHANGED"},
		1.0,
		"UpdateHealthDisplay"
	)
end

-- The callback receives a table of arg1 values.
-- Keys are the arg1 values, values are how many times each fired.
function MyAddon:ScanBags(bagIDs)
	-- bagIDs looks like: { [0] = 3, [1] = 1, [4] = 2 }
	-- meaning: bag 0 fired 3 times, bag 1 once, bag 4 twice
	for bagID, count in pairs(bagIDs) do
		-- Scan this bag
	end
end

function MyAddon:UpdateHealthDisplay(units)
	-- units looks like: { ["player"] = 5, ["target"] = 2, ["party1"] = 1 }
	if units["player"] then
		-- Update player health display
	end
	if units["target"] then
		-- Update target health display
	end
end
```

### Timer-Based Polling

```lua
local MyAddon = LibStub("AceAddon-3.0"):NewAddon("MyAddon", "AceTimer-3.0")

function MyAddon:OnEnable()
	-- One-shot timer: fires once after 5 seconds
	self:ScheduleTimer("DelayedInit", 5)

	-- Repeating timer: fires every 0.5 seconds until canceled
	-- Store the handle so you can cancel it later
	self.combatCheckTimer = self:ScheduleRepeatingTimer("CheckCombatState", 0.5)
end

function MyAddon:DelayedInit()
	-- Runs once, 5 seconds after OnEnable
end

function MyAddon:CheckCombatState()
	if InCombatLockdown() then
		-- Player is in combat
	else
		-- Player is out of combat, maybe cancel the timer
		-- self:CancelTimer(self.combatCheckTimer)
	end
end

function MyAddon:OnDisable()
	-- CancelAllTimers() is called automatically by AceTimer's OnEmbedDisable,
	-- but you can also cancel specific timers manually:
	if self.combatCheckTimer then
		self:CancelTimer(self.combatCheckTimer)
		self.combatCheckTimer = nil
	end
end
```

### Hooking a Blizzard Function Safely

```lua
local MyAddon = LibStub("AceAddon-3.0"):NewAddon("MyAddon", "AceHook-3.0")

function MyAddon:OnEnable()
	-- SecureHook: called AFTER the original function (post-hook).
	-- Cannot modify arguments or return values. Cannot block execution.
	-- Safe to use on secure functions -- does not break protected status.
	self:SecureHook("ToggleCharacter", "OnToggleCharacter")

	-- SecureHookScript: same concept but for frame scripts
	self:SecureHookScript(CharacterFrame, "OnShow", "OnCharacterFrameShow")

	-- RawHook: REPLACES the original function. You MUST call the original
	-- yourself via self.hooks[method] or self.hooks[object][method].
	-- WARNING: Cannot be used on secure functions unless you pass hookSecure=true.
	self:RawHook("GetQuestReward", "MyGetQuestReward", true)
end

function MyAddon:OnToggleCharacter(tab)
	-- Called AFTER ToggleCharacter(tab) runs
	-- 'tab' is the same argument the original received
end

function MyAddon:OnCharacterFrameShow(frame)
	-- Called AFTER CharacterFrame:OnShow fires
end

function MyAddon:MyGetQuestReward(choice)
	-- We replaced GetQuestReward -- call the original!
	self.hooks.GetQuestReward(choice)
	-- Now do our post-processing
end

-- Unhooking is handled automatically on disable, or manually:
function MyAddon:SomeCleanup()
	self:Unhook("ToggleCharacter")
	self:Unhook(CharacterFrame, "OnShow")
end
```

---

## Troubleshooting & Common Mistakes

### 1. Missing CallbackHandler-1.0

**Symptom:** Error on login: `Cannot find a library instance of "CallbackHandler-1.0"` or AceEvent/AceComm/AceDB fail to load.

**Cause:** CallbackHandler-1.0 is not listed in your TOC file, or it loads after the libraries that need it.

**Fix:** Always include CallbackHandler-1.0 in your TOC, immediately after LibStub:

```toc
Libs\LibStub\LibStub.lua
Libs\CallbackHandler-1.0\CallbackHandler-1.0.lua   <-- Must come second
Libs\AceAddon-3.0\AceAddon-3.0.lua
```

### 2. Wrong TOC Load Order

**Symptom:** `attempt to index a nil value` errors in Ace3 library code, or LibStub is undefined.

**Cause:** Libraries are listed in the wrong order. LibStub must come before everything; CallbackHandler before AceEvent, AceComm, etc.

**Fix:** Follow the load order from the Embedding section above. When in doubt, match the order in Ace3's own `Ace3.toc` file.

### 3. Registering Events in OnInitialize

**Symptom:** Events never fire, or errors about missing methods.

**Cause:** `self:RegisterEvent()` in `OnInitialize`. The AceEvent mixin methods are available on the object during OnInitialize, but the embedded library's `OnEmbedInitialize` sets things up. Event registration works technically, but the game world may not be ready (PLAYER_LOGIN hasn't fired yet). More importantly, it violates the Ace3 lifecycle contract.

**Fix:** Always register events in `OnEnable`, not `OnInitialize`:

```lua
function MyAddon:OnInitialize()
	-- DB setup, config registration, slash commands ONLY
end

function MyAddon:OnEnable()
	self:RegisterEvent("PLAYER_MONEY")    -- Correct place for events
	self:RegisterBucketEvent("BAG_UPDATE", 0.2, "OnBags")
end
```

### 4. AceDB Defaults Stripped on Logout

**Symptom:** After logging out and back in, settings appear to revert to defaults, or checking `self.db.profile.mySetting == nil` returns true even though you set a default.

**Cause:** AceDB intentionally strips default values from SavedVariables on logout (via `removeDefaults`). If a value in the profile matches the default, it gets removed from the saved file. This saves disk space and ensures changed defaults propagate correctly. The value is still accessible through AceDB's metatable magic -- it just doesn't exist in the raw table.

**Fix:** Never check `if self.db.profile.mySetting == nil then`. The value will never be nil when defaults are registered. If you need to detect "user has never changed this setting", use a sentinel:

```lua
local defaults = {
	profile = {
		mySetting = "DEFAULT_SENTINEL",
	}
}

-- Check if user has explicitly set the value:
if self.db.profile.mySetting == "DEFAULT_SENTINEL" then
	-- User hasn't changed it yet
end
```

### 5. AceConfig Options in Random Order

**Symptom:** Options appear in a different order every time you open the config panel.

**Cause:** AceConfig iterates `args` tables with `pairs()`, which returns keys in an unpredictable order. Without explicit `order` values, the layout is random.

**Fix:** Always set `order` on every option:

```lua
args = {
	option1 = { type = "toggle", name = "First",  order = 1 },
	option2 = { type = "toggle", name = "Second", order = 2 },
	option3 = { type = "range",  name = "Third",  order = 3 },
}
```

### 6. AceGUI Widgets Must Be Released

**Symptom:** Memory leaks, widgets that stop working, or errors about widgets already being used.

**Cause:** AceGUI uses an object pool. When you create a widget with `AceGUI:Create()`, you must call `widget:Release()` when you're done with it -- not just hide it or nil the reference.

**Fix:**

```lua
local frame = AceGUI:Create("Frame")
-- ... use it ...

-- When done:
frame:Release()  -- Returns it to the pool
-- Do NOT do: frame:Hide() and forget about it
```

Note: Widgets inside AceConfigDialog-managed panels are handled automatically. This only applies when you create AceGUI widgets manually.

### 7. AceComm Prefix Length Limit

**Symptom:** `AceComm:RegisterComm(prefix,method): prefix length is limited to 16 characters`

**Cause:** The WoW addon message API limits prefixes to 16 characters. This is a game engine limitation, not an Ace3 one.

**Fix:** Keep prefixes short. Use abbreviations:

```lua
-- BAD: 20 characters
self:RegisterComm("MyAddonDataSync01")

-- GOOD: 10 characters
self:RegisterComm("MASyncData")
```

### 8. AceTimer Minimum Delay

**Symptom:** Timer fires much slower than expected when using very small intervals.

**Cause:** AceTimer enforces a minimum delay of 0.01 seconds (10 milliseconds). Any value below this is clamped to 0.01. This matches the minimum granularity of the underlying `C_Timer.After` API.

**Fix:** Don't schedule timers faster than 0.01s. If you need per-frame updates, use an `OnUpdate` script instead of AceTimer.

### 9. Printing in OnInitialize

**Symptom:** `self:Print()` in OnInitialize produces no output, or errors about DEFAULT_CHAT_FRAME being nil.

**Cause:** `OnInitialize` fires during ADDON_LOADED, which can happen before the chat frame exists. `self:Print()` (from AceConsole) writes to `DEFAULT_CHAT_FRAME`, which may be nil at that point.

**Fix:** Move user-facing messages to `OnEnable` (fires at PLAYER_LOGIN, when the UI is fully loaded):

```lua
function MyAddon:OnInitialize()
	-- self:Print("Loaded!")  -- DON'T: chat frame may not exist
end

function MyAddon:OnEnable()
	self:Print("Loaded!")    -- OK: chat frame is ready
end
```

### 10. Module OnEnable Not Firing

**Symptom:** You create a module with `NewModule`, define `OnEnable`, but it never gets called.

**Cause:** The parent addon called `self:SetDefaultModuleState(false)` before creating modules. When the default state is disabled, modules won't auto-enable.

**Fix:** Either remove `SetDefaultModuleState(false)`, or explicitly enable modules:

```lua
-- If you use SetDefaultModuleState(false), you must enable modules manually:
function MyAddon:OnEnable()
	self:EnableModule("MyModule")
end

-- Or let the module enable itself on creation:
local mod = MyAddon:NewModule("MyModule", "AceEvent-3.0")
mod:SetEnabledState(true)  -- Override the default disabled state
```

---

## Ace3 vs Base WoW API Quick Reference

| Task | Ace3 Way | Base WoW API |
|------|----------|-------------|
| Create addon object | `LibStub("AceAddon-3.0"):NewAddon("Name", ...)` | `local ns = select(2, ...)` in each file |
| Register event | `self:RegisterEvent("EVENT")` | `frame:RegisterEvent("EVENT")` + OnEvent script |
| Unregister event | `self:UnregisterEvent("EVENT")` | `frame:UnregisterEvent("EVENT")` |
| Throttle burst events | `self:RegisterBucketEvent("EVENT", 0.5, "Handler")` | Manual timer + flag in OnUpdate or C_Timer |
| One-shot timer | `self:ScheduleTimer("Method", delay)` | `C_Timer.After(delay, func)` |
| Repeating timer | `self:ScheduleRepeatingTimer("Method", delay)` | `C_Timer.NewTicker(delay, func)` |
| Cancel timer | `self:CancelTimer(handle)` | `ticker:Cancel()` |
| Saved variables | `AceDB:New("SavedVarName", defaults, true)` | `ADDON_LOADED` handler + manual defaults merge |
| Profile switching | Built into AceDB with callbacks | Must implement entirely from scratch |
| Slash command | `self:RegisterChatCommand("cmd", "Handler")` | `SLASH_NAME1 = "/cmd"` + `SlashCmdList["NAME"]` |
| Parse slash args | `self:GetArgs(input, 2)` | Manual `string.match` / `strsplit` |
| Print to chat | `self:Print("message")` | `print("message")` or `DEFAULT_CHAT_FRAME:AddMessage()` |
| Hook function (post) | `self:SecureHook("FuncName", "Handler")` | `hooksecurefunc("FuncName", handler)` |
| Hook function (replace) | `self:RawHook("FuncName", "Handler")` | Save original, replace global, call saved copy |
| Unhook cleanly | `self:Unhook("FuncName")` | Restore saved original (fragile if others hooked too) |

| Task | Ace3 Way | Base WoW API |
|------|----------|-------------|
| Options UI | AceConfig options table + AceConfigDialog | Manual frame creation with widgets |
| Blizzard Settings panel | `AceConfigDialog:AddToBlizOptions()` | `Settings.RegisterCanvasLayoutCategory()` |
| Localization | AceLocale:NewLocale / GetLocale | Manual locale tables with GetLocale() checks |
| Send addon message | `self:SendCommMessage(prefix, text, dist)` | `C_ChatInfo.SendAddonMessage()` (255 byte limit) |
| Receive addon message | `self:RegisterComm(prefix)` | `RegisterEvent("CHAT_MSG_ADDON")` + manual parsing |
| Auto-split long messages | Built into AceComm | Must implement chunking manually |
| Serialize Lua data | `self:Serialize(table)` | Manual string encoding |
| Module system | `addon:NewModule("Name", ...)` | Manual table-based namespacing |
| Cross-addon messages | `self:SendMessage("MSG")` / `RegisterMessage` | Custom callbacks or global event frames |
| Auto-cleanup on disable | Automatic (OnEmbedDisable) | Must manually track and undo everything |
