# Geometry-Based RF Wireless Power Transfer (WPTCE'26)

Python tooling to coordinate a geometry-aware wireless power transfer experiment over distributed tiles (Raspberry Pi + USRP B210), using a ZMQ control plane and Ansible playbooks from the `tile-management` repo.

## Repository layout
- `experiment-settings.yaml`: central experiment config (tiles, RF params, server ports, client script + args)
- `server/`: control-node utilities (provision/update tiles, start/stop clients, run the ZMQ coordinator)
- `client/`: tile-side scripts and calibration/phase YAMLs
- `processing/`: TX phase generation and plotting scripts
- `lib/`: shared helpers
- `data/`: recorded measurements and generated heatmaps

## Prerequisites (control node)
- Python 3 and Ansible
- SSH access to the tiles
- `tile-management` checked out at `~/tile-management` (or let `server/setup-server.sh` clone/update it)
- Tiles present in `~/tile-management/inventory/hosts.yaml`

## Typical workflow
1) Create the control-node virtualenv and pull/update `tile-management`
```bash
cd server
./setup-server.sh
source bin/activate
```

2) Generate/select TX phases
```bash
python processing/compute-tx-weights.py
```
This writes `client/tx-phases-friis.yml` (and `client/tx-weights-friis.yml`). Note: it downloads the tile antenna locations from GitHub.

3) Configure `experiment-settings.yaml`
- Set `server.host` and the `tiles` list/group
- Pick a `client_script_name` (e.g. `run_gbwpt_phases.py`, `run_gbwpt_random_phases.py`, `run_reciprocity.py`)
- Adjust `client_script_args` (including `--tx-phase-file` when applicable)

4) Prepare tiles (apt, repos, UHD) and deploy the experiment repo/settings
```bash
python server/setup-clients.py --ansible-output
python server/update-experiment.py --ansible-output
```

5) Start the clients and run the ZMQ server
```bash
python server/run-clients.py --start
python server/run_server.py
```
Clients wait for sync, transmit for the configured duration, then reply with `tx-done`.

## Recording & plotting
- Energy-profiler recorder (update `FOLDER` in `server/record/record-meas-energy-profiler.py`):
```bash
python server/record/record-meas-energy-profiler.py
```
Autosaves to `data/<FOLDER>/<timestamp>_{positions,values}.npy`.

- Plot heatmaps:
```bash
python processing/plot_all_folders_heatmap.py --plot-all
```

## Data

The `data/` folder contains recorded measurements and generated plots. Most runs include:
- `<timestamp>_positions.npy` and `<timestamp>_values.npy`
- `heatmap.png`
- optionally `heatmap_vs_RANDOM_dB.png`

Regenerate heatmaps:
```bash
python processing/plot_all_folders_heatmap.py --plot-all
```

| Folder | Heatmap | Baseline vs RANDOM (dB) |
| --- | --- | --- |
| FRIIS-0 | ![FRIIS-0](data/FRIIS-0/heatmap.png) | ![FRIIS-0 vs RANDOM](data/FRIIS-0/heatmap_vs_RANDOM_dB.png) |
| FRIIS-ABS-0 | ![FRIIS-ABS-0](data/FRIIS-ABS-0/heatmap.png) | N/A |
| FRIIS-ABS-REFL-0 | ![FRIIS-ABS-REFL-0](data/FRIIS-ABS-REFL-0/heatmap.png) | N/A |
| RANDOM | ![RANDOM](data/RANDOM/heatmap.png) | ![RANDOM vs RANDOM](data/RANDOM/heatmap_vs_RANDOM_dB.png) |
| RANDOM-1 | ![RANDOM-1](data/RANDOM-1/heatmap.png) | ![RANDOM-1 vs RANDOM](data/RANDOM-1/heatmap_vs_RANDOM_dB.png) |
| RANDOM-2 | ![RANDOM-2](data/RANDOM-2/heatmap.png) | ![RANDOM-2 vs RANDOM](data/RANDOM-2/heatmap_vs_RANDOM_dB.png) |
| RANDOM-ABS-REFL-0 | ![RANDOM-ABS-REFL-0](data/RANDOM-ABS-REFL-0/heatmap.png) | N/A |
| RECI-0 | ![RECI-0](data/RECI-0/heatmap.png) | ![RECI-0 vs RANDOM](data/RECI-0/heatmap_vs_RANDOM_dB.png) |
| RECI-1 | ![RECI-1](data/RECI-1/heatmap.png) | ![RECI-1 vs RANDOM](data/RECI-1/heatmap_vs_RANDOM_dB.png) |
| RECI-3 | ![RECI-3](data/RECI-3/heatmap.png) | N/A |
| RECI-ABS-0 | ![RECI-ABS-0](data/RECI-ABS-0/heatmap.png) | N/A |
| RECI-ABS-REFL-0 | ![RECI-ABS-REFL-0](data/RECI-ABS-REFL-0/heatmap.png) | N/A |
| RECI-merged | ![RECI-merged](data/RECI-merged/heatmap.png) | ![RECI-merged vs RANDOM](data/RECI-merged/heatmap_vs_RANDOM_dB.png) |

## Maintenance utilities
- `server/cleanup-clients.py`, `server/reboot-clients.py`
- `client/ref-RF-cable.yml` and `client/tx-phases-*.yml` contain calibration and phase data

## License
MIT (see `LICENSE`).
