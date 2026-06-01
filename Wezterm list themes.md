- list themes:
	- ctrl+shift+right
	- ctrl+shift+left
- see current theme: alt+shift+t

Edit config:

powershell → notepad $HOME\.wezterm.lua

insert:

```yaml
local wezterm = require 'wezterm'
local act = wezterm.action

local config = wezterm.config_builder()

local default_theme = 'Tokyo Night'

local function get_builtin_theme_names()
  local schemes = wezterm.color.get_builtin_schemes()
  local names = {}

  for name, _ in pairs(schemes) do
    table.insert(names, name)
  end

  table.sort(names)

  return names
end

local themes = get_builtin_theme_names()

local function index_of(list, value)
  for i, v in ipairs(list) do
    if v == value then
      return i
    end
  end

  return 1
end

local function get_current_theme(window)
  local overrides = window:get_config_overrides() or {}
  return overrides.color_scheme or config.color_scheme or default_theme
end

local function set_theme(window, theme)
  local overrides = window:get_config_overrides() or {}
  overrides.color_scheme = theme
  window:set_config_overrides(overrides)
end

local function cycle_theme(window, direction)
  local current = get_current_theme(window)
  local idx = index_of(themes, current)

  idx = ((idx - 1 + direction) % #themes) + 1

  set_theme(window, themes[idx])
end

local function theme_choices()
  local choices = {}

  for _, theme in ipairs(themes) do
    table.insert(choices, {
      id = theme,
      label = theme,
    })
  end

  return choices
end

local function show_current_theme(window, pane)
  local current = get_current_theme(window)

  window:perform_action(
    act.InputSelector {
      title = 'Current theme: ' .. current,
      choices = {
        {
          id = current,
          label = current,
        },
      },
      fuzzy = false,
      action = wezterm.action_callback(function()
        -- Nothing to do. This is only for showing the current theme.
      end),
    },
    pane
  )
end

-- WSL
config.default_domain = 'WSL:Debian'

-- Theme
config.color_scheme = default_theme

-- Font
config.font = wezterm.font('Hack Nerd Font')
config.font_size = 11

-- Transparency
config.window_background_opacity = 0.95
config.text_background_opacity = 1.0

-- Windows 11 blur
config.win32_system_backdrop = 'Acrylic'

-- Window
config.initial_cols = 140
config.initial_rows = 40

config.window_padding = {
  left = 15,
  right = 15,
  top = 15,
  bottom = 15,
}

-- Hide title bar
config.window_decorations = 'RESIZE'

-- Cursor
config.default_cursor_style = 'BlinkingBar'

-- Scrollback
config.scrollback_lines = 10000

-- Better rendering
config.front_end = 'WebGpu'

-- Disable annoying bell
config.audible_bell = 'Disabled'

-- Keys
config.keys = {
  {
    key = 'Space',
    mods = 'CTRL',
    action = act.SendString('\0'),
  },

  -- Next theme
  {
    key = 'RightArrow',
    mods = 'CTRL|SHIFT',
    action = wezterm.action_callback(function(window, pane)
      cycle_theme(window, 1)
    end),
  },

  -- Previous theme
  {
    key = 'LeftArrow',
    mods = 'CTRL|SHIFT',
    action = wezterm.action_callback(function(window, pane)
      cycle_theme(window, -1)
    end),
  },

  -- Pick theme from all builtin WezTerm themes
  {
    key = 'T',
    mods = 'CTRL|SHIFT',
    action = act.InputSelector {
      title = 'Select WezTerm theme',
      choices = theme_choices(),
      fuzzy = true,
      action = wezterm.action_callback(function(window, pane, id, label)
        if id then
          set_theme(window, id)
        end
      end),
    },
  },

  -- Show current theme
  {
    key = 'T',
    mods = 'ALT',
    action = wezterm.action_callback(function(window, pane)
      show_current_theme(window, pane)
    end),
  },
}

return config
```

Interesting themes:

```
  -- Theme — see https://wezterm.org/colorschemes/a/index.html
  --color_scheme = "Sonokai (Gogh)",
  --color_scheme = "Tokyo Night",
  --color_scheme = "3024 Night (Gogh)",
  --color_scheme = "Afterglow",
  --color_scheme = "Alien Blood (Gogh)",
  --color_scheme = "Apprentice (Gogh)",
  --color_scheme = "Arthur",
  --color_scheme = "Ashes (base16)",
  --color_scheme = "Atelier Savanna (base16)",
  --color_scheme = "ayu",
  --color_scheme = "Tomorrow Night (Gogh)",
  --color_scheme = "Trim Yer Beard (terminal.sexy)",
  --color_scheme = "VWbug (terminal.sexy)",
  --color_scheme = "Violet Dark",
  --color_scheme = "Visibone Alt. 2 (terminal.sexy)",
  --color_scheme = "Wombat",
  --color_scheme = "Woodland (base64)",
  --color_scheme = "X::Erosion (terminal.sexy)",
  --color_scheme = "darkmoss (base64)",
  --color_scheme = "duskfox",
  --color_scheme = "flexoki-dark",
  --color_scheme = "jmbi (terminal.sexy)",
  --color_scheme = "kanagawabones",
  --color_scheme = "nordfox",
  --color_scheme = "rose-pine",
  --color_scheme = "s3r0 modified (terminal.sexy)",
  --color_scheme = "zenwritten_dark",
  --color_scheme = "Decaf (base16)",
  --color_scheme = "Default (dark) (terminal.sexy)",
  --color_scheme = "Gogh (Gogh)",
  --color_scheme = "Gruvbox Dark (Gogh)",
  --color_scheme = "Gruvbox Material (Gogh)",
  --color_scheme = "Gruvbox dark, pale (base16)",
  --color_scheme = "Hardcore",
  --color_scheme = "Hybrid",
  --color_scheme = "Japanesque (Gogh)",
  --color_scheme = "Ocean Dark (Gogh)",
  --color_scheme = "OneDark (Gogh)",
  --color_scheme = "Operator Mono Dark",
  --color_scheme = "Red Planet",
  --color_scheme = "Rippedcasts",
  --color_scheme = "Srcery (Gogh)",
```

original config:

```yaml
from pathlib import Path

config = """local wezterm = require 'wezterm'

return {
  -- WSL
  default_domain = 'WSL:Debian',

  -- Theme — see https://wezterm.org/colorschemes/a/index.html
  --color_scheme = "Sonokai (Gogh)",
  --color_scheme = "Tokyo Night",
  --color_scheme = "3024 Night (Gogh)",
  --color_scheme = "Afterglow",
  --color_scheme = "Alien Blood (Gogh)",
  --color_scheme = "Apprentice (Gogh)",
  --color_scheme = "Arthur",
  --color_scheme = "Ashes (base16)",
  --color_scheme = "Atelier Savanna (base16)",
  --color_scheme = "ayu",
  --color_scheme = "Tomorrow Night (Gogh)",
  --color_scheme = "Trim Yer Beard (terminal.sexy)",
  --color_scheme = "VWbug (terminal.sexy)",
  --color_scheme = "Violet Dark",
  --color_scheme = "Visibone Alt. 2 (terminal.sexy)",
  --color_scheme = "Wombat",
  --color_scheme = "Woodland (base64)",
  --color_scheme = "X::Erosion (terminal.sexy)",
  --color_scheme = "darkmoss (base64)",
  --color_scheme = "duskfox",
  --color_scheme = "flexoki-dark",
  --color_scheme = "jmbi (terminal.sexy)",
  --color_scheme = "kanagawabones",
  --color_scheme = "nordfox",
  --color_scheme = "rose-pine",
  --color_scheme = "s3r0 modified (terminal.sexy)",
  --color_scheme = "zenwritten_dark",
  --color_scheme = "Decaf (base16)",
  --color_scheme = "Default (dark) (terminal.sexy)",
  --color_scheme = "Gogh (Gogh)",
  --color_scheme = "Gruvbox Dark (Gogh)",
  --color_scheme = "Gruvbox Material (Gogh)",
  --color_scheme = "Gruvbox dark, pale (base16)",
  --color_scheme = "Hardcore",
  --color_scheme = "Hybrid",
  --color_scheme = "Japanesque (Gogh)",
  --color_scheme = "Ocean Dark (Gogh)",
  --color_scheme = "OneDark (Gogh)",
  --color_scheme = "Operator Mono Dark",
  --color_scheme = "Red Planet",
  --color_scheme = "Rippedcasts",
  --color_scheme = "Srcery (Gogh)",

  -- Font
  font = wezterm.font('Hack Nerd Font'),
  font_size = 11,

  -- Transparency
  window_background_opacity = 1.0,
  text_background_opacity = 1.0,

  -- Windows 11 blur
  win32_system_backdrop = 'Acrylic',

  -- Window
  initial_cols = 140,
  initial_rows = 40,

  window_padding = {
    left = 15,
    right = 15,
    top = 15,
    bottom = 15,
  },

  -- Hide title bar
  window_decorations = "RESIZE",

  -- Cursor
  default_cursor_style = 'BlinkingBar',

  -- Scrollback
  scrollback_lines = 10000,

  -- Better rendering
  front_end = 'WebGpu',

  -- Disable annoying bell
  audible_bell = 'Disabled',

  -- Keys
  keys = {
    {
      key = 'Space',
      mods = 'CTRL',
      action = wezterm.action.SendString('\\0'),
    },
  },
}
"""

path = Path("/mnt/data/wezterm-original.lua")
path.write_text(config, encoding="utf-8")
path.as_posix()
```