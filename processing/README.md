### Processing

Helper scripts for generating TX phases and visualizing measurements.

#### Phase generation
- `compute-tx-weights.py`: compute Friis/MRT weights and phases, and write:
  - `../client/tx-phases-friis.yml`
  - `../client/tx-weights-friis.yml`
  Note: this script downloads the tile antenna locations from GitHub.

#### Plotting
- `plot-values-positions-2d.py`: per-folder matplotlib heatmap; set `FOLDER` inside the script.
- `plot_all_folders_heatmap.py`: aggregates folders under `../data`, saves `heatmap.png` into each folder, and shows the plot. Example:
  ```
  python plot_all_folders_heatmap.py --plot-all --plot-movement
  ```

#### Reference geometry
- `room.xml`: room geometry reference used by some tooling.

