# Unity TextureGen

**Generate and validate lightweight Unity textures for standalone Meta Quest.**

Ask your coding agent for a texture in natural language. `unity-texturegen` uses the image generator built into the agent, checks tileable textures for visible seams, and reports the Unity URP import settings to use.

This is an **Agent Skill**, not a Unity package or editor extension. It generates PNG assets and guidance; it does not add anything to Unity Package Manager or import files into your project automatically.

You need a compatible coding agent with image generation, Python 3, and a Unity URP project.

<details>
<summary>中文简介</summary>

这是一个面向 Unity URP 与 Meta Quest 独立端的 GPT Image 贴图生成 Skill。你可以直接用自然语言生成无缝建筑贴图、灯光 Cookie、发光遮罩、雾和粒子纹理；Skill 会检查平铺接缝并给出 Unity 导入设置。它不是 Unity 插件，也不会自动把文件导入项目。

</details>

## Example: prompt to verified texture

```text
给 Quest 独立端 URP 生成一张 1024×1024 的无缝浅灰色清水混凝土外墙贴图，克制、现代、不要明显污渍。
```

| Generated base-color texture | 3×3 repeat QA |
|---|---|
| <img src="examples/tx_concrete_wall_light_gray_a_albedo_1024.png" width="420" alt="Generated light-gray architectural concrete texture"> | <img src="examples/qa_concrete_wall_light_gray_3x3.png" width="420" alt="Three by three repeat preview used to check the concrete texture for seams"> |

Test result:

- `1024×1024` RGB PNG
- Tiling status: `verified` by visual 3×3 repeat inspection
- Unity: sRGB On, Wrap Repeat, Mip Maps On, Bilinear
- Android starting point: ASTC 6×6 texture compression

This example was forward-tested with Codex and its built-in image generator. The texture has not yet been profiled inside a shipping Quest build.

## Compatibility

| Component | Status |
|---|---|
| Agent Skills format | Validated |
| Codex + installed `$imagegen` | Forward-tested |
| Python | Standard library only; Python 3 required for tiling preview |
| Unity 6 / URP | Target workflow; import recommendations included |
| Meta Quest standalone | Conservative defaults; real device profiling still required |
| Other Agent Skills clients | Format-compatible; execution still requires a native raster image tool, Python 3, and local preview rendering |

## Install

### Codex — recommended

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo okokelly/unity-texturegen \
  --path skills/unity-texturegen
```

Start a new Codex task after installation, then try:

```text
Use $unity-texturegen to make a seamless concrete wall texture for Quest URP.
```

<details>
<summary>Install with the cross-agent Skills CLI</summary>

Requires Node.js/npm:

```bash
npx skills add https://github.com/okokelly/unity-texturegen \
  --skill unity-texturegen \
  --agent codex \
  --global \
  --yes
```

</details>

<details>
<summary>Manual installation for Codex, Claude Code, or another agent</summary>

```bash
git clone https://github.com/okokelly/unity-texturegen.git

# Codex
cp -R unity-texturegen/skills/unity-texturegen ~/.codex/skills/unity-texturegen

# Claude Code
cp -R unity-texturegen/skills/unity-texturegen ~/.claude/skills/unity-texturegen

# Example shared Agent Skills directory; confirm your client's documented discovery path
cp -R unity-texturegen/skills/unity-texturegen ~/.agents/skills/unity-texturegen
```

Restart or reload the relevant coding-agent session so it can discover the new skill.

</details>

Nothing is installed into Unity itself.

## Use

The generated files are saved where you request. If you omit a destination, the agent chooses a stable path in the current workspace and reports the absolute path.

Try one of these:

```text
Use $unity-texturegen to generate a seamless modern concrete wall texture and check the seams.
```

```text
Create a 512×512 grayscale Light Cookie—a texture that shapes a Unity spotlight—with soft window-blind shadows.
```

```text
Generate a black-background additive ground-fog sprite for a standalone Quest scene.
```

## Capabilities

| Asset or task | Output |
|---|---|
| Seamless texture / tileable material | Architectural base-color (albedo) candidate plus 3×3 repeat QA |
| Unity Light Cookie | Grayscale image that shapes a spot or directional light |
| Emission mask | Grayscale control texture for tinted glowing materials |
| Fog, smoke, dust, particles | Additive sprites and reusable noise textures |
| Quest texture optimization | Conservative size, mipmap, color-space, and Android ASTC guidance |
| Unity texture delivery | Predictable filename, shader role, dimensions, and import checklist |

The focus is architectural environments and lighting effects—not character modeling, mesh generation, or UV unwrapping.

## How the bounded QA loop works

```text
classify → prompt → generate → inspect → 3×3 tile QA
                                      ↓ seam found
                                  repair once → recheck → deliver
```

The skill allows one repair attempt. Repeated generative edits often introduce scale changes, pattern drift, or new artifacts.

For every asset, the agent reports:

- output image and file path;
- exact dimensions and intended Unity shader role;
- `verified`, `failed`, or `unverified` tiling status when relevant;
- only the import settings that differ from defaults;
- a warning for transparency, high resolution, or another Quest-sensitive choice.

Example filename:

```text
tx_concrete_wall_a_albedo_1024.png
```

## Security and privacy

The skill prefers the image generator already built into the coding agent. It does not ask the user to paste an image API key into chat and does not configure third-party image-generation endpoints.

Prompts, reference images, and generated assets are still handled by the host agent's image-generation service. Review that provider's data policy for your project.

## Limitations

- Generative models cannot guarantee mathematical seamlessness; the skill performs a visual repeat check and reports uncertainty honestly.
- Independently generated normal, roughness, and ambient-occlusion maps may not align pixel-for-pixel, so a full PBR map set is not promised by default.
- It does not create, unwrap, or modify meshes.
- It does not generate point-light Cookie cubemaps.
- Unity settings are conservative starting points, not a substitute for profiling a real Quest build.
- Agents without an image-generation tool can read the skill but cannot generate assets.

## Repository layout

```text
skills/unity-texturegen/
├── SKILL.md
├── agents/
│   └── openai.yaml
├── references/
│   ├── prompt-recipes.md
│   └── quest-texture-specs.md
└── scripts/
    └── make_tile_preview.py
```

Test images used in this README live under `examples/`; they are not installed as part of the skill.

- [`SKILL.md`](skills/unity-texturegen/SKILL.md) contains the compact orchestration workflow.
- [`prompt-recipes.md`](skills/unity-texturegen/references/prompt-recipes.md) contains task-specific GPT Image prompts.
- [`quest-texture-specs.md`](skills/unity-texturegen/references/quest-texture-specs.md) contains Unity/Quest delivery defaults.
- [`make_tile_preview.py`](skills/unity-texturegen/scripts/make_tile_preview.py) creates a dependency-free repeat preview for seam inspection.

## Agent discovery

This repository is relevant when a user or agent searches for:

- Agent Skill, Codex skill, AI texture generator, OpenAI image texture generation, GPT Image game assets
- Unity texture generation, seamless texture, tileable material, architectural texture, PBR-friendly base-color texture
- Unity URP, Meta Quest, Quest 3, standalone VR, mobile VR textures, VR texture optimization
- Light Cookie, cookie texture, emission mask, fog texture, fog sprite, smoke texture, particle texture
- Unity贴图生成器, Unity纹理生成, 无缝贴图, 无缝纹理, 可平铺贴图, 循环纹理
- 建筑贴图, 材质贴图, 灯光Cookie, 光照Cookie, 发光遮罩, 自发光贴图
- 雾贴图, 雾效贴图, 烟雾贴图, 粒子贴图, Meta Quest贴图, 移动VR贴图, GPT图像生成

## Contributing

Issues and pull requests are welcome for new Unity texture classes, better seam QA, prompt improvements, and measured Quest recommendations. Keep the core workflow small, readable, honest about validation, and dependency-light.

## License

[MIT](LICENSE)
