# Graphical User Interface (GUI)

![SIMBA graphical user interface](/_static/fig_lukenet_gui.png)

To enhance accessibility and user workflow, `SIMBA` includes a graphical interface implemented in React that integrates the core solver with interactive visualization capabilities. The GUI provides dynamic visualization of abundance evolution, reaction rates, and chemical pathways, with comprehensive customization options. Species and reactions can be filtered based on abundance thresholds or selected manually to investigate specific chemical processes. Publication-quality figures can be exported in vector format (SVG), while numerical data can be extracted in standard formats for further analysis.

## Prerequisites

The GUI requires **Node.js** and **npm** to be installed on your system. If you don't have these installed, follow the instructions below:

### Windows and macOS

- Download and install Node.js from the [official Node.js website](https://nodejs.org/). This installer also includes `npm`.

### Linux

#### Ubuntu/Debian

To get the latest stable release, use the NodeSource setup script:

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
```
(Replace 18.x with your desired Node.js version if needed.)


#### CentOS/RHEL

Use the NodeSource setup script:

```bash
curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install -y nodejs
```

(For Fedora, use dnf instead of yum.)


### Verify the installation:
```bash
node --version
npm --version
```


## Installation and Setup

The GUI is included as part of the standard `pip` installation:

```bash
pip install simba_chem
```

During installation, only the React source code is downloaded (not the built application) to keep the package size manageable. **The GUI will be built automatically on your system when first launched.**

## Launching the GUI

To launch the GUI from the command line:

```bash
simba-gui
```

Or in Python:
```python
from simba_chem.gui_server import launch_gui
launch_gui()
```


**On first launch**, the system will:
1. Install the required JavaScript dependencies (`npm install`)
2. Build the React application (`npm run build`) - this may take a few minutes
3. Start the local server and automatically open your browser

**Subsequent launches** will start immediately since the application is already built.

For development purposes, you can also launch in development mode with hot reloading:
```python
from simba_chem.gui_server import launch_gui_dev
launch_gui_dev()
```

## Capabilities and Use Cases

A key strength of the GUI is its ability to rapidly analyse parameter dependencies without the computational overhead of full multi-dimensional models. While codes like `DALI` (Bruderer et al. 2012) provide comprehensive modelling capabilities, analysing the vast output from high-resolution models (often hundreds of gigabytes) can be cumbersome. The `SIMBA` GUI complements such models by allowing users to efficiently explore chemical evolution under varying conditions at specific points of interest. This makes it particularly valuable for investigating reaction mechanisms, understanding parameter sensitivities, and analysing localized chemical processes.