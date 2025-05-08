# SIMBA Module Reference

## core.py

The core module implements the main chemical network solver functionality for SIMBA.

### <span style="background-color: #E9F2F9">*class* Simba</span>

Base class containing all functionality for solving astrochemical networks. SIMBA is designed to model and simulate complex chemical processes in various cosmic settings such as the ISM, molecular clouds, and protoplanetary disks.

#### Key Features
- Initialization of chemical species, reactions, and environmental parameters
- Efficient solving of stiff ODEs representing chemical reactions
- Support for various reaction types including gas-phase, grain-surface, and photochemistry
- Integration of self-shielding factors for specific molecules (H2, CO, N2, C)
- Optimization using Numba JIT compilation for performance-critical functions
- Comprehensive logging and progress tracking

#### Class Attributes

- `elements` (*model_classes.Elements*): Container for element-related data
- `species` (*model_classes.Species*): Container for chemical species data
- `gas` (*model_classes.Gas*): Gas phase properties and parameters
- `dust` (*model_classes.Dust*): Dust grain properties and parameters
- `environment` (*model_classes.Environment*): Environmental conditions
- `reactions` (*model_classes.Reactions*): Chemical reaction network data
- `parameters` (*model_classes.Parameters*): Solver parameters and constants
- `abundance_history` (*list*): Time evolution of species abundances

#### Methods

#### <span style="background-color:rgba(215, 217, 219, 0.33)">init_simba(*input_file*)</span>

Initialize the SIMBA chemical network solver with parameters from an input file.

**Parameters:**

- `input_file` (*str*): Path to the input parameter file

The input file should contain the following parameters:
- `n_gas`: Gas number density [cm^-3]
- `t_gas`: Gas temperature [K]  
- `n_dust`: Dust number density [cm^-3]
- `t_dust`: Dust temperature [K]
- `gtd`: Gas-to-dust ratio
- `Av`: Visual extinction [mag]
- `G_0`: UV field strength [Habing]
- `Zeta_X`: X-ray ionization rate [s^-1]
- `Zeta_CR`: Cosmic ray ionization rate [s^-1]
- `pah_ism`: PAH abundance relative to ISM
- `t_chem`: Chemical evolution time [years]
- `self_shielding`: Enable molecular self-shielding [bool]
- `column`: Enable column density calculations [bool]
- `h2_col`: H2 column density [cm^-2] (if column=True)


#### <span style="background-color:rgba(215, 217, 219, 0.33)">compute_rate_coefficients(*y*)</span>

Computes reaction rate coefficients for all chemical reactions in the network.

**Parameters:**

- `y` (*numpy.ndarray*): Current abundances of all species [cm^-3]

**Updates:**

- `reactions.k` (*numpy.ndarray*): Array of rate coefficients for all reactions
- `disso_H2` (*float*): H2 dissociation rate, used for reaction types 90 & 91

Handles various reaction types including:
- H2 formation on dust grains (Cazaux & Tielens 2002, 2004; Bosman et al. 2022)
- Hydrogenation reactions (Visser et al. 2011)
- Photodesorption processes (Visser et al. 2011)
- Gas-phase reactions with temperature dependencies
- Photodissociation with self-shielding
- Cosmic ray induced reactions (Stauber et al. 2005, Heays et al. 2014, Visser et al. 2018)
- X-ray induced reactions (Stauber et al. 2005, Bruderer et al. 2009b)
- PAH/grain charge exchange
- Thermal desorption and freezeout (Visser et al. 2009a, Visser et al. 2011)
- H2 excitation processes (Tielens & Hollenbach 1985, Visser et al. 2011)


#### <span style="background-color:rgba(215, 217, 219, 0.33)">solve_network()</span>

Solves the chemical reaction network using a stiff ODE solver.

This method integrates the system of ordinary differential equations that govern the chemical reaction network using SciPy's `solve_ivp` function with the backward differentiation formula (BDF) method.

**Returns:**

Dictionary containing:
- `time` (*numpy.ndarray*): Time points of the solution
- `abundances` (*numpy.ndarray*): Species abundances over time
- `rates` (*numpy.ndarray*): Reaction rates over the evaluated time points
- `success` (*bool*): Whether the integration was successful
- `message` (*str*): Solver message (success or error)
- `species` (*list*): Names of the chemical species
- `reaction_labels` (*list*): List of formatted strings describing each reaction

If the solver fails, returns a dictionary with:
- `success`: False
- `message`: Error message describing the failure
- `last_time`: Last successful integration time (if any)





<br/>
<br/>

***

## calculus.py

The calculus module implements core numerical methods for solving chemical reaction networks in `SIMBA`. It provides high-performance routines for computing reaction rates, species derivatives, and Jacobian matrices, serving as the computational foundation for the chemical network solver.

The module focuses on efficient computation of chemical reaction dynamics through optimized numerical methods. Performance optimization is achieved through Numba JIT compilation and sparse matrix operations.

#### Key Features
- Fast computation of chemical reaction rates and species derivatives
- Efficient generation of Jacobian matrices for ODE integration
- Support for complex reaction networks with multiple reactants and products
- Performance optimization using Numba JIT compilation
- Memory-efficient sparse matrix operations
- Handling of reaction networks with up to 3 reactants and 5 products per reaction


#### <span style="background-color:rgba(215, 217, 219, 0.33)">calculate_derivatives(*y*, *k*, *idx*, *ydot*, *nr*)</span>

Computes the formation and destruction rates for all chemical species in the network.

**Parameters:**
- `y` (*numpy.ndarray*): Species concentrations [cm^-3]
- `k` (*numpy.ndarray*): Reaction rate coefficients
- `idx` (*numpy.ndarray*): Reaction indices array with shape (nr, 8), where:
  - `idx[i, 0:3]`: Indices of up to 3 reactants
  - `idx[i, 3:8]`: Indices of up to 5 products
  - Unused indices are set to -1
- `ydot` (*numpy.ndarray*): Array for storing derivatives
- `nr` (*int*): Number of reactions

**Returns:**
A tuple containing:
- `numpy.ndarray`: Net change rates for each species (formation - destruction)
- `numpy.ndarray`: Individual reaction rates

**Implementation Notes:**
- Uses Numba JIT compilation for performance
- Handles reactions with variable numbers of reactants and products
- Computes both formation and destruction terms separately
- Accumulates rates through efficient array operations

#### <span style="background-color:rgba(215, 217, 219, 0.33)">calculate_jacobian_dense(*y*, *k*, *idx*, *nr*)</span>

Computes the dense Jacobian matrix representing partial derivatives of reaction rates.

**Parameters:**
- `y` (*numpy.ndarray*): Species concentrations [cm^-3]
- `k` (*numpy.ndarray*): Reaction rate coefficients
- `idx` (*numpy.ndarray*): Reaction indices array (same format as above)
- `nr` (*int*): Number of reactions

**Returns:**
- `numpy.ndarray`: Dense Jacobian matrix with shape (ns, ns), where ns is the number of species

**Implementation Notes:**
- Optimized with Numba JIT compilation
- Handles single-reactant and two-reactant reactions separately
- Computes all partial derivatives efficiently
- Each matrix element jac[j,i] represents ∂(dy[j]/dt)/∂y[i]
- Special handling for different reaction types:
  - Single-reactant reactions affect only one row/column
  - Two-reactant reactions include cross-terms for species interactions

#### <span style="background-color:rgba(215, 217, 219, 0.33)">calculate_jacobian(*y*, *k*, *idx*, *nr*)</span>

Converts the dense Jacobian matrix to a sparse format for efficient solver operations.

**Parameters:**
- `y` (*numpy.ndarray*): Species concentrations [cm^-3]
- `k` (*numpy.ndarray*): Reaction rate coefficients
- `idx` (*numpy.ndarray*): Reaction indices array (same format as above)
- `nr` (*int*): Number of reactions

**Returns:**
- `scipy.sparse.lil_matrix`: Sparse Jacobian matrix in LIL format

**Implementation Notes:**
- Calls calculate_jacobian_dense internally
- Converts dense matrix to scipy's LIL sparse format
- Preserves memory efficiency for large, sparse networks
- LIL format chosen for efficient matrix construction




<br/>
<br/>

***

## model_classes.py

The `model_classes.py` module serves as the foundational data structure layer for the SIMBA astrochemical network solver. It implements a comprehensive set of Python dataclasses that define and manage the core components of the chemical network simulation.

### <span style="background-color: #E9F2F9">*class* Elements</span>

A container class for managing chemical elements data within the network.

#### Attributes:
- `name` (List[str]): List of chemical element names

#### Methods:
- `validate()`: Performs validation checks on element data
- `info()`: Displays basic information about the elements

#### Validation Rules:
- Element names must be strings in a list format
- Names cannot be empty
- No duplicate element names allowed

### <span style="background-color: #E9F2F9">*class* Species</span>

Manages chemical species data and their properties within the network.

#### Attributes:
- `name` (List[str]): Chemical species names
- `abundance` (NDArray): Species abundances relative to total H
- `number` (NDArray): Number densities (cm^-3)
- `mass` (NDArray): Masses in atomic mass units (amu)
- `charge` (NDArray): Charges in elementary charge units (e)

#### Methods:
- `validate()`: Validates species data integrity
- `info()`: Displays species information summary

#### Validation Rules:
- All arrays must match the length of name list
- Abundances must be between 0 and 1
- Number densities and masses must be positive
- Charges must be whole numbers between -4 and +4

### <span style="background-color: #E9F2F9">*class* Gas</span>

Defines gas phase properties for the chemical network.

#### Attributes:
- `n_gas` (float): Gas number density (cm^-3)
- `t_gas` (float): Gas temperature (K)
- `h2_col` (float): H2 column density (cm^-2)

#### Methods:
- `validate()`: Validates gas property data
- `info()`: Displays gas property information

#### Validation Rules:
- All properties must be numerical
- Number density and temperature must be positive
- H2 column density must be non-negative

### <span style="background-color: #E9F2F9">*class* Dust</span>

Manages dust grain properties within the chemical network.

#### Attributes:
- `n_dust` (float): Dust grain number density (cm^-3)
- `t_dust` (float): Dust temperature (K)
- `radius` (float): Grain radius (cm)
- `binding_sites` (float): Number of molecular binding sites per grain

#### Methods:
- `validate()`: Validates dust property data
- `info()`: Displays dust property information

#### Validation Rules:
- All properties must be numerical and positive
- Default radius is 1e-5 cm (0.1 micron)
- Default binding sites is 1e6

### <span style="background-color: #E9F2F9">*class* Reactions</span>

Handles chemical reaction network data and properties.

#### Attributes:
- `educts` (List[List[str]]): Reactants for each reaction
- `products` (List[List[str]]): Products for each reaction
- `reaction_id` (NDArray): Unique reaction identifiers
- `itype` (NDArray): Reaction type identifiers
- `a`, `b`, `c` (NDArray): Rate coefficient parameters
- `temp_min`, `temp_max` (NDArray): Temperature range validity
- `pd_data` (NDArray): Photodissociation data
- `k` (NDArray): Calculated rate coefficients
- `labels` (List[str]): Human-readable reaction descriptions

#### Methods:
- `validate()`: Validates reaction network data
- `info()`: Displays reaction network information

#### Validation Rules:
- All arrays must have matching lengths
- Reaction IDs must be unique positive integers
- Temperature limits must be physically reasonable
- Rate coefficients must be non-negative

### <span style="background-color: #E9F2F9">*class* Environment</span>

Manages physical environmental conditions for the simulation.

#### Attributes:
- `gtd` (float): Gas-to-dust mass ratio
- `Av` (float): Visual extinction (magnitudes)
- `G_0` (float): FUV radiation field (Habing units)
- `G_0_unatt` (float): Unattenuated FUV field
- `Zeta_X` (float): X-ray ionization rate (s^-1)
- `Zeta_CR` (float): Cosmic ray ionization rate (s^-1)
- `pah_ism` (float): PAH abundance relative to ISM
- `dg100` (float): Normalized gas-to-dust ratio

#### Methods:
- `validate()`: Validates environmental conditions
- `info()`: Displays environmental parameters

#### Validation Rules:
- All values must be numerical
- Most parameters must be non-negative
- Gas-to-dust ratio must be positive

### <span style="background-color: #E9F2F9">*class* Parameters</span>

Contains simulation parameters and physical constants.

#### Attributes:
- `n_elements` (int): Number of chemical elements
- `n_species` (int): Number of chemical species
- `n_reactions` (int): Number of reactions
- `time_initial` (float): Initial time (seconds)
- `time_final` (float): Final time (seconds)
- `delta_v` (float): Velocity dispersion (km/s)
- `av_nH` (float): Column to extinction conversion
- `self_shielding` (bool): Self-shielding flag
- `column` (bool): Column-based shielding flag
- `k_B` (float): Boltzmann constant (erg/K)
- `yr_sec` (float): Seconds per year
- `m_p` (float): Proton mass (g)
- `validate()`: Validates parameter values
- `info()`: Displays parameter information

#### Validation Rules:
- Network parameters must be non-negative integers
- Time values must be positive with final > initial
- Physical constants must be positive
- Boolean flags must be proper boolean values






<br/>
<br/>

***

## selfshielding.py

The selfshielding module handles the calculation of self-shielding factors for H2, CO, N2, and atomic C.

#### Key Features
- Reading and interpolation of pre-computed CO shielding tables (Visser et al. 2009)
- Reading and interpolation of pre-computed N2 shielding tables (Visser et al. 2018)
- Analytical calculation of H2 self-shielding (Draine & Bertoldi 1996)
- Analytical calculation of atomic carbon self-shielding (Kamp & Bertoldi 2000)
- Support for temperature-dependent shielding effects
- Flexible column density handling with input or Av-based calculations


#### <span style="background-color:rgba(215, 217, 219, 0.33)">locate(*x*, *arr*)</span>

Finds interpolation indices and weights for a value within a sorted array.

**Parameters:**
- `x` (*float*): Value to locate
- `arr` (*array-like*): Sorted array to search within

**Returns:**
- `tuple`: Contains:
  - `index` (*int*): Lower bound index
  - `alpha` (*float*): Interpolation factor (0-1)


#### <span style="background-color:rgba(215, 217, 219, 0.33)">read_selfshielding_co(*file*)</span>

Reads CO self-shielding data from pre-computed tables.

**Parameters:**
- `file` (*str*): Path to CO self-shielding data file

**Returns:**
- `tuple`: Contains:
  - `chem_coss_NCO` (*numpy.ndarray*): Log10 CO column density grid
  - `chem_coss_NH2` (*numpy.ndarray*): Log10 H2 column density grid
  - `chem_coss` (*numpy.ndarray*): Log10 self-shielding factors (2D array)

**Data Format:**
- Expects 47 CO column density points
- Expects 42 H2 column density points
- Based on Visser et al. (2009) calculations

#### <span style="background-color:rgba(215, 217, 219, 0.33)">read_selfshielding_n2(*file*)</span>

Reads N2 self-shielding data from pre-computed tables.

**Parameters:**
- `file` (*str*): Path to N2 self-shielding data file

**Returns:**
- `tuple`: Contains:
  - `chem_n2ss_NN2` (*numpy.ndarray*): Log10 N2 column density grid
  - `chem_n2ss_NH2` (*numpy.ndarray*): Log10 H2 column density grid
  - `chem_n2ss_NH` (*numpy.ndarray*): Log10 H column density grid
  - `chem_n2ss` (*numpy.ndarray*): Log10 self-shielding factors (3D array)

**Data Format:**
- Expects 46 N2 column density points
- Expects 46 H2 column density points
- Expects 10 H column density points
- Based on Visser et al. (2018) calculations

#### <span style="background-color:rgba(215, 217, 219, 0.33)">calc_selfshielding_co(*chem_coss_NCO*, *chem_coss_NH2*, *chem_coss*, *col_h2*, *col_co*)</span>

Calculates CO self-shielding factor using bilinear interpolation.

**Parameters:**
- `chem_coss_NCO` (*numpy.ndarray*): Log10 CO column density grid
- `chem_coss_NH2` (*numpy.ndarray*): Log10 H2 column density grid
- `chem_coss` (*numpy.ndarray*): Log10 self-shielding factors
- `col_h2` (*float*): Current H2 column density [cm^-2]
- `col_co` (*float*): Current CO column density [cm^-2]

**Returns:**
- `float`: CO self-shielding factor (linear scale)

#### <span style="background-color:rgba(215, 217, 219, 0.33)">calc_selfshielding_n2(*chem_n2ss_NN2*, *chem_n2ss_NH2*, *chem_n2ss_NH*, *chem_n2ss*, *col_h2*, *col_h*, *col_n2*)</span>

Calculates N2 self-shielding factor using trilinear interpolation.

**Parameters:**
- `chem_n2ss_NN2` (*numpy.ndarray*): Log10 N2 column density grid
- `chem_n2ss_NH2` (*numpy.ndarray*): Log10 H2 column density grid
- `chem_n2ss_NH` (*numpy.ndarray*): Log10 H column density grid
- `chem_n2ss` (*numpy.ndarray*): Log10 self-shielding factors
- `col_h2` (*float*): Current H2 column density [cm^-2]
- `col_h` (*float*): Current H column density [cm^-2]
- `col_n2` (*float*): Current N2 column density [cm^-2]

**Returns:**
- `float`: N2 self-shielding factor (linear scale)

#### <span style="background-color:rgba(215, 217, 219, 0.33)">calc_selfshielding_h2(*col_h2*, *delta_v*)</span>

Calculates H2 self-shielding factor using the Draine & Bertoldi (1996) approximation.

**Parameters:**
- `col_h2` (*float*): H2 column density [cm^-2]
- `delta_v` (*float*): Velocity dispersion [km/s]

**Returns:**
- `float`: H2 self-shielding factor

**Implementation Notes:**
- Uses fixed delta_v = 0.2 km/s
- Includes both low and high column density limiting behavior
- Accounts for line overlap effects

#### <span style="background-color:rgba(215, 217, 219, 0.33)">calc_selfshielding_c(*col_h2*, *col_c*, *t_gas*)</span>

Calculates atomic carbon self-shielding factor using the Kamp & Bertoldi (2000) approximation.

**Parameters:**
- `col_h2` (*float*): H2 column density [cm^-2]
- `col_c` (*float*): C column density [cm^-2]
- `t_gas` (*float*): Gas temperature [K]

**Returns:**
- `float`: C self-shielding factor

**Implementation Notes:**
- Includes both C self-shielding and H2 cross-shielding
- Temperature-dependent H2 shielding component
- Enforces minimum H2 shielding factor of 0.5




<br/>
<br/>

***

## analysis.py






<br/>
<br/>

***


## helpers.py

The helpers module provides essential utility functions for the SIMBA chemical network solver. It handles file I/O operations, data parsing, and formatting tasks that support the core solver functionality. The module consists of standalone functions that handle various auxiliary tasks required by the main SIMBA solver.

#### Key Features
- Reading and parsing of input configuration files
- Processing of chemical network files with complex formatting requirements
- Generation of human-readable reaction labels for output
- Comprehensive logging functionality
- DALI model cell parameter extraction and display

### Functions

#### <span style="background-color:rgba(215, 217, 219, 0.33)">read_input_file(*file_path*)</span>

Reads a configuration file containing key-value pairs and converts them to appropriate Python data types.

**Parameters:**
- `file_path` (*str*): Path to the input configuration file

**Returns:**
- `dict`: Dictionary containing parsed parameters with properly typed values

The function handles various data types including:
- Boolean values ('true'/'false')
- Integer numbers
- Floating point numbers
- String values (with quote stripping)

#### <span style="background-color:rgba(215, 217, 219, 0.33)">read_chemnet(*network_file*)</span>

Parses a chemical network file containing species information and reaction data.

**Parameters:**
- `network_file` (*str*): Path to the chemical network file

**Returns:**
A tuple containing:
- `n_elements` (*int*): Number of elements
- `elements_name` (*list*): Element names
- `n_species` (*int*): Number of species
- `species_name` (*list*): Species names
- `species_abu` (*numpy.ndarray*): Initial abundances
- `species_mass` (*numpy.ndarray*): Species masses
- `species_charge` (*numpy.ndarray*): Species charges
- `n_reactions` (*int*): Number of reactions
- `reactions_educts` (*list*): Reactants for each reaction
- `reactions_products` (*list*): Products for each reaction
- `reactions_reaction_id` (*numpy.ndarray*): Reaction IDs
- `reactions_itype` (*numpy.ndarray*): Reaction type identifiers
- `reactions_a` (*numpy.ndarray*): Rate coefficient parameter a
- `reactions_b` (*numpy.ndarray*): Rate coefficient parameter b
- `reactions_c` (*numpy.ndarray*): Rate coefficient parameter c
- `reactions_temp_min` (*numpy.ndarray*): Minimum valid temperatures
- `reactions_temp_max` (*numpy.ndarray*): Maximum valid temperatures
- `reactions_pd_data` (*list*): Photodissociation data

#### <span style="background-color:rgba(215, 217, 219, 0.33)">create_reaction_labels(*n_reactions*, *educts*, *products*)</span>

Generates human-readable string representations of chemical reactions.

**Parameters:**
- `n_reactions` (*int*): Number of reactions to process
- `educts` (*list*): List of lists containing reactant species names
- `products` (*list*): List of lists containing product species names

**Returns:**
- `list`: Formatted strings representing each reaction (e.g., "A + B -> C + D")

#### <span style="background-color:rgba(215, 217, 219, 0.33)">log_section(*title*)</span>

Creates a formatted section header for logging output.

**Parameters:**
- `title` (*str*): Section title to be displayed



#### <span style="background-color:rgba(215, 217, 219, 0.33)">log_param(*name*, *value*, *unit=""*)</span>

Formats and logs a parameter with aligned name, value, and optional unit.

**Parameters:**
- `name` (*str*): Parameter name
- `value` (*any*): Parameter value
- `unit` (*str*, optional): Unit of measurement (default: "")


#### <span style="background-color:rgba(215, 217, 219, 0.33)">print_dali_cell(*dali_model_outdat_path*, *r*, *z*)</span>

Extracts and displays physical parameters from a specified DALI model cell.

**Parameters:**
- `dali_model_outdat_path` (*str*): Path to the DALI out.dat file
- `r` (*int*): Radial index of the cell
- `z` (*int*): Vertical index of the cell

**Displays:**
- Gas density and temperature
- Dust density and temperature
- Gas-to-dust ratio
- Visual extinction (Av)
- UV field strength
- X-ray ionization rate
- H2 column density

#### <span style="background-color:rgba(215, 217, 219, 0.33)">create_input(*path_to_output*)</span>

Creates a new input file with default SIMBA parameters.

**Parameters:**
- `path_to_output` (*str*): Path where the input file should be created