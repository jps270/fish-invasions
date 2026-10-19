# Data dictionary: FW_species_richness_nat_exotic_Sinas_Tedesco.csv

Produced by `fish_invasions.Rmd`. The same columns are in `FW_species_richness_nat_exotic_Sinas_Tedesco_no_NativeBasin.csv`, which drops rows whose status is `NativeBasin`.

## What a row is

One row per **food web × fish species** (each `FoodWeb_ID` × `Taxon` pair appears once). The species come from two places:

- **Liuyong basin species list** (most rows): every fish species that Liuyong (2026) lists for the drainage basin the food web sits in. These are the species of the whole basin, not only the fish sampled in the food web.
- **IsoFresh fish not on that list** (1,097 rows): fish sampled in the food web whose species isn't on the basin list. These rows have only the IsoFresh and SInAS columns filled; the basin and Tedesco columns are blank.

Duplicates have been merged. Before merging, 25,478 pairs had 2 or more rows, almost always because Tedesco lists the species in the basin under several source names that now resolve to the same `Taxon`. In a merged row, columns whose values differed hold all of them joined with `;`, e.g. `X2.Species.Name.in.Source` = `Notropis.stramineus;Notropis.ludibundus`. This happens mainly in the Tedesco columns.

Blank cells are missing values. Counts of every adjustment (name updates, duplicates merged, rows per status) are in `adjustments_summary.csv`.

## Sources

| Short name | Source |
|---|---|
| IsoFresh | Boulêtreau et al. 2025, global stable isotope database of freshwater food webs ([Zenodo 14982323](https://zenodo.org/records/14982323)) |
| Liuyong | Liuyong 2026, freshwater fish species by drainage basin, Catalog of Fishes names as of Dec 2024 ([Zenodo 19511163](https://doi.org/10.5281/zenodo.19511163)) |
| Tedesco | Tedesco et al. 2017, freshwater fish occurrence in drainage basins ([figshare](https://doi.org/10.6084/m9.figshare.c.3739145.v1)) |
| SInAS | Gómez-Suárez et al. 2025, native and alien distributions by country, v3.1.1 ([Zenodo 18220953](https://zenodo.org/records/18220953)) |
| FishBase | Current FishBase, accessed through the `rfishbase` R package |

## Columns

### Food web (IsoFresh)

| Column | Type | Description |
|---|---|---|
| `FoodWeb_ID` | text | IsoFresh food web identifier, e.g. `ID_0001`. Never blank. |
| `Country` | text | Country the food web is in, as recorded by IsoFresh. |
| `Longitude` | number | Food web longitude, decimal degrees (WGS84). |
| `Latitude` | number | Food web latitude, decimal degrees (WGS84). |
| `Site_name` | text | IsoFresh site name. |
| `Ecosystem` | text | `Lentic`, `Lotic` or `Both`. |
| `Country_ISO` | text | ISO 3166 three-letter code for `Country`, e.g. `CAN`. Used to match FishBase country records. |
| `Region` | text | IsoFresh continental region, e.g. `North America`. |

### Species

| Column | Type | Description |
|---|---|---|
| `Taxon` | text | Species name, updated to the current FishBase valid name. Synonyms are replaced by the valid name. Names FishBase doesn't know, or flags as misapplied or ambiguous, are left as in the source. Never blank. |
| `class` | text | Taxonomic class (Liuyong). |
| `order` | text | Taxonomic order (Liuyong). |
| `family` | text | Taxonomic family (Liuyong). |

### Drainage basin (Liuyong)

The food web is assigned to the nearest Liuyong basin polygon.

| Column | Type | Description |
|---|---|---|
| `basin` | text | Basin name, e.g. `Saint.Laurent`. |
| `basin_d` | text | Basin identifier, e.g. `B0750`. |
| `bggrph_` | text | Biogeographic realm, e.g. `Nearctic`, `Neotropic`. |
| `country` | text | All countries the basin covers, separated by `;`. Different from `Country`, which is the food web's own country. |
| `n_specs` | integer | Number of fish species in the basin, as given in the Liuyong basin layer. |
| `cntr_ln` | number | Longitude of the basin centroid. |
| `cntr_lt` | number | Latitude of the basin centroid. |

### Tedesco occurrence record

Blank where Tedesco has no record of the species in the basin.

| Column | Type | Description |
|---|---|---|
| `X2.Species.Name.in.Source` | text | The species name as it appears in Tedesco's original source, with `.` in place of spaces. Not updated to current names. Several names joined with `;` where merged rows had different source names. |
| `X3.Native.Exotic.Status` | text | **Native/exotic status from all sources combined.** See the next section. Always a single value. |
| `X4.TSN.ITIS.Code` | text | ITIS taxonomic serial number, from Tedesco. Blank more often than the other Tedesco columns. Stored as text because merged rows can hold several codes joined with `;`. |
| `X5.Fishbase.Species.Code` | text | FishBase species code, from Tedesco (2017). It may refer to an older name than `Taxon`. Text for the same reason as `X4`. |
| `X7.Occurrence.Status` | text | Tedesco's confidence in the occurrence: `valid` or `questionable`, or `valid;questionable` where merged records disagreed. |

### SInAS status

Filled only for IsoFresh-sampled fish found in SInAS for that country; blank everywhere else.

| Column | Type | Description |
|---|---|---|
| `establishmentMeans_introduced` | 0/1 | 1 if SInAS lists the species as introduced in the food web's country. |
| `establishmentMeans_native` | 0/1 | 1 if SInAS lists the species as native in the food web's country. Both columns can be 1 when a species is native in part of a country and introduced in another part. |

## `X3.Native.Exotic.Status` values

Each row takes its status from the first source in this list that has one. Lower sources only fill rows the sources above them left blank.

| Value | Source | Level | Meaning | Rows |
|---|---|---|---|---|
| `native` | Tedesco | basin | Native in the basin. | 164,621 |
| `exotic` | Tedesco | basin | Introduced in the basin. | 426 |
| `MixedTedesco` | Tedesco | basin | Tedesco lists the species in the basin as both native and introduced. | 9 |
| `ExoticSinas` | SInAS | country | Introduced in the country and not native anywhere in it. | 484 |
| `MixedSinas` | SInAS | country | Both native and introduced in the country (native in part of it, introduced elsewhere). | 434 |
| `NativeSinas` | SInAS | country | Native in the country. | 8 |
| `NativeFishBase` | FishBase | country | Native or endemic in the food web's country. | 18,345 |
| `ExoticFishBase` | FishBase | country | Introduced in the food web's country. | 97 |
| `NativeBasin` | FishBase | basin | Native in another country of the same basin, with no FishBase record for the food web's country. The species is probably not present in the food web's country. | 6,587 |
| `NativeLiuyong` | Liuyong | basin | No FishBase species record (mostly described in the last decade). Assumed native because Liuyong's lists come from Catalog of Fishes native ranges. | 11,984 |
| *(blank)* | | | No source gives a status. | 986 |

FishBase statuses use only whole-country records. Sub-national areas such as Hawaii are ignored, and so are FishBase records marked questionable, not established or extirpated.

Rows with a `Native…` value can be treated as native, rows with an `Exotic…` value as exotic, and `Mixed…` values as mixed. Each fallback keeps its own label, so any of them can be filtered out.

Row counts are from the run on 2026-10-17 (203,981 rows). `adjustments_summary.csv` is regenerated on every run and always has the current counts.
