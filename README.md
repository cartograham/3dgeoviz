# Reality Capture Approaches

An interactive, single-file viewer that renders the same surface as a **point cloud**, a **mesh**, and a **Gaussian splat**, side by side, so you can see the practical differences between the approaches. It also includes a comparison matrix that adds **panoramic and street-level imagery**, and it can open your own 3D Gaussian Splatting `.ply` files.

It was built to explain reality modeling tradeoffs to transportation and GIS audiences: what each representation is good at, where it falls short, and how density, noise, and gaps change what you see.

## Quick start

1. Download `reality-capture-view-sandbox.html`.
2. Open it in a desktop browser. There is nothing to install or build.

Requirements:

- A browser with **WebGL2** (any recent Chrome, Edge, Firefox, or Safari on desktop).
- Internet access on first load. The page pulls [three.js r128](https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js) from cdnjs. To run offline, download that file and point the `<script src>` tag at your local copy.

## What's in the app

### Views

| View | What it shows |
|---|---|
| **Compare** (default) | Point cloud, mesh, and Gaussian splat in three columns, sharing one camera |
| **Points / Mesh / Splats** | One approach full width |
| **Comparison matrix** | A table of 12 factors across imagery, point cloud, mesh, and splat |

Each column has **Strengths** and **Weaknesses** toggles that open short notes with icons. The matrix covers measurement accuracy, visual realism, free-viewpoint navigation, thin features, gaps, file size, streaming, rendering demand, production effort, editing, GIS/CAD support, and typical use.

### Demo scenes

- **Pineapple:** a procedural fruit with a crown of thin leaves. Good for showing how thin features survive (or don't) in each representation.
- **Roadside:** a road with lane paint, guardrail, warning sign, light pole, and cones.

Both are generated in code. Points and splats are sampled from the reference surface.

### Controls

The left panel is organized into cards. Each card appears only when it applies to the visible views.

| Card | Control | What it does |
|---|---|---|
| **Point Cloud** | Point size | Dot size in pixels |
| | Shrink points as density rises | Keeps dots distinct as density grows (never grows them) |
| | Capture gaps | Drops whole patches of points, like missing returns on occluded or shiny surfaces |
| **Mesh** | Link mesh detail to capture density | Ties mesh resolution to the density slider (on by default) |
| | Mesh detail | Mesh resolution when unlinked |
| | Mesh display | Textured, textured with edges, clay, or wireframe |
| **Gaussian Splat** | Splat size | Scales every splat |
| | Show splat outlines | Draws the 1-sigma ellipse of each splat |
| **Capture Details** | Capture density | Number of samples, up to 100,000 in the demo scenes |
| | Capture noise | Jitters points and splats to mimic sensor noise |
| **Other Options** | Leaf roughness | Pineapple only. Makes thin leaves sparse, ragged, and blobby |
| | Camera auto-rotate, Reset view | Camera helpers |
| | Open .ply file | Load your own splat file (see below) |

Mouse and keyboard:

- **Drag** to orbit, **scroll** to zoom, **right-drag** or **Shift-drag** to pan.
- On touch screens, one finger orbits and two fingers pinch to zoom.
- **Esc** closes the comparison matrix.

## Opening your own data

Use **Other Options → Open .ply file**, or drop a file onto the viewer.

Supported:

- **3D Gaussian Splatting `.ply`**, binary little-endian, with all vertex properties stored as `float32`.
- Required properties: `x, y, z, f_dc_0..2, opacity, scale_0..2, rot_0..3`. Extra properties (such as `f_rest_*`) are allowed and ignored.

What the loader does:

- Shows the file as a splat render next to a point cloud made from the same splat centers and colors. The mesh column is disabled because splat files have no faces.
- Shuffles the splats on load so the density slider shows an even random subset instead of cropping the scene. The default is 30% of the splats.
- Recenters the data, normalizes quaternions, and decodes the standard 3DGS encodings (log scale, logit opacity, spherical-harmonic base color).
- Picks an up axis from the smallest extent (usually Z for aerial captures). Use the **Up axis** selector if it's wrong.
- Re-sorts splats by depth in a background worker, so orbiting stays smooth with millions of splats.

Not supported yet: plain point-cloud PLYs (xyz + rgb), mesh PLYs, ASCII or big-endian PLYs, `.splat`, `.spz`, glTF splats, and LAS/LAZ.

Notes:

- Only the base color is used. View-dependent color (higher-order spherical harmonics) is not rendered for loaded files.
- Loaded files are held in a float texture on the GPU, roughly 48 bytes per splat. A 3.1M-splat file needs about 150 MB of GPU memory.
- If a Web Worker can't be created (for example, blocked by a page security policy), sorting falls back to the main thread and will stutter on large files.

## How it works

- **One reference surface.** Each demo scene is built as a triangle soup with baked lighting in the vertex colors.
- **Points and splats are sampled from it.** Samples are area-weighted, so each sample gets a position, normal, and color. Splats also get an oriented ellipsoid sized to the local sample spacing.
- **Splat rendering.** A vertex shader projects each 3D Gaussian's covariance to a 2D screen-space ellipse, and a fragment shader draws a Gaussian falloff. Splats are depth-sorted back to front and alpha-blended with no depth test.
- **Splat realism.** To stand in for what a trained splat would capture from photos, demo splats get extra texture detail, cast shadows on the ground, and view-dependent shine. Points and mesh keep their original colors.
- **Mesh.** Rebuilt at the chosen detail level. On the pineapple, leaf triangles are dropped and pulled toward a lumpy shell to mimic a mesh rebuilt from sparse data.
- **Point gaps.** A shader drops points where a smooth 3D noise field falls below a threshold, so gaps stay open as density rises.

## Limitations

- The demo scenes are procedural, and the splats are synthesized from a known surface. They are not trained from imagery, so they don't show real training artifacts such as floaters and haze.
- Size estimates in the stats are approximate: about 26 bytes per point (LAS), about 248 bytes per splat for a standard uncompressed 3DGS file, and the file's own stride for loaded files.
- The comparison matrix is a set of judgment calls, not benchmarks. Imagery accuracy depends on camera calibration and range. "Emerging" for splats reflects glTF and 3D Tiles support that was at release-candidate stage in the sources checked when the matrix was written.
- Imagery has no 3D scene in the app, so it appears only in the matrix.

## Ideas for next steps

- Load SPZ and glTF (`KHR_gaussian_splatting`) splat files.
- Load LAS/LAZ or plain-PLY point clouds into the point cloud view.
- Run the point cloud and splat from the same real capture to compare them on your own data.

## Credits

- [three.js](https://threejs.org) (MIT license) for rendering.
- The splat renderer follows the approach in Kerbl et al., "3D Gaussian Splatting for Real-Time Radiance Field Rendering" (2023).

## License

MIT. See [LICENSE](LICENSE).
