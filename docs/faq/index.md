# FAQ

## General Questions

### What is SIMBA?
`SIMBA` is a Python software package that numerically solves time-dependent chemical reaction networks for astrophysical environments. The code is specifically optimized for modeling complex chemical evolution in settings such as the interstellar medium (ISM), molecular clouds, and protoplanetary disks.

### How does SIMBA compare to other astrochemistry codes?
`SIMBA` is designed to be compatible with the established `DALI` code format, allowing seamless use of chemical networks developed for `DALI` without requiring reformatting or conversion. This design choice facilitates network validation and comparison of results between codes. A detailed benchmarking of `SIMBA` against `DALI` is presented in our code overview paper.

### What physical processes does SIMBA model?
`SIMBA` models a comprehensive set of physical processes, including:
- Gas-phase chemistry (neutral-neutral, ion-neutral, etc.)
- Photochemistry (including self-shielding)
- Cosmic ray chemistry
- X-ray chemistry
- PAH chemistry
- Grain-surface processes (freeze-out, desorption, hydrogenation)
- H₂ formation and chemistry

## Installation & Setup

### Is SIMBA available via pip?
Not yet! While we plan to make `SIMBA` available via pip in the future, currently you need to download the source code from our repository.

### What are the system requirements for running SIMBA?
`SIMBA` requires:
- Python 3.8 or higher
- NumPy
- SciPy
- Matplotlib
- Numba (for just-in-time compilation of performance-critical routines)

### What are the computational requirements for SIMBA?
SIMBA is designed to be lightweight and can run on standard desktop or laptop hardware. For most applications with networks containing ~100-200 species and ~1000-2000 reactions, a modern dual-core processor with 4GB RAM is sufficient. Larger networks or longer timescales may benefit from additional computational resources. The GUI requires minimal additional resources beyond the core solver. Typical runtime is 5-30 seconds.

## Chemical Networks

### Can I use my own chemical network with SIMBA?
Yes! While `SIMBA` is configured by default to use chemical networks in the `DALI` format, the modular structure allows you to implement your own parser for different network formats. See the "Alternative Network Formats" section in the documentation for details.

### How do I add new reactions to the network?
To add new reactions to the network, you'll need to edit the chemical network file, ensuring you follow the correct format:
1. Each reaction should have properly defined reactants and products
2. Specify the appropriate reaction type code
3. Include all required rate coefficient parameters (α, β, γ)
4. Define valid temperature ranges

### What reaction types are supported?
`SIMBA` supports numerous reaction types including standard gas-phase reactions, photodissociation (with and without self-shielding), cosmic ray chemistry, X-ray chemistry, PAH reactions, grain chemistry, and H₂* chemistry. See the "Reaction Types" section in the documentation for the complete list.

## Running Simulations

### How can I improve convergence for difficult models?
If your model is having convergence issues, try these approaches:
1. Increase the temperature slightly to avoid very stiff reaction networks
2. Adjust the solver tolerance parameters using the `rtol` and `atol` options
3. Try running the model with a smaller chemical time to reach a stable state, then use those results as initial conditions for a longer run
4. Simplify your chemical network by removing unimportant reactions

### How do I interpret the output from SIMBA?
The terminal output provides a summary of the model parameters, integration statistics, and key results like the most abundant species and dominant reactions. More detailed results are stored in the returned solution object, which can be accessed for custom analysis or visualized using the built-in `Analysis` module.

### Can SIMBA handle time-dependent physical conditions?
The current version of `SIMBA` is designed for fixed physical conditions. However, time-dependent evolution can be approximated by running a series of models with different conditions and using the final state of each model as the initial conditions for the next.

## Visualization and Analysis

### How do I create custom plots of my results?
The `Analysis` module provides basic plotting routines, but you can also access the raw data in the solution object to create custom visualizations:

```python
# Access the raw data
time = network.time  # time points in seconds
abundances = network.species_abundance  # species abundances over time

# Create custom plots
import matplotlib.pyplot as plt
plt.figure(figsize=(10, 6))
plt.loglog(time/3.15e7, abundances[:, species_index], 'b-')  # convert to years
plt.xlabel('Time (years)')
plt.ylabel('Abundance')
plt.title(f'Abundance of {species_name} over time')
plt.grid(True, which="both", ls="-")
plt.show()
```

### How do I analyze the dominant formation/destruction pathways?
The `Analysis` module includes functionality to identify and visualize the dominant formation and destruction pathways for any species:

```python
# Analyze pathways for CO
analysis = simba.Analysis(network)
analysis.plot_pathways('CO')
```

### Can I export my results for use in other tools?
Yes, you can export the results to standard formats like CSV for further analysis:

```python
# Export abundances to CSV
analysis = simba.Analysis(network)
analysis.export_abundances('abundances.csv')
```

## Troubleshooting

### Why is my simulation running very slowly?
Chemical network simulations can be computationally intensive. To improve performance:
1. Use the included numba-optimized routines
2. Reduce the size of your chemical network if possible
3. Adjust the solver parameters for a performance/accuracy trade-off
4. For very large networks, consider running on a more powerful machine

### I'm getting "stiff system" warnings. What should I do?
Stiff systems are common in astrochemical networks due to the wide range of reaction timescales. `SIMBA` uses the BDF method which is designed for stiff systems, but you may need to:
1. Increase the maximum number of integration steps
2. Adjust the error tolerances
3. Check for unrealistic rate coefficients in your network

### How do I debug a custom chemical network?
To debug a custom chemical network:
1. Start with a simplified version of your network
2. Validate individual reactions against analytical solutions where possible
3. Use the `plot_reaction_rate` function to examine how specific reactions behave
4. Check the conservation of elements and charge as a sanity check

## Support and Contribution

### How do I report a bug or request a feature?
Please email [l.keyte@qmul.ac.uk](mailto:l.keyte@qmul.ac.uk) with a detailed description of the bug or feature request. Include information about your system, SIMBA version, and any relevant code snippets or input files.

### How can I contribute to SIMBA?
We welcome contributions! Please contact us to discuss how you might contribute to the project, whether through code development, documentation improvements, or network validations.

### Where can I find more examples of SIMBA in use?
Examples of `SIMBA` applications can be found in:
- The included examples directory
- Our code paper (citation in the documentation)
- [Keyte et al. 2023](https://ui.adsabs.harvard.edu/abs/2023NatAs...7..684K/abstract), which demonstrates a practical application of the code