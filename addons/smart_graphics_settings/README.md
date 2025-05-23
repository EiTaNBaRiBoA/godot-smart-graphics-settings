# Smart Graphics Settings - Technical Documentation

This document provides detailed technical information about the Smart Graphics Settings addon for Godot 4.4, including its architecture, available settings, and API reference.

## Architecture

The addon consists of several key components:

1. **SmartGraphicsSettings** (`smart_graphics_settings.gd`): The main controller that manages the UI and serves as the entry point.
2. **AdaptiveGraphics** (`adaptive_graphics.gd`): Handles the automatic adjustment of graphics settings based on performance.
3. **GraphicsSettingsManager** (`graphics_settings_manager.gd`): Manages all available graphics settings and their application.
4. **FPSMonitor** (`fps_monitor.gd`): Monitors and analyzes framerate performance.
5. **AdaptiveGraphicsUI** (`adaptive_graphics_ui.gd`): Provides the user interface for adjusting settings.

## Installation and Setup

### Manual Installation

1. Copy the `addons/smart_graphics_settings` folder to your project's `addons` directory
2. Enable the plugin in Project Settings → Plugins

### Autoload Singleton (Recommended)

The plugin automatically registers a singleton named `SmartGraphicsSettings`. To use it:

```gdscript
SmartGraphicsSettings
```

### Manual Node Addition

You can also add the SmartGraphicsSettings node manually to any scene:

```gdscript
var settings_node: Node = preload("res://addons/smart_graphics_settings/smart_graphics_settings.gd").new()
add_child(settings_node)
```

## Configuration Options

### AdaptiveGraphics Properties

| Property | Type | Description |
|----------|------|-------------|
| `target_fps` | `int` | Target framerate to maintain (default: 60) |
| `fps_tolerance` | `int` | Acceptable FPS range around target (default: 5) |
| `adjustment_cooldown` | `float` | Seconds between adjustments (default: 3.0) |
| `measurement_period` | `float` | Seconds to measure FPS before adjusting (default: 2.0) |
| `enabled` | `bool` | Whether adaptive graphics is enabled (default: true) |
| `allow_quality_increase` | `bool` | Whether to increase quality when FPS is high (default: false) |
| `use_threading` | `bool` | Whether to use threading for adjustments (default: true) |
| `setting_change_delay` | `float` | Seconds between applying each setting change (default: 0.5) |
| `match_refresh_rate` | `bool` | Whether to match target FPS to display refresh rate (default: false) |

### Available Graphics Settings

The following settings can be adjusted automatically or manually:

#### Render Scale

- Controls the internal rendering resolution
- Priority: Highest (adjusted first)
- Values: 0.5, 0.6, 0.7, 0.75, 0.8, 0.9, 1.0

#### Anti-Aliasing

- Controls the anti-aliasing method
- Priority: High
- Values: Off, FXAA, MSAA 2x, MSAA 4x, MSAA 8x, TAA

#### Shadows

- Controls shadow quality and resolution
- Priority: Medium
- Values: Off, Low, Medium, High, Ultra

#### Reflections

- Controls screen-space reflections quality
- Priority: Low
- Values: Off, Low, Medium, High

#### Global Illumination

- Controls global illumination method and quality
- Priority: Lowest (adjusted last)
- Values: Off, Low, Medium, High

## API Reference

### SmartGraphicsSettings

```gdscript
# Signal emitted when the AdaptiveGraphics node has been initialized.
# Connect to this signal to safely access adaptive_graphics after startup.
signal initialized

# Toggle the settings UI visibility
func toggle_ui() -> void

# Apply a specific quality preset (1-5)
func apply_quality_preset(preset_index: int) -> void

# Save current settings to config file
func save_settings() -> void

# Load settings from config file
func load_settings() -> void

# Get the adaptive graphics controller
func get_adaptive_graphics() -> AdaptiveGraphics
```

### AdaptiveGraphics

```gdscript
# Signal emitted when settings change
signal changed_settings

# Start the adaptive graphics system
func start() -> void

# Stop the adaptive graphics system
func stop() -> void

# Force an immediate adjustment
func force_adjustment() -> void

# Get the current action being performed
func get_current_action() -> String

# Set a specific setting value
func set_setting(setting_name: String, value_index: int) -> void

# Get the current value of a setting
func get_setting(setting_name: String) -> int
```

### GraphicsSettingsManager

```gdscript
# Apply a specific quality preset (1-5)
func apply_preset(preset_index: int) -> void

# Get all available settings
func get_available_settings() -> Dictionary

# Get the current value of a setting
func get_setting_value(setting_name: String) -> Variant

# Set a specific setting value
func set_setting_value(setting_name: String, value_index: int) -> void

# Register a custom setting
func register_setting(setting_name: String, values: Array[Variant], 
      current_index: int, priority: SettingPriority, 
      type: SettingType) -> void
```

## Custom Settings

You can register custom settings to be managed by the system:

```gdscript
# Access SmartGraphicsSettings singleton directly
# No need for get_node() as it's registered as an autoload
var settings_manager = SmartGraphicsSettings.adaptive_graphics.settings_manager

# Register a custom setting
var values: Array[Variant] = [false, true]
settings_manager.register_setting(
 "my_custom_setting",
 values,
 1, # Current index (true)
 GraphicsSettingsManager.SettingPriority.POST_PROCESSING,
 GraphicsSettingsManager.SettingType.ENVIRONMENT
)

# Connect to the changed_settings signal
SmartGraphicsSettings.adaptive_graphics.changed_settings.connect(_on_settings_changed)

func _on_settings_changed() -> void:
 # Handle custom setting changes
 var custom_value = settings_manager.get_setting_value("my_custom_setting")
 print("Custom setting value: ", custom_value)
```

## Handling Initialization Timing

Due to Godot's node initialization order and the use of `call_deferred` within the addon, the `SmartGraphicsSettings.adaptive_graphics` instance might not be immediately available when accessed from another node's `_ready()` function. Trying to access it too early can result in getting a `null` value.

To safely access `adaptive_graphics` and its properties or methods at startup, you should connect to the `initialized` signal emitted by the `SmartGraphicsSettings` singleton. This signal is emitted once the internal `AdaptiveGraphics` node is ready.

**Example:**

```gdscript
# In a script that needs to customize settings (e.g., attached to your main scene root)

func _ready():
	# Check if the singleton and its property are already available (e.g., if the scene is reloaded)
	if SmartGraphicsSettings != null and SmartGraphicsSettings.adaptive_graphics != null:
		_on_smart_graphics_ready()
	elif SmartGraphicsSettings != null:
		# Connect to the signal if not yet ready
		SmartGraphicsSettings.initialized.connect(_on_smart_graphics_ready)
	else:
		push_error("SmartGraphicsSettings singleton not found!")

func _on_smart_graphics_ready():
	# Now it's safe to access adaptive_graphics
	if SmartGraphicsSettings.adaptive_graphics:
		print("SmartGraphicsSettings is ready! Customizing...")
		# Example: Set a custom target FPS
		SmartGraphicsSettings.set_target_fps(90) 
		# Apply other custom settings here using SmartGraphicsSettings wrapper functions
		# or directly via SmartGraphicsSettings.adaptive_graphics if needed.
	else:
		push_error("Failed to get adaptive_graphics even after initialized signal.")

```

## Troubleshooting

### Performance Issues

- If you experience stuttering during adjustments, try setting `use_threading` to false
- Increase `adjustment_cooldown` to reduce the frequency of adjustments

### UI Issues

- If the UI doesn't appear, check that the input action `toggle_graphics_settings` is properly registered
- The default key to toggle the UI is F7
- If you encounter errors related to `adaptive_graphics` being `null` at startup, ensure you are waiting for the `initialized` signal as described in the "Handling Initialization Timing" section.

### Custom Renderer Support

- For custom renderers, you may need to manually register compatible settings
- Use the `register_setting` method to add renderer-specific settings

## Best Practices

1. **Initialize Early**: Configure the addon's parameters (e.g., via Project Settings if applicable, or saved configuration files) before the game starts intensive rendering. For runtime access to the `adaptive_graphics` object from your scripts at startup, wait for the `SmartGraphicsSettings.initialized` signal.
2. **Default Presets**: Provide sensible default presets for different hardware capabilities
3. **Save User Preferences**: Always save and restore user settings between sessions
4. **Testing**: Test on various hardware configurations to ensure proper adaptation
5. **Feedback**: Provide visual feedback when settings are being adjusted automatically
