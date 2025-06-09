# Graphical User Interface (GUI)

![SIMBA graphical user interface](/_static/fig_lukenet_gui.png)

To streamline the user experience, `SIMBA` includes a modern browser-based graphical interface built in React. The GUI provides dynamic visualization of abundance evolution, reaction rates, and chemical pathways, with comprehensive customization options. Species and reactions can be filtered based on abundance thresholds or selected manually to investigate specific chemical processes. Publication-quality figures can be exported in vector format (SVG), while numerical data can be extracted in standard formats for further analysis.

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
*Replace 18.x with your desired Node.js version if needed.*


#### CentOS/RHEL

Use the NodeSource setup script:
```bash
curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install -y nodejs
```

*For Fedora, use dnf instead of yum*


### Verify the installation:
```bash
node --version
npm --version
```

<br>
<br>

## Installation and Setup

The GUI is included as part of the standard `pip` installation:

```bash
pip install simba_chem
```

During installation, only the React source code is downloaded (not the built application) to keep the package size manageable. **The GUI will be built automatically on your system when first launched.**

<br>
<br>

## Launching the GUI

To launch the GUI from the command line:

```bash
simba-gui
```

or in Python:
```python
from simba_chem.gui_server import launch_gui
launch_gui()
```


**On first launch**, the system will:
1. Install the required JavaScript dependencies (`npm install`)
2. Build the React application (`npm run build`) - this may take a few minutes
3. Start the local server and automatically open your browser

**Subsequent launches** will start immediately since the application is already built. The application will open in your default browser.

<br>
<br>

## Using the GUI

The GUI features three main panels. The **Input Parameters** panel on the left organizes input parameters into logical groups covering gas/dust properties, environmental conditions, and network specifications. Parameters can be entered manually or loaded from a pre-configured `SIMBA` input file using the **Load Config** button. Once parameters are set, click **Run Solver** to execute the model. A progress bar at the bottom tracks the status.

Upon completion, results appear in the central **Visualization Panel**. The default view displays abundance evolution over time for the 8 most abundant species (ranked by final timestep abundances). Interactive tooltips provide precise values when hovering over the plot.

The **Control Panel** on the right enables switching between three visualization modes:
- **Abundances vs. Time** - Species concentration evolution
- **Reaction Rates vs. Time** - Chemical reaction rate evolution  
- **Pathway Diagram** - Formation and destruction pathway analysis

For abundance and reaction rate plots, users can either display the 'N' most abundant species automatically or manually select specific species from a dropdown menu. The pathway diagram allows selection of a target species and highlights the most efficient formation and destruction reactions based on final timestep rates.

Additional controls provide plot styling options, export of publication-quality figures, and download of raw data for further analysis.

A significant advantage of the GUI lies in its ability to rapidly explore the parameter space without the computational burden of full multi-dimensional models. While sophisticated codes like `DALI` provide comprehensive modelling capabilities, analysing the vast output from high-resolution models (often hundreds of gigabytes) can be cumbersome. `SIMBA` is intended to complement, rather than replace, such models by enabling efficient exploration of chemical evolution under varying conditions at specific points of interest. This makes it particularly valuable for investigating reaction mechanisms, understanding parameter sensitivities, and analysing localized chemical processes. Additionally, the intuitive interface serves as an effective and easy-to-use educational tool for students learning astrochemistry, allowing them to visualize how different physical conditions influence chemical evolution in real-time.