# Dynamic Foraging — NWB Packaging Capsule

Packages a raw [dynamic foraging](https://github.com/AllenNeuralDynamics/Aind.Behavior.DynamicForaging)
acquisition into an NWB file. The capsule loads the raw acquisition through the
data contract, assembles the NWB acquisition entries and the `trials` table, and
writes the NWB (Zarr) store plus its `processing.json` metadata.

> **Note:** Data must be acquired in the `aind-behavior-dynamic-foraging` data
> contract format to be compatible with this capsule.

## Input
A single raw acquisition directory mounted under `/data`. It must contain the
data-contract streams (Harp device registers, software events, camera data) and
the acquisition metadata files used to seed the NWB file:

- `acquisition.json` (or `session.json`), `subject.json`, `data_description.json`,
  `procedures.json`, `processing.json`

```
/data/
└── <asset_name>/            # the acquisition directory
    ├── acquisition.json
    ├── subject.json
    ├── data_description.json
    ├── ...
    └── behavior/            # Harp registers, software events, ...
```

## Output
Written to `/results`:

| File | Description |
| --- | --- |
| `behavior.nwb.zarr` | The NWB file (Zarr backend) with the acquisition module and the `trials` table. |
| `processing.json` | `aind-data-schema` processing metadata for the packaging step. |

The NWB file contains:

- **Acquisition**
  - Four derived event series: `left_reward_delivery_time` /
    `right_reward_delivery_time` (reward deliveries per lick port, each annotated
    `earned` / `manual` / `automatic`) and `left_lick_time` / `right_lick_time`
    (detected licks per lick port).
  - Every raw contract stream, packaged verbatim as a `DynamicTable`.
- **Trials table** — one row per trial (see
  [`trials_table_mapping.md`](trials_table_mapping.md)).

## How it works
The capsule is a thin wrapper over `dynamic_foraging_processing.pipeline.Pipeline`:

```python
from pathlib import Path

from dynamic_foraging_processing.pipeline import Pipeline
from dynamic_foraging_processing.raw_data_loader import RawDataLoader

DATA_DIR = Path("/data")
RESULTS_DIR = Path("/results")

# The acquisition directory is the single dataset mounted under /data.
acquisition_dir = next(path for path in DATA_DIR.iterdir() if path.is_dir())

loader = RawDataLoader(path=acquisition_dir)
Pipeline(loader).run_nwb(RESULTS_DIR)
```

`run_nwb` writes `RESULTS_DIR / "behavior.nwb.zarr"` and `RESULTS_DIR / "processing.json"`.

## Environment
The capsule installs `dynamic-foraging-processing` with the `full` extra, which
pulls in `aind-nwb-utils`, `pynwb`, `aind-data-schema` and `hdmf-zarr` on top of the base
`aind-behavior-dynamic-foraging` data contract:

```bash
pip install "dynamic-foraging-processing[full] @ git+https://github.com/AllenNeuralDynamics/dynamic-foraging-processing.git"
```

