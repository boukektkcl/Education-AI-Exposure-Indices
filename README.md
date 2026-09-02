# Public education-level AI-exposure indices

This folder contains AI-exposure indices for university degrees in 51 countries: for each university × subject pair, a score for how exposed its graduates' careers are to large language models. The indices combine Revelio Labs LinkedIn data with the occupation-level AI-exposure scores of Eloundou et al. (2024). They accompany the paper ["Artificial Intelligence and the Future Supply of Skills: Evidence from UK University Applications"](https://ssrn.com/abstract=7354159) by Aristotle Vossos, Andrei Andronic, Bouke Klein Teeselink, and Kartik Akileswaran. Every country's index applies the exact data restrictions of the paper's UK index. The release covers 66,385 university × subject pairs across 51 countries.

## What the index measures

The index answers how exposed to large language models the jobs that graduates of a given degree actually end up in are. Construction proceeds in three steps.

1. **Occupation level.** Eloundou et al. (2024) score each O*NET task with β: 1 if a large language model alone can halve the time the task takes at equal quality, ½ if that requires additional software built on the LLM, 0 otherwise. We average β across each occupation's tasks using O*NET task-importance weights, which gives every occupation a score between 0 and 1. Revelio assigns each LinkedIn position an O*NET occupation code, which links jobs to these scores.
2. **Graduate level.** For each bachelor's graduate, we average the β scores of their post-graduation jobs, weighting each job by the number of months the graduate held it after graduation. The first-jobs variant instead uses only each graduate's first job after graduation (ties on the start date are averaged).
3. **Degree level.** We average the graduate-level scores across all graduates of the same subject at the same university, counting each graduate once.

The **main index** (`*_exposure_main.csv`, all jobs, duration-weighted) is the paper's headline measure; the **first-jobs index** (`*_exposure_first_jobs.csv`) is its robustness variant. Higher values mean more exposed; published UK values run from 0.26 to 0.68. β measures whether LLMs can perform the work, not whether the jobs will disappear: it pools tasks where AI replaces the worker with tasks where it assists them.

## Data restrictions (identical for every country)

- Bachelor's graduates of universities in the country itself, with a non-missing subject field (Revelio's global profile data: `degree == "Bachelor"`, `university_country == <country>`).
- Graduation cohorts 2002–2022.
- Jobs come from the country's Revelio Profiles extract and must carry an O*NET occupation code with an Eloundou et al. (2024) score. The extract contains every position that was **active on or after 1 January 2021** — an end date on or after that day, or no end date yet — whatever its start date; about half of all positions started earlier. Positions that ended before 2021 are not in the data.
- **No labour-market data after 1 October 2022 enters the index**: jobs that start later are dropped, ongoing spells are capped at that date, and later graduation cohorts are excluded. This keeps the index free of any post-ChatGPT labour-market response. Together with the extract window above, every job in the index was therefore held at some point between 1 January 2021 and 1 October 2022.
- Only jobs active at or after graduation count. Each job's weight is its post-graduation duration: the months from the later of its start date and graduation to the earlier of its end date and 1 October 2022. Months before 2021 therefore do count, for jobs that survived into 2021.
- Only university × subject pairs with **at least 10 graduates** are published.

## The data files

Each country has a folder `Exposure indices/<Country>/` with two files: `<Country>_exposure_main.csv` (the headline index) and `<Country>_exposure_first_jobs.csv` (the robustness variant). `Exposure indices/coverage_summary.csv` adds one row of sample-size and coverage statistics per country.

The `United_Kingdom` folder holds two extra files, `United_Kingdom_ucas_provider_cah1_exposure_main.csv` and `United_Kingdom_ucas_provider_cah1_exposure_first_jobs.csv` (1,268 rows; 212 UCAS providers × 12 CAH1 subject groups). These aggregate the same graduate-level exposures to UCAS provider × CAH1 subject group, via the paper's hand-reviewed university crosswalk and its field-to-CAH1 mapping, and reproduce the index used in the paper exactly.

## Columns

Index files:

| Column | Description |
|---|---|
| `university_name` | Revelio-standardized university name (free text). UK UCAS files use `ucas_provider` (UCAS code + name) instead |
| `field` | Revelio subject field, 17 categories (listed below). UK UCAS files use `cah1` (HECoS CAH1 subject group) instead |
| `exposure_alljobs_beta` | Main index: duration-weighted mean β of graduates' post-bachelor jobs (main files) |
| `exposure_firstjob_beta` | First-jobs index: mean β of graduates' first post-bachelor job (first-jobs files) |
| `n_people` | Number of graduates behind the pair (minimum 10) |

The 17 fields: Accounting, Architecture, Biology, Business, Chemistry, Economics, Education, Engineering, Finance, Information Technology, Law, Marketing, Mathematics, Medicine, Nursing, Physics, Statistics. The two files of a country cover the same university × subject pairs, so `n_people` is identical across them.

`coverage_summary.csv`:

| Column | Description |
|---|---|
| `country` | Country name |
| `n_profile_users` | Unique LinkedIn users in the country's Revelio Profiles extract |
| `n_person_programs` | Bachelor's degrees entering the index: graduate × degree records that meet every restriction above, before the 10-graduate minimum |
| `n_universities` | Universities in the published index |
| `n_pairs_published` | University × subject pairs published |
| `mean_exposure_alljobs_beta` | Unweighted mean of the main index across the country's published pairs |

## How to use the index

The files are plain CSVs; any statistics package reads them directly. In R:

```r
library(data.table)
fExposure <- fread("Exposure indices/Germany/Germany_exposure_main.csv")
fExposure[, exposure_z := scale(exposure_alljobs_beta)]   # per-SD units within country
```

- Check `coverage_summary.csv` before using a country: coverage differs by an order of magnitude across countries (see caveats below). Within-country comparisons across degrees are the intended use; cross-country level comparisons inherit the coverage differences.
- `university_name` is Revelio free text. Matching it to an official university register requires a crosswalk. For the UK, skip that step: use the UCAS files, which are keyed on UCAS provider codes and match UCAS applications data directly.
- `n_people` measures how many graduates a pair's score rests on. Weight by it, or restrict to larger pairs, when precision matters.

## Comparability caveats

- Revelio standardizes degree levels across countries, but the "Bachelor" label tracks national degree systems. Coverage is high where bachelor's degrees dominate (about 11% of UK and German profile users hold a domestic bachelor record) and low where they do not (about 2% in France, where Licence and Grande École credentials rarely standardize to "Bachelor"). China has no Profiles extract and is not included.
- LinkedIn penetration differs across countries and occupations; white-collar work is over-represented everywhere.
- The Profiles extracts contain only positions active on or after 1 January 2021, so jobs that ended before 2021 never enter (jobs that survived into 2021 enter with their full post-graduation duration). For older cohorts the index therefore reflects mid-career rather than early-career occupations.
- Revelio data count positions, not deduplicated workers, at the job level; the index counts each graduate once at the degree level.

## Sources

- **Revelio Labs**: LinkedIn profiles (positions and education records), extracts of all positions active on or after 1 January 2021.
- **Eloundou, T., Manning, S., Mishkin, P., & Rock, D. (2024)**, "GPTs are GPTs: Labor market impact potential of LLMs," *Science* 384(6702): task-level α/β exposure labels.
- **O*NET**: task-importance ratings used to aggregate task scores to occupations.

## License

The indices and this documentation are released under the [Creative Commons Attribution 4.0 International license](https://creativecommons.org/licenses/by/4.0/) (CC BY 4.0), the same license as the paper: share and adapt the data freely, with attribution — cite the paper below.

## Citation and contact

If you use these indices, cite: Vossos, A., Andronic, A., Klein Teeselink, B., & Akileswaran, K. (2026), "Artificial Intelligence and the Future Supply of Skills: Evidence from UK University Applications," SSRN working paper, https://ssrn.com/abstract=7354159, doi:10.2139/ssrn.7354159. The code that constructs the indices is available upon request. Questions and corrections: bouke.klein_teeselink@kcl.ac.uk.
