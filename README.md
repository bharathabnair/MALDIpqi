
<!-- README.md is generated from README.Rmd. Please edit that file -->

# MALDIpqi

<!-- badges: start -->

[![DOI](https://zenodo.org/badge/436219305.svg)](https://zenodo.org/badge/latestdoi/436219305)
<!-- badges: end -->

MALDIpqi calculates Parchment Glutamine Index from MALDI TOF ZooMS data.
It is a sample level measure of the glutamine deamidation from a list of
peptides.

For details in the data processing and mathematical method and models to
estimate PQI, refer to our publication Nair et al. (2022)

The method was initially developed to estimate parchment quality based
on deamidation, following the ideas from Wilson et al. (2012). However,
it can be applied in other tissues as in van Doorn et al. (2012) or
Brown et al. (2021)

## Installation

You can install the released version of MALDIpqi from github with:

``` r
install.packages('devtools')
#> Installing package into '/tmp/RtmpRxvg1B/temp_libpath23f4d680ddaa7'
#> (as 'lib' is unspecified)
# We need to tell R to also look in Bioconductor for the packages Spectra and mzR
setRepositories(ind=1:2)
devtools::install_github("ismaRP/MALDIpqi")
#> Using GitHub PAT from the git credential store.
#> Downloading GitHub repo ismaRP/MALDIpqi@HEAD
#> rlang     (1.1.3 -> 1.1.4 ) [CRAN]
#> evaluate  (0.23  -> 0.24.0) [CRAN]
#> farver    (2.1.1 -> 2.1.2 ) [CRAN]
#> SparseM   (1.81  -> 1.83  ) [CRAN]
#> minqa     (1.2.6 -> 1.2.7 ) [CRAN]
#> quantreg  (5.97  -> 5.98  ) [CRAN]
#> backports (1.4.1 -> 1.5.0 ) [CRAN]
#> waveslim  (1.8.4 -> 1.8.5 ) [CRAN]
#> ggsci     (3.0.3 -> 3.1.0 ) [CRAN]
#> Installing 9 packages: rlang, evaluate, farver, SparseM, minqa, quantreg, backports, waveslim, ggsci
#> Installing packages into '/tmp/RtmpRxvg1B/temp_libpath23f4d680ddaa7'
#> (as 'lib' is unspecified)
#> ── R CMD build ─────────────────────────────────────────────────────────────────
#> * checking for file ‘/tmp/RtmpXlcebm/remotes24c761dd0697a/ismaRP-MALDIpqi-95f47d6/DESCRIPTION’ ... OK
#> * preparing ‘MALDIpqi’:
#> * checking DESCRIPTION meta-information ... OK
#> * checking for LF line-endings in source and make files and shell scripts
#> * checking for empty or unneeded directories
#> * building ‘MALDIpqi_0.0.1.tar.gz’
#> Installing package into '/tmp/RtmpRxvg1B/temp_libpath23f4d680ddaa7'
#> (as 'lib' is unspecified)
```

## Example

``` r
data_folder = "data/mzML"
```

This is minimal workflow. It assumes the spectra are in data/mzML, in
mzML format and with file names in the form “samplename_replicate.ext”.
Where the replicate number is 1, 2 or 3 and ext is the extension of the
files.

``` r
# Read metadata and remove rows that don't have a corresponding mzML file
zooms_metadata = read_csv('./metadata.csv')
zooms_metadata = clean_metadata(zooms_metadata, data_folder)

# Read peptides, or don't provide to use default
peptides = read_csv('peptides.csv'))

# Preprocess spectra
peaks = MALDIpqi::preprocess_spectra(
  indir = data_folder, metadata = zooms_metadata,
  mono_masses = peptides$mass,
  smooth_wma_hws = 4,
  smooth_sg_hws = 6,
  iterations = 50,
  halfWindowSize = 20,
  snr = 2, k = 0L, threshold = 0.33,
  local_bg = FALSE,
  mass_range = 100, bg_cutoff = 0.5, l_cutoff = 1e-8,
  tolerance = 0.4, ppm = 50,
  n_isopeaks = 5,
  min_isopeaks = 4,
  ncores = 6, chunk_size = 60
)
peaks = prepare_peaks(peaks, peptides, n_isopeaks = 5)

# Calculate q2e for each sample, replicate and peptide
q2e_vals = peaks %>% filter(n_peaks > 0) %>%
  group_by(sample, replicate, pep_number) %>%
  summarise(wlm_q2e(norm_int, weight, deam_0, deam_1, deam_2))

# Calculate PQI
pqi_vals = lme_pqi(q2e_vals, logq = TRUE, g = 'free', return_model = TRUE)
```

## References

Nair, B. et al. (2022) ‘Parchment Glutamine Index (PQI): A novel method
to estimate glutamine deamidation levels in parchment collagen obtained
from low-quality MALDI-TOF data’, bioRxiv.
<doi:10.1101/2022.03.13.483627>.

Wilson, J., van Doorn, N.L. and Collins, M.J. (2012) ‘Assessing the
extent of bone degradation using glutamine deamidation in collagen’,
Analytical chemistry, 84(21), pp. 9041–9048.
<https://doi.org/10.1021/ac301333t>

van Doorn, N.L. et al. (2012) ‘Site-specific deamidation of glutamine: a
new marker of bone collagen deterioration’, Rapid communications in mass
spectrometry: RCM, 26(19), pp. 2319–2327.
<https://doi.org/10.1002/rcm.6351>

Brown, S. et al. (2021) ‘Examining collagen preservation through
glutamine deamidation at Denisova Cave’, Journal of archaeological
science, 133, p. 105454. <http://doi.org/10.1016/j.jas.2021.105454>

Bethencourt, J.H. et al. (2022) ‘Data from “A biocodicological analysis
of the medieval library and archive from Orval abbey, Belgium”’, Journal
of open archaeology data, 10(0). Available at:
<https://doi.org/10.5334/joad.89>.

Ruffini-Ronzani, N. et al. (2021) ‘A biocodicological analysis of the
medieval library and archive from Orval Abbey, Belgium’, Royal Society
Open Science, 8(6), p. 210210. <https://doi.org/10.1098/rsos.210210>
