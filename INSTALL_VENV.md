# Virtual Environment Setup Guide (`.venv`) for ABIDES

This guide describes the complete procedure for setting up from scratch a virtual environment compatible with **ABIDES** (`abides-core`, `abides-markets`, `abides-gym`) and the demo Jupyter notebooks.

---

## System Requirements

* **Python 3.9** (ABIDES and dependencies such as Ray 1.7.0 / Pomegranate 0.14 do not support Python ≥ 3.12).
* **uv** (recommended for fast Python version and package management) or standard `python3.9-venv`.
* C compiler (`gcc` / `build-essential`) for Pomegranate's Cython extensions.

---

## Step-by-Step Setup

Run the following commands from the repository root (`abides-jpmc-public`):

### 1. Create the virtual environment (Python 3.9)

```bash
# Remove any existing .venv
rm -rf .venv

# Create a new environment using Python 3.9 with seed packages (pip, setuptools)
uv venv .venv --python 3.9 --seed
```

> **Alternative without uv**: using Python's built-in `venv` module:
> ```bash
> python3.9 -m venv .venv
> ```

---

### 2. Activate the virtual environment

```bash
source .venv/bin/activate
```

---

### 3. Install build prerequisites for Pomegranate

Pomegranate 0.14 contains Cython modules that require `cython<3` and `setuptools<65` to compile properly:

```bash
pip install "cython<3" "setuptools<65" "numpy<1.24" "scipy<1.11"
```

---

### 4. Install Pomegranate

Compile and install `pomegranate` using the build prerequisites without build isolation:

```bash
pip install --no-build-isolation pomegranate==0.14.8
```

*(The compiled wheel is automatically cached locally by pip for instant subsequent reinstallations).*

---

### 5. Install Gym, Ray RLlib, and notebook dependencies

```bash
pip install "gym==0.22.0" \
            "ray[rllib]==1.7.0" \
            "protobuf<=3.20.3" \
            "torch==1.7.1" \
            "coloredlogs==15.0.1" \
            "pandas<2" \
            "psutil" \
            "tqdm" \
            "p_tqdm" \
            "pathos" \
            "matplotlib" \
            "seaborn" \
            "ipykernel" \
            pytest
```

* **`gym==0.22.0`**: includes prebuilt binary wheels and maintains full backward compatibility with the original environment API (`gym.make` and `env.seed()`).
* **`protobuf<=3.20.3`**: strictly required for compatibility with `ray==1.7.0` (prevents Protobuf 4 descriptor errors).

---

### 6. Install local ABIDES packages in editable mode

```bash
pip install -e abides-core -e abides-markets -e abides-gym
```

The `-e` (editable) flag ensures that any local modifications to the code in `abides-core`, `abides-markets`, or `abides-gym` take effect immediately without requiring package reinstallation.

---

### 7. Register the Jupyter Kernel

Register the environment as a Jupyter kernel so it can be selected inside VS Code or JupyterLab:

```bash
python -m ipykernel install --user --name abides-py39 --display-name "Python 3.9 (ABIDES .venv)"
```

---

## All-in-One Setup Script

To run the entire setup in a single command:

```bash
rm -rf .venv && \
uv venv .venv --python 3.9 --seed && \
source .venv/bin/activate && \
pip install "cython<3" "setuptools<65" "numpy<1.24" "scipy<1.11" && \
pip install --no-build-isolation pomegranate==0.14.8 && \
pip install "gym==0.22.0" "ray[rllib]==1.7.0" "protobuf<=3.20.3" "torch==1.7.1" "coloredlogs==15.0.1" "pandas<2" "psutil" "tqdm" "p_tqdm" "pathos" "matplotlib" "seaborn" "ipykernel" pytest && \
pip install -e abides-core -e abides-markets -e abides-gym && \
python -m ipykernel install --user --name abides-py39 --display-name "Python 3.9 (ABIDES .venv)"
```

---

## Verification & Testing

### Quick smoke test (Gym environment reset & step)

```bash
python -c '
import gym
import abides_gym

env = gym.make("markets-execution-v0", background_config="rmsc04")
env.seed(0)
state = env.reset()
action = env.action_space.sample()
state, reward, done, info = env.step(action)
print("ABIDES-Gym is working correctly! Reward:", reward)
'
```

### Run unit test suite

```bash
pytest abides-core/tests/ abides-markets/tests/
```

---

## Notebook Usage

Inside notebooks such as:
* [Demo_ABIDES-Gym.ipynb](notebooks/Demo_ABIDES-Gym.ipynb)
* [demo_ABIDES-Markets.ipynb](notebooks/demo_ABIDES-Markets.ipynb)

Select the kernel **`Python 3.9 (ABIDES .venv)`** (`abides-py39`) from the kernel selector located in the top-right corner of the editor interface.
