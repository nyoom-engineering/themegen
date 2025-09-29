# themegen

All-in-one themeing utility 

Generate themes from industry-standard templates

E.g. VSCode -> ST, Zed, Vim/Neovim, Emacs, XCode
base16 -> wallpaper
so on so forth

- write themes in standard formats (e.g. vscode .json), either using text editor or gui
- make workflow (either in GUI using nodes (export as lisp) or in lisp)
- run workflow to generate zip containing themes, adjusted wallpapers, etc. 

example workflows 

- in: vsc.toml: toml->json, out vsc.json
- in: vsc.toml: toml->compatibility-widget->(twofiles)oled-widget->(fourfiles)json, out vsc.json
- toml->data->map-vscode-to-zed->json, out Zed.json
- base16+wallpaper->lutgen->png, out wallpaper.png
- vsc.json->vscode-web-screencap-widget->png, out preview.png

nodes for workflows are written in lua/fennel, using mlua to bind rust functions vice versa

serializing files, color transformations, operations etc. 

and then within a configuration language (lua/fennel, steel, undecided) we then define a template/schema to translate the VSCode theme

in rust we define simple primatives w/ bindings

```rust
let format_rgba = |(r, g, b, a): (f32, f32, f32, f32)| {
    let q = |x: f32| (x * 1_000_000.0).round() / 1_000_000.0;
    format!("{} {} {} {}", q(r), q(g), q(b), q(a))
};
```

and then in fennel lua bindings

can be as simple as conversion, or can just map, or can handle logic

assume `ffi_map_to_xccolors` etc. are defined in rust

```cl
;; node name ->xccolors
;; assumes node input: map, lua table, automatically binded as lua table
;; output json
(defnode ->xccolors [table]
  (ffi_map_to_xccolors table))

;; node name ->xcode
;; assumes node input: vscmap, mapped to lua table automatically
;; output xccolor
(defnode ->xcode [vscmap]
  ;; table to hold colors
  (local xcode {:name "" :type "" :tokencolors []})
  
  ;; map
  (local xcode.name vscmap.name)
  (local xcode.type vscmap.themetype)
  (local xcode.tokencolors.xcode.syntax.string (ffi_format_rgba vscmap.string))
  
  ;; export to xccolors
  (->xccolors xcode))
```