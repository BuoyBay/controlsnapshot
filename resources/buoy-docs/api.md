---
title: "API and ownership"
summary: "Builtin native control snapshots for visual editors."
category: "Reference"
order: 1
---

# ControlSnapshot 0.1.0

Native builtin control images for visual editors. Import
`controlsnapshot.ControlSnapshot`. This is a design-time descriptor renderer;
Buoy's runtime still has no general `UIControl.RenderTo` operation.

`ControlSnapshotDescriptor` carries Kind, Title, Text, Placeholder, State,
Minimum, Maximum, Value, Flag, Elements, Items, RowHeight, Enabled, FontName and
FontSize. It contains literal values only. No project constructor, source
expression, event handler or custom painter is executed.

- `Render(spec, width, height, scale, dark) As Picture`: detached image or Nil.
- `TryRender(...) As ControlSnapshotOutcome`: Image, Status and Message.
  Status is rendered, unsupported, invalid-request or failed. A missing native
  facility or failed capture does not return a successful empty Picture.
- `CacheKey(spec, width, height, dark, scale)`: length-delimited literal visual
  identity, excluding canvas position and selection. Append `AppearanceKey()`
  for the current native theme/font/accent context. Keys are process-local;
  do not persist across package/runtime/OS upgrades. Compare keys case-sensitively.
- `BackingScale()`: main display density on macOS; 1 elsewhere. Hosts can supply
  their own display density. Dimensions are logical; pixels use Ceiling(size*scale).

Known kinds: button, checkbox, radio, radiogroup, label, textfield, search,
combo, popup, date, slider, progress, stepper, segments, tabs, list, textarea.
AppKit and GTK also provide disclosure. Windows reports disclosure unsupported.
Unknown/custom controls return unsupported. Composite controls use representative
content (including fixed 2026-01-01 09:41 UTC dates); these are not live runtime
control instances. Container children remain separate editor objects.

Call synchronously on the UI thread, serialized, with native application
initialization and an available display. There is no background worker,
application event-loop pumping or asynchronous cancellation. AppKit uses
cacheDisplay into an initialized bitmap under a never-shown NSWindow and restores
current appearance. Win32 uses WM_PRINT into a top-down DIB under a never-shown
parent, flushes GDI before CPU reads and releases GDI/window resources in Finally.
The progress bar commits its value by increasing one unit then decreasing before
restoring its maximum, avoiding the native increase animation without a sleep.
GTK4 needs a mapped transparent undecorated host and a working GSK renderer;
it snapshots the child into a texture and destroys the host before returning.
Snapshot descendants cannot take keyboard focus; entry/search selections are
cleared after mapping. The GTK progress trough uses its natural height centered
in the requested rectangle.
GTK cannot promise an entirely unmapped/never-shown host or focus isolation on
every compositor. This dependency is part of the current workaround.

AppKit/GTK images can have transparent pixels; composite onto the intended canvas
background before comparing. Windows images include an opaque white or #2a2a2a
background. Snapshots clip at the requested rectangle; overflow, focus/shadow
metadata, arbitrary control subtrees, special GPU/web surfaces and comprehensive
visual invalidation are not provided. Enabled/font/theme behavior follows each
adapter's native mappings, not a cross-platform fidelity guarantee. Windows dark
checkbox/radio snapshots use classic native glyphs with dark backgrounds and
readable captions (including radio-group children). List bodies are dark, but
headers remain light on the tested Windows VM. DateTimePicker and TabView remain
light; ComboBox/pop-up surfaces can also remain light. TryRender.Message reports
these full/partial dark fallbacks even when Status is rendered. No synthetic dark
recoloring or full runtime theme parity is claimed.

Before the rendering correction, GTK text/search snapshots selected the text,
and the Windows dark checkbox was approximately 99% white with an invisible
caption; radio images were also white and date/list/tab surfaces remained light.
Those were visible rendering defects, beyond merely poor caption contrast.

The caller owns returned Pictures, cache limits, scheduling and invalidation.
No package cache retains native hosts. LayoutCanvas keeps its existing 256-entry,
16,777,216-pixel cache and scale rounding. Individual renders reject nonpositive
sizes/scales, scales above 8, dimensions above 32767 pixels and images above
16,777,216 pixels (64 MiB decoded RGBA, excluding transient native/PNG allocations).
Callers must clear caches for visual changes not represented by AppearanceKey,
such as font registration, runtime revisions or backdrop-dependent properties.

See tests/controlsnapshot_test.bui and BUI-709's reference-evidence for semantic,
native execution and visual inspection scope. Cross-target checks prove parsing
and typing only; native tests cover the architecture actually executing them.
