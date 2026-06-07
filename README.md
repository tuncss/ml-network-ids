# ML-Based Network Intrusion Detection and Response System

A mini next-generation firewall (NGFW) proof of concept that captures live network
traffic, classifies each flow with a machine-learning model, and automatically blocks
malicious source IPs in real time. Built for the *Introduction to Computer Security* course.

## Overview

The system reconstructs network flows from live packet capture, extracts statistical
features, classifies them with a Random Forest model trained on the **CICIDS2017** dataset,
and streams the results to a real-time web dashboard. Flows classified as malicious trigger
an automated response that blocks the source IP via `iptables`.

## Pipeline

```
Packet capture (Scapy) -> flow builder -> feature extractor
    -> Random Forest classifier -> action engine (iptables) -> event bus -> live dashboard
```

| Module | Responsibility |
|---|---|
| `ngfw/sniffer.py` | Live packet capture with Scapy |
| `ngfw/flow_builder.py` | Groups packets into bidirectional flows |
| `ngfw/feature_extractor.py` | Computes statistical flow features |
| `ngfw/classifier.py` | Random Forest inference (scikit-learn) |
| `ngfw/action_engine.py` | Blocks malicious source IPs via iptables |
| `ngfw/event_bus.py` | Publishes events to the dashboard |
| `ngfw/dashboard/` | Flask + Socket.IO real-time dashboard |

## Tech stack

- **Python 3.12**
- **Scapy** — live packet capture
- **scikit-learn** — Random Forest classifier (trained on CICIDS2017)
- **pandas / imbalanced-learn** — data preparation and class balancing
- **Flask + Flask-SocketIO** — real-time dashboard
- **iptables** — automated IP blocking
- **pytest** — unit tests

## Project structure

```
ngfw/         Core pipeline (capture, flow, features, classifier, action, dashboard)
attacker/     Attack-simulation scripts for the demo (port scan, brute force, SYN flood)
setup/        VM provisioning scripts (firewall VM and Kali attacker VM)
notebooks/    Model training and real-world false-positive-rate analysis
models/       Trained model artifacts
tests/        Unit tests
docs/         Design spec, implementation plan, and report figures
```

## Getting started

> Requires Python 3.12.x. Live capture and iptables actions need root/administrator privileges.

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

Train (or retrain) the model from `notebooks/01_train_model.ipynb`, then start the system:

```bash
sudo python -m ngfw.main
```

Open the dashboard in your browser (default `http://localhost:5000`).

## Demo

The `attacker/` scripts reproduce a labeled demo from a second machine (for example the Kali
VM provisioned by `setup/kali_vm_setup.sh`):

```bash
attacker/run_demo.sh        # normal traffic, port scan, brute force, SYN flood
attacker/reset_demo.sh      # clears iptables rules created during the demo
```

## Tests

```bash
pytest
```

## Disclaimer

Educational proof of concept. Run it only on networks and machines you own or are
explicitly authorized to test.
