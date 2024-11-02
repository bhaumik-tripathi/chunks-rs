# Chunks-rs
[![Crates.io](https://img.shields.io/crates/d/chunks-rs?style=flat-square&color=red)](https://crates.io/crates/chunks-rs)

A library that simplifies the process of making widgets for Wayland Compositors.

<div style="display: flex; align-items: center;">
    <img src="screenshots/scrot1.jpg" style="height: 150px;">
    <img src="screenshots/scrot3.jpg" style="height: 150px;">
    <img src="screenshots/scrot4.jpg" style="height: 150px;">
</div>

Chunks uses GTK4 and GTK4 Layer Shell at its core, and comes stock with a listener for the Hyprland IPC. This helps with changing Widget states when something changes, such as making the current window fullscreen.

## Usage

Make sure you have GTK4 and GTK4-Layer-Shell installed on your system.

> For more in depth examples, please refer to [example-chunks](https://github.com/drkrssll/example-chunks)

```toml
[dependencies]
chunks-rs = "0.5.2"
```

This will create a storage widget, similar to the one in the screenshot:
```rs
const STYLE: &str = "
window {
    background-color: transparent;
}

#storage {
    font-size: 34px;
    background-color: #000000;
    color: #FFFFFF;
}
";

fn main() {
    let factory = Factory::new("chunk.factory");

    let chunks = |factory: Application| {
        storage(&factory);

        load_css(STYLE);
    };

    factory.pollute(chunks);
}

fn storage(factory: &Application) {
    let tag = tag("storage");
    let margins = vec![(Edge::Top, 20), (Edge::Right, 160)];
    let anchors = EdgeConfig::TOP_RIGHT.to_vec();

    let storage_closure = || {
        let text = format!(
            "<span foreground='#FFFFFF'>{:.0}%</span>",
            Internal::get_storage(),
        );
        text
    };

    Internal::update_storage(&tag, storage_closure);

    Chunk::new(
        factory.clone(),
        "Storage".to_string(),
        tag,
        margins,
        anchors,
        Layer::Bottom,
    )
    .build();
}
```

## Slabs & Plates

Chunks recently introduced two new popup widget types: Slabs and Plates.

    Slabs: Display dynamically, triggered by changes in underlying text (e.g., volume detection).
    Plates: Display once at startup, then disappear after a set duration (e.g., welcome messages).

Both share similar implementations but differ in their display behavior.

> These widget types do not need a designated layer, as they are set to Overlay by default.
> Instead of a layer, enter the amount of seconds you would like the Popups to display for.
```rs
Slab::new(
    factory.clone(),
    "Volume".to_string(),
    tag,
    margins,
    anchors,
    2,
)
.build();
```

```rs
Plate::new(
    factory.clone(),
    "Greeter".to_string(),
    tag,
    margins,
    anchors,
    2,
)
.build();
```
