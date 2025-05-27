# Chemical Model

The chemical model in `SIMBA` follows the standard rate equation approach to evolve molecular abundances under specified physical conditions. The model solves a system of coupled ordinary differential equations (ODEs) that describe the time evolution of species number densities:

$$
\frac{dn(i,t)}{dt} = \sum_j k_{ij} n(j,t) + \sum_{jl} k_{ijl} n(j,t) n(l,t)
$$
where $n(i,t)$ is the abundance ($\small{\text{cm}}^{-3}$) of species $i$ at time $t$, and $k_{ij}$ and $k_{ijl}$ are the respective destruction and formation rates of a given species.

<br/>

## Gas-phase processes

Gas-phase reactions follow standard temperature-dependent rate coefficients ($\small{k}$) of the modified Arrhenius form:

$$
k = \alpha \left(\frac{T_\text{gas}}{300\,\text{K}}\right)^\beta \exp\left(-\frac{\gamma}{T_\text{gas}}\right)
$$
where $\alpha$, $\beta$, and $\gamma$ are reaction-specific parameters, and $T_\text{gas}$ is the gas temperature. Each reaction has defined temperature limits ($T_\text{min}$ and $T_\text{max}$) that constrain its validity range.

### Photodissociation and photoionization
Photochemical processes, including direct photodissociation and photoionization, are parametrized using the standard form:

$$
k = \alpha \beta G_0 \exp(-\gamma A_\text{V})
$$
where $\small{G_0}$ is the integrated FUV field strength in Draine units ($\small \sim 2.7 \times 10^{-3} \; \text{erg s} ^{-3}\text{cm}^{-2}$), $\small{A_\text{V}}$ is the visual extinction, and $\alpha$, $\beta$, and $\gamma$ are reaction-specific parameters. The model can optionally account for self-shielding effects in CO, N₂, H₂, and atomic C, following the prescriptions of [Visser et al. (2009)](https://ui.adsabs.harvard.edu/abs/2009A%26A...503..323V/abstract), [Visser et al. (2018)](https://ui.adsabs.harvard.edu/abs/2018A%26A...615A..75V/abstract), [Draine & Bertoldi (1996)](https://ui.adsabs.harvard.edu/abs/1996ApJ...468..269D/abstract), and [Kamp & Bertoldi (2000)](https://ui.adsabs.harvard.edu/abs/2000A%26A...353..276K/abstract), respectively.

To calculate these self-shielding factors, the code requires the total hydrogen ($\small{H}+{H}_2$) column density between the point of interest and the UV source (denoted $\small{N}_\text{H}$). This can be provided directly by the user, or alternatively, can be approximated using the standard relationship between hydrogen column density and visual extinction eg. [Guver & Ozel (2009)](https://ui.adsabs.harvard.edu/abs/2009MNRAS.400.2050G/abstract):

$$
N_\text{H} \approx 2.21 \times 10^{21} A_\text{V}
$$
The column density of a given species is then inferred from the total hydrogen column using the local fractional abundance:

$$
N_\text{X} = N_\text{H} \times \frac{X}{n_\text{gas}}
$$
where $\small{X}$ denotes local number density of the species of interest.



### Cosmic-rays

Direct cosmic ray ionization is parametrized using the standard form:

$$
k = \alpha
$$
and cosmic ray photoreactions by:

$$
k = \alpha \frac{\zeta_\text{CR}+\zeta_\text{x}}{0.5} \bigg(\frac{T_\text{gas}}{300 \;\text{K}}\bigg)^\beta
$$
where in both cases $\alpha$ is taken from the [UMIST database](https://ui.adsabs.harvard.edu/abs/2007A%26A...466.1197W/abstract), which is normalised to a total rate
for electron production from cosmic ray ionisation (primarily from $\small{\text{H}}_2$ and $\small{\text{He}}$ in dark clouds) of $\small{\zeta_0} = 1.36 \times 10^{-17} \; \text{s}^{-1}$ [(Prasad & Huntress 1980)](https://ui.adsabs.harvard.edu/abs/1980ApJS...43....1P/abstract). 


### Other gas-phase reactions

The model also includes cosmic-ray and X-ray ionization processes, with both direct ionization and secondary electron effects ([Gredel et al. 1987](https://ui.adsabs.harvard.edu/abs/1987ApJ...323L.137G/abstract); [Maloney et al. 1996](https://ui.adsabs.harvard.edu/abs/1996ApJ...466..561M/exportcitation); [Yan 1997](https://ui.adsabs.harvard.edu/abs/1997PhDT.........4Y/abstract); [Stäuber et al. 2005](https://ui.adsabs.harvard.edu/abs/2005A%26A...440..949S/abstract)). 

Simple PAH chemistry is incorporated through charge exchange and electron attachment/detachment processes ([Le Page et al. 2001](https://ui.adsabs.harvard.edu/abs/2001ApJS..132..233L/abstract); [Wolfire et al. 2003](https://ui.adsabs.harvard.edu/abs/2003ApJ...587..278W/abstract); [Jonkheid et al. 2006](https://ui.adsabs.harvard.edu/abs/2006A%26A...453..163J/abstract)), with PAH abundances scaled relative to ISM values. 

Reactions with vibrationally excited H2 are included following [London (1978)](https://ui.adsabs.harvard.edu/abs/1978ApJ...225..405L/abstract); [Tielens & Hollenbach (1985)](https://ui.adsabs.harvard.edu/abs/1985ApJ...291..722T/abstract); [Visser et al. (2018)](https://ui.adsabs.harvard.edu/abs/2018A%26A...615A..75V/abstract).


<br/>

***

<br/>




## Grain processes

Grain-surface chemistry incorporates H₂ formation, hydrogenation, freeze-out, thermal desorption, and non-thermal desorption processes. 

The number density of dust grains ($n_\text{gr}$) is defined relative to the total gas number density ($n_\text{gas}$), scaled by the dust-to-gas ratio. For a standard interstellar dust-to-gas mass ratio of 1:100, we use a reference grain number density of:

$$ n_\text{gr,100} = 10^{-12} $$

This means that for every trillion gas particles, there is roughly one dust grain when the dust-to-gas ratio is 1:100. The actual grain density used in the chemistry calculations is then:

$$ n_\text{gr} = n_\text{gas} \cdot n_\text{gr,100} \cdot \small{\text{DG}_{100}} $$

where $\small \text{DG}_{100}$ is the dust-to-gas ratio normalized to the standard value of 1:100.

Each dust grain is assumed to have $\small N_\text{b} = 10^6$ binding sites where molecules can stick. This number comes from assuming spherical grains with radius $a_\text{gr} = 0.1$ μm and a typical surface site density of $\small N_\text{ss} = 8 \times 10^{14}$ cm⁻². The total number of sites is then simply the grain surface area multiplied by the site density:

$$ N_\text{b} = 4\pi a_\text{gr}^2 \cdot N_\text{ss} \approx 10^6 \text{ sites per grain} $$

This formulation allows us to properly account for how the availability of grain surface sites affects various processes like freeze-out, desorption, and surface chemistry.


### H₂ formation on dust

The H₂ formation rate on dust grains can be expressed as:

$$ k = s(\eta) \cdot \alpha \cdot T_\text{gas}^b \cdot \frac{n_\text{gas}}{1 + n_\text{H}} \cdot \small{\text{DG}_{100}} $$

where $s$ is the sticking coefficient and $\eta$ is the formation efficiency. The sticking coefficient depends on both gas and dust temperatures:

$$
s = \frac{1}{1 + 0.04\sqrt{T_\text{gas} + T_\text{dust}} + 2\times10^{-3} \; T_\text{gas} + 8\times10^{-6} \; T_\text{gas}^2}
$$

The formation efficiency $\eta$ follows two regimes. For very cold dust ($\small T_\text{dust} < 10 \text{ K}$), $\eta = 1$ as physisorbed H atoms efficiently form H₂. At higher temperatures, the efficiency becomes:

$$ \eta = \frac{\xi}{1 + 0.005\frac{f_\text{mlps}}{2\beta_\text{H₂}} + \beta_\alpha} $$

where $f_\text{mlps}$ is the flux of H atoms in monolayers per second, $\beta_\text{H₂}$ represents the desorption rate of physisorbed H₂, $\beta_\alpha$ accounts for the competition between diffusion and desorption, and $\xi$ is the probability of H₂ formation before desorption. This formulation, based on [Cazaux & Tielens (2002,](https://ui.adsabs.harvard.edu/abs/2002ApJ...575L..29C/abstract)[ 2004)](https://ui.adsabs.harvard.edu/abs/2004ApJ...604..222C/abstract) and updated by [Bosman et al. (2022)](https://ui.adsabs.harvard.edu/abs/2022ApJ...930L..26B/abstract), captures how H₂ formation becomes less efficient at higher temperatures due to faster desorption, while remaining possible through chemisorbed sites.

### Hydrogenation

The hydrogenation rate is implemented following [Visser et al. (2011)](https://ui.adsabs.harvard.edu/abs/2011A%26A...534A.132V/abstract):

$$
k = \pi a_\text{gr}^2 n_\text{gr} n(\text{H}) f(X) \sqrt{\frac{8kT_\text{gas}}{\pi m_\text{p}}}
$$

where $a_\text{gr}$ is the grain radius, $n_\text{gr}$ is the grain number density, $n(\text{H})$ is the number density of atomic hydrogen, $k$ is the Boltzmann constant, $T_\text{gas}$ is the gas temperature, and $m_\text{p}$ is the proton mass. 

The factor $\small f(X)$ is defined as

$$
f(X) = \frac{n_\text{s}(X)}{\text{max}(n_\text{hydro}, N_\text{b}n_\text{gr})}
$$

where $\small n_\text{s}(X)$ is the number density of species $\small X$ in the ice, $n_\text{hydro}$ is the total number density of all species that can be hydrogenated, and $N_\text{b}$ is the number of binding sites per grain. This factor ensures that hydrogenation occurs proportionally to the abundance of each species in the ice surface layer, where the reactions take place.

### Freeze-out

Freeze-out is implemented following [Visser et al. (2009)](https://ui.adsabs.harvard.edu/abs/2009A%26A...503..323V/abstract):

$$
k =\alpha \pi a_\text{gr}^2 n_\text{gr}  \sqrt{\frac{8kT_\text{gas}}{\pi m_\text{p}}}
$$

where $\alpha$ is the sticking coefficient (probability that a molecule sticks when it hits a grain), $a_\text{gr}$ is the grain radius, $n_\text{gr}$ is the grain number density, $k$ is the Boltzmann constant, $T_\text{gas}$ is the gas temperature, and $m_\text{p}$ is the proton mass. This rate essentially combines the geometric cross-section of dust grains with the thermal velocity of gas molecules, modified by how likely molecules are to stick when they collide with a grain surface.


### Thermal desorption

Thermal desorption is implemented following [Visser et al. (2009)](https://ui.adsabs.harvard.edu/abs/2009A%26A...503..323V/abstract):

$$
k = 4 \pi a_\text{gr}^2 n_\text{gr} f(X) \nu(X)N_\text{ss}\exp{\bigg(-\frac{E_\text{b}(X)}{T_\text{dust}}\bigg)}
$$

where $a_\text{gr}$ is the grain radius, $n_\text{gr}$ is the grain number density, $\small f(X)$ is a factor ensuring desorption scales with surface abundance, $\small \nu(X)$ is the characteristic vibration frequency of the molecule on the grain surface, $\small N_\text{ss}$ is the number of binding sites per unit grain area, $\small E_b(X)$ is the binding energy, $k$ is the Boltzmann constant, and $T_d$ is the dust temperature.

The factor $\small f(X)$ is defined as follows:

$$
f(X) = \frac{n_\text{s}(X)}{\text{max}(n_\text{ice}, N_\text{b}n_\text{gr})}
$$

where $\small n_\text{s}(X)$ is the number density of species $\small X$ in the ice, $n_\text{ice}$ is the total number density of all ice species, and $N_\text{b}$ is the number of binding sites per grain. This factor ensures that each species desorbs in proportion to its abundance in the ice, and the desorption behavior transitions naturally from zeroth-order when multiple layers of ice are present to first-order when less than one monolayer remains.

### Non-thermal desorption

Non-thermal desorption by UV photons or cosmic rays is implemented following [Visser et al. (2011)](https://ui.adsabs.harvard.edu/abs/2011A%26A...534A.132V/abstract):

$$
k = \pi a_\text{gr}^2 n_\text{gr} f(X) Y(X) F_0 \exp(-\tau_\text{UV})
$$

where $a_\text{gr}$ is the grain radius, $n_\text{gr}$ is the grain number density, $\small f(X)$  is the same factor as for thermal desorption, $\small Y(X)$ is the yield (number of molecules desorbed per grain per incident UV photon), $\small F_0$ is the unattenuated UV flux, and $\small \tau_\text{UV}$ is the effective UV extinction.

Additionally, a small background UV flux of $\small 10^4 \;\text{photons cm}^{-2} \text{ s}^{-1}$ is added to the stellar UV flux in order to simulate desorption by cosmic rays [(Shen et al. 2004)](https://ui.adsabs.harvard.edu/abs/2004A%26A...415..203S/abstract).

