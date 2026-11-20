LibDropDownExtension
==========================
This library can be used to add additional options to dropdown menus without causing taint. Until the core issue is resolved, this is an alternative strategy to provide a way to add options to existing menus. Luckily the new Menu system is used everywhere and the API is meant as a drop-in replacement which means it should be very easy to implement in your own projects.

The game has two dropdown systems and both are supported:

| System | Used by | API |
|---|---|---|
| The modern `Menu` system | Practically everything on all game versions | [`DDE.Menu.ModifyMenu`](#menu) |
| The legacy `UIDropDownMenu` system | Exclusively a PVP related menu in Mists of Pandaria | [`DDE.RegisterEvent`](#uidropdownmenu) |

# Getting started

The library needs [LibStub](https://www.curseforge.com/wow/addons/libstub). Load `embed.xml` to load the library with the files it needs:

```
libs\LibStub\LibStub.lua
libs\LibDropDownExtension\embed.xml
```

Then get the library like any other:

```lua
local DDE = LibStub("LibDropDownExtension-1.0", true)
local ModifyMenu = DDE and DDE.Menu.ModifyMenu or Menu.ModifyMenu
-- ModifyMenu is either the library API or it can fallback to the Blizzard API if the library wasn't available
```

There are `DDE.HasBackend("Menu")` and `DDE.HasBackend("Legacy")` tell you if a system is available, if you ever needed to know.

All of the functions are called with a dot, like `DDE.RegisterEvent(...)`.

# Menu

`local handle = DDE.Menu.ModifyMenu(tags, callback)`

This is a drop-in replacement for [`Menu.ModifyMenu`](https://github.com/Gethe/wow-ui-source/blob/live/Interface/AddOns/Blizzard_Menu/11_0_0_MenuImplementationGuide.lua). You use it the same way as you would the Blizzard version: your callback is called with the owner of the menu, a `rootDescription` and the `contextData` every time a menu with one of your tags opens, and the calls you make on the `rootDescription` add elements to the menu.

```lua
local function OnMenu(owner, rootDescription, contextData)
    rootDescription:CreateDivider()
    rootDescription:CreateTitle("My addon")
    rootDescription:CreateButton("Say hello", function(data, inputData, menu)
        print("Hello!", data, inputData.buttonName)
    end, "some data")
end

local handle = DDE.Menu.ModifyMenu("MENU_UNIT_PLAYER MENU_UNIT_FRIEND", OnMenu)
-- you can unregister later by calling: handle:Unregister()
```

## tags

A tag of a menu, like `MENU_UNIT_PLAYER`. You can pass one tag, several tags separated by a space, or a table of tags. The tag of a unit menu is `MENU_UNIT_<UNIT_TYPE>`, and you can see the tags of the menus that are open by calling `Menu.PrintOpenMenuTags()` in the game.

## callback arguments

`owner` - the region that the menu belongs to.

`rootDescription` - add your elements to it, see below. It has a `tag` and a `contextData` field.

`contextData` - what the game knows about the menu, or `nil`. For the unit menus it is the table that is given to [`UnitPopupManager:OpenMenu`](https://github.com/Gethe/wow-ui-source/blob/live/Interface/AddOns/Blizzard_UnitPopupShared/UnitPopupShared.lua), with fields like `which`, `unit`, `name`, `server` and `accountInfo`. Which fields are set depends on where the menu was opened.

Add nothing if you don't want to change the menu.

## return value

A handle with `handle:Unregister()`, or `nil` if the modern Menu system isn't available.

## rootDescription

These are the calls that are supported, with the same names and arguments as the ones of the game, and they return the element that they added:

| Call | |
|---|---|
| `CreateTitle(text)` | A line of text. |
| `CreateDivider()` | A divider line. |
| `CreateSpacer([extent])` | Empty room, 10 high by default. |
| `CreateButton(text[, callback[, data]])` | A button. The callback is called as `callback(data, inputData, menu)`, where `inputData.buttonName` is the mouse button that was used. |
| `CreateCheckbox(text, isSelected, setSelected[, data])` | A checkbox. `isSelected(data)` tells if it is checked, and `setSelected(data)` is called when it is clicked, so change your own state in it. |
| `CreateRadio(text, isSelected, setSelected[, data])` | A radio button, it works like the checkbox. |

On the element that you get back: `element:SetEnabled(enabled)` and `element:SetKeepShown(keepShown)`, which can be chained. A button closes the menu when it is clicked unless you keep it shown, a checkbox or radio button stays open unless you tell it not to.

Anything else, like submenus, tooltips, color swatches or custom frames, isn't supported yet. Calling a method that isn't supported raises an error that tells you which one, so a typo can't silently do nothing.

# UIDropDownMenu

The legacy dropdown menu doesn't have an official way to modify menus, so you must register a callback which will later be called for you to evaluate if you wish to insert your options to the dropdown. The data you pass is however a drop-in replacement as the same info table is used as you would use with `UIDropDownMenu_AddButton`. This means you should practically be able to with a bit of scaffolding rewrite your existing code to utilize the library.

`DDE.RegisterEvent(events, func[, levels[, data]])`

`DDE.UnregisterEvent(events, func[, levels])`

`DDE.Option.Separator`

`DDE.Option.Space`

Both functions return a boolean, `false` if the legacy system isn't available.

## events

You can provide one or a mix of events that will call your callback function when a dropdown menu event occurs:

`"OnShow"`

`"OnHide"`

`"OnShow OnHide"`

## func (arguments)

The callback function has the following arguments passed to it when called from the library:

`dropdown` - the game dropdown frame that owns the list that is opening, you can inspect it to decide if you want to add options

`event` - the event `OnShow` or `OnHide` that is triggering the callback

`options` - a table you need to populate with options that you wish to add to the dropdown

`level` - the level of the dropdown

`data` - your previously passed object from registering this callback function

On `"OnShow"`, fill `options` and return `true` to add them, or return a table of options to use instead. Return `false` (or nothing) to add nothing. On `"OnHide"` you can clear your options again.

## options

An option is the same table that you give to `UIDropDownMenu_AddButton`. All of its fields are optional, and you can find them in [the game's UIDropDownMenu.lua](https://github.com/Gethe/wow-ui-source/blob/ab5cb850a3f65e742959e379c4689353a2f4549f/Interface/AddOns/Blizzard_SharedXML/Mainline/UIDropDownMenu.lua#L281-L336) (the [classic version](https://github.com/Gethe/wow-ui-source/blob/4192ac60067fc55566182291bc67c85dd9b6794d/Interface/AddOns/Blizzard_SharedXML/Classic/UIDropDownMenu.lua#L254-L295) has fewer). Some of the fields only exist in some of the games.

When an option is clicked, `func` is called as `func(button, arg1, arg2, checked[, mouseButton])`, where `mouseButton` is only given in the games that give it to the dropdown of the game too.

`DDE.Option.Separator` and `DDE.Option.Space` are ready made options that behave like a separator line or a space, you can use them in your `options` without defining them yourself. The library already adds a separator between the options of the game and yours.

Note that you can either re-use the `options` you are provided in the callback, or you can give it a new table with `options`. The library will update the internal table which you are provided in either case.

## levels (optional)

The level of the dropdown list, `1` to `3`, or `true` for any level. It specifies when you want the callback to be executed, otherwise omit it to run at every dropdown level.

## data (optional)

Optional data (a table) to be passed to the callback function for convenience if you need to pass something unique when registering your callback function. If you pass nothing, the callback gets an empty table.

## Example

```lua
local DDE = LibStub("LibDropDownExtension-1.0", true)
-- it's recommended to have the options prepared before hand so they can be re-used on demand
local dropdownOptions = {
    {
        text = "Click me to print some stuff!",
        notCheckable = true,
        func = function(...) print("You clicked me!", ...) end,
    },
    DDE.Option.Separator,
    {
        text = "This is after the separator",
        notCheckable = true,
        func = print,
    },
    DDE.Option.Space,
    {
        text = "This has a space above",
        notCheckable = true,
        func = print,
    },
}
-- the callback function for when the dropdown event occurs
-- if you need additional logic you are provided the dropdown itself that you can use to decide if you wish to add options or not
local function OnEvent(dropdown, event, options)
    if event == "OnShow" then
        -- show our dropdown options to any dropdown menu
        return dropdownOptions
    elseif event == "OnHide" then
        -- you can perform cleanup if you need
    end
end
-- registers our callback function for the show and hide events for the first dropdown level only
DDE.RegisterEvent("OnShow OnHide", OnEvent, 1)
```

# Order and errors

- Your options are shown below the ones of the game. The callbacks are shown in the order that they were registered in, and the options of one callback stay together, so the options of two addons never mix. There is no priority to set.
- The legacy system puts one separator between the options of the game and the first callback that has options. The modern system adds nothing on its own, so add your own divider or title like you would with the game's `Menu.ModifyMenu`.
- An error in your callback is reported like any other addon error. It does however not interfere with the library routine or the users of the library.

# Upgrading from the previous version

The legacy API has changed and will require to be updated in your code if you already used this library:

- The functions are called with a dot. `DDE:RegisterEvent(...)` is now `DDE.RegisterEvent(...)`, and `DDE:UnregisterEvent(...)` is now `DDE.UnregisterEvent(...)`.
- The dropdown levels are `1` to `3`, meanwhile the old version only had `1` and `2`.

# Typing

`meta.lua` has the annotations that the [Lua language server](https://luals.github.io/) needs to know the types of the game that the library uses, it isn't loaded by anything. The library types itself, and the callbacks are generic over your `data`. To get `data` typed inside your callback, annotate the function:

```lua
---@class MyData
---@field name string

---@type LibDropDownExtension-1.0_LegacyCallback<MyData>
local function OnEvent(dropdown, event, options, level, data)
    print(data.name)
end
```
