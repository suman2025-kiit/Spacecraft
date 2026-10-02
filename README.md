# Spacecraft#
pip install -r requirements.txt


python run_all.py



python -m unittest discover -s tests -v


# Spacecraft-Fraud FPGA Validation Code

GitHub-ready reference implementation for **Spacecraft-Fraud: Secure Onboard Federated Telemetry Mining for Spacecraft Fault Detection and Subsystem Localization**.

This repository provides a reproducible software-to-FPGA validation flow for spacecraft telemetry fault detection using a quantized LSTM-GCN-inspired inference core, fixed-point arithmetic, AXI-style input checking, watchdog recovery, and blockchain/federated-learning security checks.

## Main Contributions

- Onboard telemetry preprocessing for multi-subsystem spacecraft signals.
- Quantized LSTM-GCN-style fault-detection pipeline suitable for FPGA deployment.
- Fixed-point inference path for AMD-Xilinx ZCU104 style validation.
- RTL modules for inference, AXI-stream integrity checking, and watchdog recovery.
- Testbench and vector-generation scripts for FPGA simulation.
- Federated trust aggregation with hash, timestamp, similarity, reliability, and consistency checks.
- Example validation logs for anomaly score, fault flag, subsystem ID, confidence, IRQ, and cycle count.

## Repository Structure

```text
spacecraft-fraud-fpga/
├── README.md
├── requirements.txt
├── LICENSE
├── src/
│   └── spacecraft_fraud/
│       ├── __init__.py
│       ├── config.py
│       ├── data.py
│       ├── fixed_point.py
│       ├── model.py
│       ├── security.py
│       ├── federated.py
│       └── run_pipeline.py
├── fpga/
│   ├── rtl/
│   │   ├── axi_stream_guard.v
│   │   ├── lstm_gcn_fault_core.v
│   │   └── watchdog_recovery.v
│   ├── tb/
│   │   └── tb_lstm_gcn_fault_core.v
│   └── constraints/
│       └── zcu104_template.xdc
├── scripts/
│   ├── generate_vectors.py
│   └── run_sim.sh
└── docs/
    └── GITHUB_UPLOAD_GUIDE.md
```

## Installation

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
pip install -e . --no-build-isolation
```

## Run the Complete Software Validation Flow

```bash
python -m spacecraft_fraud.run_pipeline
```

If editable installation is unavailable, run directly with:

```bash
PYTHONPATH=src python -m spacecraft_fraud.run_pipeline
```

Expected output:

```text
Spacecraft-Fraud FPGA validation run
Samples: 512
Detection accuracy: ...
Precision: ...
Recall: ...
F1-score: ...
AUC proxy: ...
Mean inference latency: ... ms
TC01-PASS preprocessing
TC02-PASS fixed-point inference
TC03-PASS trust validation
TC04-PASS watchdog recovery
```

## Generate FPGA Test Vectors

```bash
python scripts/generate_vectors.py --output fpga/tb/input_vectors.mem
```

The generated memory file can be loaded in the Verilog testbench:

```verilog
$readmemh("input_vectors.mem", sample_mem);
```

## Run RTL Simulation

With Icarus Verilog:

```bash
sudo apt-get install iverilog
bash scripts/run_sim.sh
```

With Vivado:

1. Create a new RTL project.
2. Add files from `fpga/rtl/`.
3. Add `fpga/tb/tb_lstm_gcn_fault_core.v` as simulation source.
4. Add `fpga/constraints/zcu104_template.xdc` for ZCU104 pin/clock planning.
5. Run behavioral simulation, then synthesis and implementation.

## FPGA Architecture

The FPGA validation flow follows six stages:

1. Telemetry sources provide subsystem sensor streams.
2. AFE/ADC converts physical signals into digital telemetry.
3. PS/CPU performs windowing, normalization, security tagging, and DMA transfer.
4. ZCU104 PL executes quantized LSTM-GCN-style inference.
5. Output logic reports anomaly score, fault flag, subsystem ID, confidence, IRQ, error status, and cycle count.
6. Hashes and logs are sent to federated aggregation and audit.

## Input Format

The default code uses synthetic spacecraft telemetry with the following features:

| Feature | Description |
| --- | --- |
| `power_bus_v` | Power bus voltage |
| `battery_temp_c` | Battery temperature |
| `reaction_wheel_rpm` | Reaction wheel speed |
| `imu_vibration_g` | IMU vibration level |
| `thermal_plate_c` | Thermal plate temperature |
| `comms_snr_db` | Communication signal-to-noise ratio |
| `payload_current_a` | Payload current |
| `attitude_error_deg` | Attitude pointing error |

To use a real CSV file, replace the synthetic generator in `data.py` or call `load_telemetry_csv()`.

## Notes for Research Use

This repository is a reproducible validation package. The included synthetic telemetry, FPGA testbench, and logs demonstrate the intended implementation flow. For measured hardware claims, run the RTL/HLS design on the actual AMD-Xilinx ZCU104 board and report board-level measurements separately.

## Citation

If you use this code, cite the associated manuscript:

```bibtex
@article{majumder2026spacecraftfraud,
  title={Spacecraft-Fraud: Secure Onboard Federated Telemetry Mining for Spacecraft Fault Detection and Subsystem Localization},
  author={Majumder, Suman},
  journal={Manuscript under review},
  year={2026}
}
```

