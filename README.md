LibDropDownExtension
==========================
This library can be used to expand the dropdown menu with additional options.

# API

`DDE.Option.Separator`

`DDE.Option.Space`

`DDE:RegisterEvent(events, func[, levels[, data]])`

`DDE:UnregisterEvent(events, func[, levels])`

## option (separator and space)

Default entries that behave like a separator line or a space in the dropdown, you can use these templates in your event callback function to specify these entries without defining them yourself.

## events

You can provide one or a mix of events that will call your callback function when a dropdown menu event occurs:

`"OnShow"`

`"OnHide"`

`"OnShow OnHide"`

## func (arguments)

The callback function has the following arguments passed to it when called from the library:

`dropdown` - the game dropdown frame triggering the callback function

`event` - the event `OnShow` or `OnHide` that is triggering the callback

`options` - a table you need to populate with options that you wish to add to the dropdown

`level` - the level of the dropdown

`data` - your previously passed object from registering this callback function

## levels (optional)

The dropdown level is either 1 or 2 and can be used to specify when you want the callback to be executed, otherwise ommit to run at every dropdown level.

## data (optional)

Optional data (usually a table) to be passed to the callback function for convenience if you need to pass something unique when registering your callback function.

# Example

```
local DDE = LibStub("LibDropDownExtension-1.0", true)
-- we prefer to have all the options we wish to show as static tables in memory and just add the entries to the dropdown when needed
-- you can find more information in the lua file on what properties exist and can be used when defining dropdown options
local dropdownOptions = {
	{
		text = "Click me to print some stuff!",
		func = function(...) print("You clicked me!", ...) end,
	},
	DDE.Option.Separator,
	{
		text = "This is after the separator",
		func = print,
	},
	DDE.Option.Space,
	{
		text = "This has a space above",
		func = print,
	},
	DDE.Option.Separator,
	{
		text = "This is after yet another separator",
		func = print,
	},
}
-- the callback function for when the dropdown event occurs
local function OnEvent(dropdown, event, options)
	if event == "OnShow" then
		-- add the dropdown options to the options table
		for i = 1, #dropdownOptions do
			options[i] = dropdownOptions[i]
		end
		-- we have added options to the dropdown menu
		return true
	elseif event == "OnHide" then
		-- when hiding we can remove our dropdown options from the options table
		for i = #options, 1, -1 do
			options[i] = nil
		end
	end
end
-- registers our callback function for the show and hide events for the first dropdown level only
DDE:RegisterEvent("OnShow OnHide", OnEvent, 1)
```
