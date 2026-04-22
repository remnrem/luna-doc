# Cross-frequency coupling

| Command | Description |
| ----- | ----- | 
| `CFC` | Cross-frequency coupling via the GLM method |
| `PAC` | Phase-amplitude coupling |


## CFC

_Estimates cross-frequency coupling via the GLM method_

The `CFC` command implements 
[van Wilk et al](https://www.ncbi.nlm.nih.gov//pubmed/25677405)'s method
for parametric cross-frequency coupling, using a simple and
computationally-efficient approach to phase-amplitude and
amplitude-amplitude coupling.

<h3>Methods</h3>

Cross-frequency coupling is estimated using the parametric GLM method of van Wijk et al. (2015). For each epoch, the signal is band-pass filtered at the lower (phase-providing) band A and upper (amplitude-providing) band B using the Kaiser-window FIR filter with the specified ripple and transition width, and the analytic signal is obtained via the Hilbert transform for each filtered result. Three z-scored regressors are constructed from band A: the sine and cosine of the instantaneous phase (capturing phase-amplitude coupling) and the instantaneous amplitude envelope (capturing amplitude-amplitude coupling). These regressors are entered into a general linear model with the z-scored amplitude envelope of band B as the dependent variable. Phase-amplitude coupling (R²_PAC) is derived as the sum of squared standardized regression coefficients for the sine and cosine terms, which equals the proportion of variance in the high-frequency amplitude explained by the low-frequency phase. Amplitude-amplitude coupling (C_AMP) is the Pearson correlation coefficient corresponding to the amplitude regressor. The total R² encompasses all three regressors.

<h3>Parameters</h3>

| Parameter | Example | Description |
| --- | --- | --- |
| `sig` | `sig=C3` | Signal(s) to analyse |
| `a` | `a=0.5,4` | Lower (phase) frequency band (Hz) |
| `b` | `b=12,16` | Upper (amplitude) frequency band (Hz) |
| `epoch` | `epoch` | Compute per-epoch output |
| `ripple` | `ripple=0.01` | Kaiser window ripple (default 0.02) |
| `tw` | `tw=0.5` | Filter transition width in Hz (default 0.5) |

<h3>Output</h3>

Per-signal, per-epoch output (strata: `CH`)

| Variable | Description |
| ---- | ---- |
| `FA` | Lower frequency range |
| `FB` | Upper frequency range |
| `R2_PAC` | Phase-amplitude coupling R² [0,1] |
| `C_AMP` | Amplitude-amplitude coupling correlation [−1,+1] |
| `Z_AMP` | Standardized AAC correlation |
| `R2_TOT` | Total CFC R-squared |
| `OKAY` | Flag indicating valid results |


## PAC

_Phase-amplitude coupling via the mean-vector-length method_

`PAC` estimates phase-amplitude coupling between a low-frequency phase-providing signal and a high-frequency amplitude-providing signal using a continuous wavelet transform approach.

<h3>Methods</h3>

For each specified phase frequency and amplitude frequency pair, Morlet continuous wavelet transforms are applied to extract the instantaneous phase at the lower frequency and the instantaneous amplitude envelope at the higher frequency. The mean-vector-length (MVL) statistic is computed as the modulus of the mean of the complex phasors formed by weighting unit-length phase vectors by the corresponding amplitude envelope values: MVL = |mean(amp · exp(i·phase))|. Statistical significance is assessed by a circular time-shift permutation test: the amplitude time-series is circularly shifted by a random offset (drawn uniformly from 1 to N−1 samples) relative to the phase time-series for the requested number of replicates (default 1000), and the null distribution of MVL values is used to derive an empirical p-value and a z-score for the observed MVL.

<h3>Parameters</h3>

| Parameter | Example | Description |
| --- | --- | --- |
| `sig` | `sig=C3` | Signal(s) to analyse |
| `ph` | `ph=0.5,1,2,4` | Phase-frequency grid values (Hz) |
| `amp` | `amp=12,15,18` | Amplitude-frequency grid values (Hz) |
| `nreps` | `nreps=1000` | Number of permutation replicates (default 1000) |
| `epoch` | `epoch` | Compute per-epoch output |

<h3>Output</h3>

Per-signal, per-frequency-pair output (strata: `CH` x `FPH` x `FAMP`)

| Variable | Description |
| ---- | ---- |
| `MVL` | Mean vector length PAC statistic |
| `MVLZ` | Z-score relative to permutation null |
| `MVLP` | Empirical p-value |





