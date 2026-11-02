# Native control snapshots for visual design tools

Native builtin control images for visual editors. Import
`controlsnapshot.ControlSnapshot`. This is a design-time descriptor renderer;
the runtime still has no general `UIControl.RenderTo` operation.

`ControlSnapshotDescriptor` carries Kind, Title, Text, Placeholder, State,
Minimum, Maximum, Value, Flag, Elements, Items, RowHeight, Enabled, FontName and
FontSize. It contains literal values only. No project constructor, source
expression, event handler or custom painter is executed.

## Install

From a Buoy consumer project with BPM installed:

```sh
bpm add controlsnapshot --git https://github.com/BuoyBay/controlsnapshot.git --rev refs/heads/main
```

BPM records the resolved commit in its lockfile. Keep that lockfile with the
consumer project for reproducible installs.

## API entrypoints

- `ControlSnapshotDescriptor`
- `Render(spec, width, height, scale, dark) As Picture`
- `TryRender(...) As ControlSnapshotOutcome`
- `CacheKey(spec, width, height, dark, scale)`
- `AppearanceKey()`

## Build from source

With the native `build-packages` driver from [devtools](https://github.com/BuoyBay/devtools)
on your `PATH`, run from this checkout:

```sh
build-packages --root "$PWD" --compile-consumer controlsnapshot
```

The driver checks source targets and compiles an import consumer. See the
[API guide](docs/API.md) for behavior, limits and any host prerequisites.

## Local Buoy setup

On your own machine, install Buoy and run:

```sh
buoy ai init --both
```


## Contributing

[Contributing](CONTRIBUTING.md) · [License](LICENSE)
