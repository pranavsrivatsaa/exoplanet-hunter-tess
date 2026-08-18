# 🔭 Exoplanet & Stellar Transit Hunter (TESS Pipeline)

A complete step-by-step Python pipeline running in **Google Colab** to pull, clean, analyze, and vet space telescope data from NASA's TESS (Transiting Exoplanet Survey Satellite) archive via **Lightkurve**. 

---

## 🚀 What This Project Does
* **Pulls Data from MAST:** Downloads real light curves for designated stars measured by NASA's TESS.
* **Flattens & Cleans:** Removes stellar "mood swings," instrument noise, and outlier spikes while keeping physical transit dips intact.
* **Period Search (BLS):** Uses Box Least Squares algorithms to test thousands of possible orbital periods and find repeating signals.
* **Phase Folding:** Stacks data points over time to map out clear transit profiles.
* **Astrophysical Vetting:** Runs diagnostic tests (Secondary Eclipse and Odd-Even Transit checks) to distinguish exoplanets from eclipsing binary star systems.

---

## 🛠️ The Toolkit
* **Google Colab:** Browser-based Python notebook environment.
* **Lightkurve:** NASA-funded Python package for downloading and analyzing Kepler and TESS data.
* **MAST Archive:** Public repository holding raw space photometry data.
* **ExoFOP-TESS & NASA Exoplanet Archive:** Reference databases used to check known candidates and stellar parameters.

---

## 📂 Notebook Structure & Code Workflow

### **1. Setup & Installation**
```python
%pip install lightkurve -q

import lightkurve as lk
import numpy as np
2. Warm-Up: Re-finding Pi Mensae c
Searching and downloading the first official TESS planet discovery (Sector 0-3 data):

Python
search = lk.search_lightcurve("Pi Mensae", author="SPOC", exptime=120)
lc = search[0:3].download_all().stitch().remove_nans()
3. Data Processing & BLS Search
Flattening the curve to isolate signals and scanning for periodicity:

Python
flat = lc.flatten(window_length=401)
flat = flat.remove_outliers(sigma_upper=4, sigma_lower=np.inf)

period_grid = np.linspace(0.5, 15, 20000)
bls = flat.to_periodogram(method="bls", period=period_grid, frequency_factor=500)
4. Automated Rapid-Fire Hunt Function
A consolidated function to quickly scan arbitrary star targets (such as HD 63433 or AU Mic):

Python
def hunt(star, min_period=0.5, max_period=15):
    search = lk.search_lightcurve(star, author="SPOC", exptime=120)
    if len(search) == 0:
        search = lk.search_lightcurve(star, author="QLP")
    if len(search) == 0:
        print(f"No TESS data for {star}, try another star")
        return
    lc = search[0:3].download_all().stitch().remove_nans()
    flat = lc.flatten(window_length=401)
    flat = flat.remove_outliers(sigma_upper=4, sigma_lower=np.inf)
    grid = np.linspace(min_period, max_period, 20000)
    bls = flat.to_periodogram(method="bls", period=grid, frequency_factor=500)
    period = bls.period_at_max_power
    t0 = bls.transit_time_at_max_power
    depth = bls.depth_at_max_power
    print(f"Star: {star}  |  TIC: {lc.meta.get('TICID', 'see search table')}")
    print(f"Best period: {period:.4f}")
    print(f"Depth: {depth:.5f} ({depth * 100:.3f} percent)")
    print(f"Power score: {bls.max_power:.4f}")
    folded = flat.fold(period=period, epoch_time=t0)
    ax = folded.scatter(s=1)
    folded.bin(time_bin_size=10 / 60 / 24).plot(ax=ax, color="red", lw=2)
5. Vetting & False Positive Checks
Secondary Eclipse Test (Check 1): Scans the halfway point of the orbital phase to verify whether a secondary companion dip is present (indicative of an eclipsing binary).

Python
half = flat.fold(period=planet_period, epoch_time=planet_t0 + planet_period / 2)
half.scatter(s=1)
Odd vs. Even Transit Test (Check 2): Compares alternating transits to ensure consistent depth formatting.

Python
ax = folded[folded.odd_mask].scatter(s=1, label="odd transits")
folded[folded.even_mask].scatter(ax=ax, s=1, color="red", label="even transits")
```
##📊 Results & Case Analysis (TIC 261136679)
Running pipeline diagnostics on target TIC 261136679 yielded the following output metrics and structural behavior:

Best Period: 13.7167 d

Transit Depth: 0.00094 (0.094%)

BLS Power Score: 11230.1160

Diagnostic Observations:
BLS Spectrum & Folded Flux: Displays a high-power period spike accompanied by distinct primary and secondary architectural signatures.

Vetting Outcome: The appearance of secondary structures during phase validation indicates that this target maps to an eclipsing binary star system rather than a single transiting exoplanet—demonstrating the successful filtration of astrophysical false positives using the vetting module.
