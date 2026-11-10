---
name: render-glb-headless
description: Render a GLB or glTF avatar or prop to a PNG, animated PNG or terminal image with no GPU, WebGL or browser using @three-ws/render. Use to preview a model, make a headshot or thumbnail, or capture an animation clip.
---

# Render a GLB without a GPU

`@three-ws/render` is a pure JavaScript software rasterizer for glTF/GLB: skinned avatars, meshopt-compressed and quantized models, mip-mapped textures, alpha-masked hair, a contact shadow. `three` is the only required dependency; `sharp` is optional and only decodes JPEG textures (without it, JPEG-textured models fall back to material colours).

```bash
npm install @three-ws/render
```

## CLI

```bash
npx -p @three-ws/render three-ws-tty <model.glb | url> [options]
```

Options: `--out <file>` (PNG; animated PNG when `--frames` > 1), `--width`, `--height`, `--focus full|bust|head`, `--preset studio|terminal|daylight`, `--clip <name|idx>`, `--animation <file>` (GLB or clip JSON retargeted onto the rig), `--frames <n>`, `--fps <n>` (default 20), `--spin <deg>`, `--yaw`, `--pitch`, `--bg <hex|transparent>`, `--loop`, `--blocks`, `--clips` (list clips and exit).

```bash
three-ws-tty avatar.glb --focus head --width 512 --out headshot.png
three-ws-tty avatar.glb --animation wave.glb --clip wave --frames 24 --bg transparent --out wave.png
three-ws-tty prop.glb --spin 360 --frames 36 --out turntable.png
three-ws-tty model.glb --clips
```

## Library

```js
import { renderAvatar, encodePng, encodeApng } from '@three-ws/render';
import { writeFile } from 'node:fs/promises';

const { frames } = await renderAvatar('avatar.glb', { width: 1200, height: 630, focus: 'full', background: { inner: '#1b2440', outer: '#05060c' } });
await writeFile('card.png', encodePng(frames[0]));
await writeFile('walk.png', encodeApng((await renderAvatar('avatar.glb', { animation: 'walk.glb', clip: 'walk', frames: 24, fps: 20, background: 'transparent' })).frames, { fps: 20 }));
```

`source` is a URL, file path, `ArrayBuffer` or typed array. Frames are `{ width, height, data }` with straight-alpha RGBA. Other exports: `AvatarModel.load`, `renderFrame`, `renderFrames`, `toHalfBlocks`, `toKitty`, `toITerm`, `detectTerminal`, `normalizeBoneName`, `retargetClip`. Options include `supersample` (1-4), `fov`, `shadow`, `maxBytes` (64 MB), `maxTextureSize` (1024).

## Limits to know

- Draco geometry is refused with an error; re-export with `gltfpack -cc`.
- KTX2/Basis textures are skipped (base colour used). Morph targets are not evaluated. Shading is Blinn-Phong, not full PBR.
- A clip whose rig shares fewer than four bones with the target is refused.
- Use APNG, not GIF, for animation output.
