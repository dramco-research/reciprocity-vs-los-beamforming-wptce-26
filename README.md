# Experimental Evaluation of Geometry- and Reciprocity-Based Beamforming for RF WPT (WPTCE'26)

This repository contains the experiment control + analysis code used in the paper:
“Experimental Evaluation of Geometry and Reciprocity-Based Beamforming with Large Arrays for RF Wireless Power Transfer”.

We evaluate RF wireless power transfer (WPT) with a large distributed indoor transmit array (Techtile) at **920 MHz**, comparing:
- **GEO**: geometry-based phase-only precoding (free-space LoS/Friis model)
- **CSI**: channel/reciprocity-based phase-only precoding (pilot + loopback calibration)
- **RPS**: random-phase sweeping (non-coherent baseline)

Key paper result (beamforming gain relative to RPS):

| Scenario | GEO | CSI | CSI − GEO |
| --- | --- | --- | --- |
| LoS | 18.75 dB | 19.57 dB | 0.82 dB |
| oLoS + reflector/absorbers | 16.70 dB | 20.53 dB | 3.83 dB |

## Repository layout
- `experiment-settings.yaml`: central experiment config (tiles, RF params, server ports, client script + args)
- `client/`: tile-side Python (USRP B210) scripts + calibration and phase YAMLs
- `server/`: control-node tooling (Ansible orchestration, experiment service management)
- `server/record/`: ZMQ sync server and measurement helpers
- `processing/`: plotting scripts for measurement heatmaps
- `lib/`: shared helpers
- `data/`: recorded measurements and generated plots (heatmaps)

## Mapping paper terms to this repo

**Beamforming strategies**
- **GEO** (geometry-based): `processing/compute-tx-weights.py` -> `client/tx-phases-friis.yml` -> run clients with `client/run_gbwpt_phases.py`
- **CSI** (reciprocity-based): run clients with `client/run_reciprocity.py` (pilot + loopback + cable correction; see `client/ref-RF-cable.yml`)
- **RPS** (random-phase baseline): run clients with `client/run_gbwpt_random_phases.py`

`processing/compute-tx-weights.py` downloads the Techtile antenna locations from GitHub; if you are offline, replace that part with a local copy of the antenna location YAML.

**Data folders**
- `data/FRIIS-*` (GEO / Friis)
- `data/RECI-*` (CSI / reciprocity)
- `data/RANDOM*` (random baselines; multiple variants are included)

## Experimental setup (paper)
- Distributed indoor ceiling array: 8 m x 4 m (Techtile)
- Active TX antennas: 41 (one element was faulty during the experiments)
- Carrier frequency: 920 MHz
- Spatial evaluation: 1.25 m x 1.25 m 2D scan, harvested DC power measured with an RF-to-DC energy profiler

## Experimental scenarios (paper)
- **Scenario 1 (LoS)**: unobstructed propagation in the scan area.
- **Scenario 2 (oLoS + reflector/absorbers)**: direct path obstructed + strong specular reflection.

## Paper datasets (heatmaps)

| Scenario | RPS | GEO | CSI |
| --- | --- | --- | --- |
| LoS | ![RPS LoS](data/RANDOM-1/heatmap.png) | ![GEO LoS](data/FRIIS-0/heatmap.png) | ![CSI LoS](data/RECI-merged/heatmap.png) |
| oLoS + reflector/absorbers | ![RPS oLoS](data/RANDOM-ABS-REFL-0/heatmap.png) | ![GEO oLoS](data/FRIIS-ABS-REFL-0/heatmap.png) | ![CSI oLoS](data/RECI-ABS-REFL-0/heatmap.png) |

Notes:
- Heatmaps show harvested DC power over a **1.25 m x 1.25 m** scan area.
- This README uses `data/RANDOM-1` as the RPS baseline (matching the paper figures); other random baselines are available under `data/RANDOM` and `data/RANDOM-2`.
- Some `data/*` folders also include `heatmap_vs_RANDOM*_dB.png` comparisons.

## Running the experiment (control node)

### Prerequisites
- Python 3 and Ansible
- SSH access to the tiles
- `tile-management` checked out at `~/tile-management` (or let `server/setup-server.sh` clone/update it)
- Tiles present in `~/tile-management/inventory/hosts.yaml`

### Setup
```bash
cd server
./setup-server.sh
source bin/activate
```

### Configure
Edit `experiment-settings.yaml`:
- `server.host`, `server.messaging_port`, `server.sync_port`
- `tiles` (space-separated list or group)
- choose `client_script_name`:
  - `run_gbwpt_phases.py` (GEO)
  - `run_reciprocity.py` (CSI)
  - `run_gbwpt_random_phases.py` (RPS)

### Recording measurements (energy profiler)
Update `FOLDER` inside `server/record/record-meas-energy-profiler.py`, then run:
```bash
python record/record-meas-energy-profiler.py
```
This script periodically writes `data/<FOLDER>/<timestamp>_{positions,values}.npy`.

### Start/stop the experiment service on the tiles
From `server/`:
```bash
python run-clients.py --start -a
python run-clients.py --stop -a
```

### Synchronization server (measurement rounds)
From repo root:
```bash
python server/record/sync-server.py --num-subscribers 42
```
Set `--num-subscribers` to the number of active tiles/clients (41 in the paper; 42 max in the setup).

## REF and pilot (paper workflow)

REF (absolute phase anchor): run on the REF machine using the UHD example scripts (not part of this repository):
```bash
python3 examples/tx_waveforms.py --args "type=b200" --freq 920e6 --rate 250e3 --duration 1E6 --channels 0 --wave-ampl 0.8 --gain 73 -w sine --wave-freq 0
```

Pilot (receiver-side pilot transmission; run on the pilot USRP host):
```bash
python3 client/usrp_pilot.py --phase 0
```

## Plotting

Regenerate heatmaps from recorded `data/*` runs:
```bash
python processing/plot_all_folders_heatmap.py --plot-all
```

## License
MIT (see `LICENSE`).
