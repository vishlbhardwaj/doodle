# Registered doodles — a doodle that sits exactly on a render

> Part of the `doodle` skill. Read when a doodle has to LINE UP with an existing image: a
> doodle → render wipe or crossfade in a film, a "sketch to final" reveal, an overlay.

A transition between two drawings of one object only reads as ONE object changing material if
the two silhouettes sit on top of each other. A doodle drawn from memory never does.

## The recipe

1. **Silhouette: measured, not drawn.** Trace it from the render's alpha channel (marching /
   Moore-neighbour boundary, Ramer–Douglas–Peucker, then Chaikin). Trace EVERY blob over ~1% of
   the area: a two-object icon (a doorbell beside a lock) has two silhouettes.
   - **Resample long edges BEFORE smoothing.** Chaikin on a 4-point rectangle makes a lens: the
     doorbell came back as a pointed ellipse. With edges cut to ~3 units only true corners round.
2. **Interior: authored**, on a 0..100 grid read off the render (draw the grid over the image at
   ~5 px/unit and read coordinates). Seams, faces, buttons, grain, the dark materials as hatch.
   - **Colour-cluster tracing of the interior does not work** (tried: k-means on colour, chroma
     weighted over lightness, compactness filter). It follows shading bands and highlights, not
     construction: half faces, broken shirts, wood grain as contour. A darkness threshold for
     "black materials" fails the same way on shaded black plastic.
3. **Frame = the render's own square (0..100), fixed.** Never auto-fit to the drawing's bbox.
4. **Undo the class fit on export.** Classes fit the art to a fraction of the frame (`object`:
   .84), centred. Crop `(1 − fit)/2` off every side of the PNG and the drawing and render share
   pixel coordinates again. Check it: multiply the line PNG over the render — the outlines must
   sit on the edges.
   - **Measure the crop; don't assume it.** On a tall page the panel came out at 0.819 of the tile,
     not 0.84, which left a white strip at the top. Take the bbox of the non-white pixels in the
     colour shot (the clipped panel) and crop every export to that same box: line, colour and each
     frame of the live replay.
   - **The `?cal=1` switch paints the frame panel pure green and draws NOTHING else** (return right
     after `S.panel`). If the drawing stays on top, green shows only through the sky and the "frame"
     bbox comes back at the wrong aspect. Check that the box's aspect equals `ASPECT` before cropping.
   - **Periodic parts fool the shift search** (lattice bracing, cobble courses, a grille): its best
     shift lands one period off or on the search bound with a ~2% gain. Place those members from a
     darkness profile of the photo, and judge registration on the non-periodic edges only.
   - **A see-through lattice (45° strips) is two families of lines, u = x + y and u = x − y.** Fit each
     family's pitch and phase by projecting the photo's darkness onto u, PER WINDOW the lattice shows
     through (between shelves, inside a box): one global fit drifted a third of a period between windows.
     Then each hole is a diamond centred between strip lines (half-diagonal = (pitch − strip)/2), clipped
     to its window and outlined: the outlines ARE the line drawing. The same lattice seen edge-on (a shelf
     top from above, a ceiling from below, a side panel) is rows of flat diamonds, not a foreshortened grid.
   - **A narrow window round a vertical edge cannot measure dy** (and vice versa): a pot-side window
     reported a 0.7-unit dy that was the rim and the hatch; read the shift only along the edge's normal.
     A cluster of flower heads is periodic too (rings of petals): check each isolated head on its own
     window and the cluster by eye in the multiply overlay.
   - **Full-bleed lines run PAST the frame** (−1.5 … W + 1.5) and `clip` cuts them. Filtering a line's points to the frame instead drops its end points and the edge draws as a stub.
   - **A see-through part on the outline** (a wire loop, a ring pull, a handle with a gap) traces SOLID,
     because the silhouette's holes are filled. Splice the body's own edge in under it (cut the traced run
     inside a box, join its neighbours with a dense straight segment) and draw the part separately.
   - **Shoot at 2× the tile and downsample** the measured crop to the photo's pixels. A 1× crop is smaller
     than the photo, and upscaling it softens every line.
5. **Export ink on transparent** (alpha = 255 − luminance of the line render) so it sits on any
   plate or ground.

Thin parts (hose, shaft, wire, strap) are `strip(centreline, halfWidth)` round a Catmull-Rom
centreline: ONE strip, outlined, hatched if dark. Things lying on top are drawn AFTER what they
cover, so their fills cut its lines (the screwdriver over the wrench).

## Showing the process (photo → numbers → points → drawing)

A registered drawing can show its own construction, step by step, on the same frame as the photo:

- **Points view.** Before `Stage.build`, wrap `Hand.mark` and `Hand.region` to record every point list
  they are handed. Then clear the tile and draw those lists as dots joined by a faint polyline,
  using the tile's own transform. This is the geometry exactly as the pen receives it, before any
  wander or pressure. Dedupe lists by their first three points, because regions and their outlines
  repeat. Crop `(1 − fit)/2` off each side, as for any registered export, and the dots sit on the photo.
- **Recording the live replay.** `?draw=1` runs on real timers. `--virtual-time-budget` does not advance
  it in headless Chrome: four budgets from 1.5 s to 40 s all showed only the first pen dot. Record it
  over CDP in real time instead, with `Page.captureScreenshot` in a loop, and stop once the frame has
  been unchanged for a few seconds. Encode at the measured fps, then speed it up (a 100 s replay plays
  well at 12–17×).
- **Points as data** (for a motion film): add `?json=1` to the points view to dump the lists into a `<pre>`, read with `--dump-dom`. Use tile-space coordinates, calibrated by the frame panel (the first list), never `getScreenCTM`.
- **Check registration per part, not only globally.** Search the shift that maximises line-ink × photo-gradient in a window round each part (cup, saucer, legs, panels); a whole-picture search reports 0 while one small part sits half a unit off.
- **Many pale flower heads on a pale ground**: no brightness mask separates them, but a disc does. Take each disc's centre and semi-axes from a chroma blob (2σ along its principal axes = the semi-axes), read the petal-tip ellipse per head off a grid, and build the petals on a unit circle in the head's plane mapped through that ellipse.
- **Correct a misplaced part by moving the part, not its numbers.** When the per-part search reports a consistent offset (grid reading errors are often a uniform ~0.5 unit on a part), draw that part through a shifted stage — `{ ...S, F, L, hatch }` with every point moved by (dx, dy) — and re-run the search until it reports ≤ 0.1 unit.
- **The search is blind on a hatched ground.** A part lying on a field you ruled (a table top, a wall) can report a large gain at a wrong shift, because the ruling matches photo texture. Judge those parts by a zoomed overlay (ink in a strong colour over the photo), not by the number.
- **Closed outlines in a tracer:** RDP on a loop whose first and last points are the same collapses it to two points. Split at the point farthest from the start and simplify each half.
- **Check on a contours-only render, with a small search radius.** Hatch, a frame band's double edge (two parallel edges under ~1 unit apart) and strong edges just outside the window all pull the shift search to its bound. Give the drawing a `?nohatch=1` switch (`S.hatch = () => {}`), search ±0.5 unit, and keep windows off double edges; confirm any reported shift on a zoomed overlay.
- **A wheel seen off-axis is not concentric with its hub.** Away from the camera's centre line the tread shows on the side facing the camera, so the tyre silhouette shifts ~0.3 unit off the hubcap centre. Take the tyre from luminance profiles (left/right edges on a row, the foot on a column) and the hubcap separately; a hub-centred tyre put the rear wheel 0.5 unit off.
- **A sparse mass (an open crown) over a same-hue ground (brick):** separate by the chroma RATIO `(r − b)/r` (leaves ≳ 0.5, brick ≲ 0.45), not `r − b`; then blur the BINARY mask (σ ~10 px, threshold 0.3) before tracing, so leaf sprays become one envelope instead of shreds. Blurring the photo instead still traces holes.
- **Leaves with slits (monstera): trace an OPENED mask, never a closed one.** Closing fuses the slit mouths and the slits come back as enclosed black dashes. Open (erode, then dilate; this also drops thin stems, which are authored as strips), erode a further ~2 px so slits stay open under the outline, and drop traced holes under ~0.35 unit². To hatch only part of a traced shape, wrap `S.L` while `S.hatch` runs and clip each stroke to the silhouette minus its holes.
- **Spoked wheels and parts in front of textured ground** (a tyre over a trunk, a rack over bark) pull the shift search toward the spokes or bark: judge them on narrow windows round their outer edges and confirm with luminance profiles.
