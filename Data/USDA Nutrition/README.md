# USDA Nutrition data

A relational extract of whole-food / base-ingredient nutrition from the U.S. government's
**USDA FoodData Central** database. One CSV per table; the files join together on the ID
columns noted below.

## Source & attribution

**USDA FoodData Central** — https://fdc.nal.usda.gov/

> U.S. Department of Agriculture, Agricultural Research Service. *FoodData Central.*
> fdc.nal.usda.gov. Accessed 2026-07-16.

FoodData Central is a work of the U.S. federal government and is in the **public domain**
(not subject to domestic copyright). No permission is needed to use or redistribute it;
the citation above is the USDA's requested acknowledgment.

This extract is limited to **two** of FoodData Central's data types — the whole/base
ingredients, not the ~400k branded UPC products:

| Dataset | Version | Rows | What it is |
|---|---|---|---|
| SR Legacy | 2018-04 | 7,793 | Classic Standard Reference base ingredients |
| Foundation Foods | 2025-12-18 | 436 | Newest, deeply-analyzed whole foods |

## Files

| File | Rows | Description | Key |
|---|---|---|---|
| `food.csv` | 8,229 | One row per food/ingredient | `fdc_id` (PK) |
| `food_nutrient.csv` | 663,926 | The macro + micro values, per 100 g of food | `(fdc_id, nutrient_id)` |
| `nutrient.csv` | 477 | Reference list of nutrients (energy, protein, vitamins, minerals…) | `nutrient_id` (PK) |
| `food_portion.csv` | 14,636 | Household measures (cup, tbsp…) → gram weight, per food | `id` (PK) |
| `food_category.csv` | 28 | USDA food categories (Fruits, Cereal Grains…) | `id` (PK) |
| `measure_unit.csv` | 123 | Household measure units | `id` (PK) |

### How they relate

```
food.food_category_id ─────► food_category.id
food_nutrient.fdc_id ──────► food.fdc_id
food_nutrient.nutrient_id ─► nutrient.nutrient_id
food_portion.fdc_id ───────► food.fdc_id
food_portion.measure_unit_id ► measure_unit.id
```

A food's nutrition is `food` ⋈ `food_nutrient` ⋈ `nutrient`, where `food_nutrient.amount`
is the amount of that nutrient **per 100 g** of the food (in `nutrient.unit_name` units).

### Column notes

- `food_nutrient.amount` — per **100 g** of the edible food.
- `nutrient.unit_name` — the unit the amount is in (`G`, `MG`, `UG`, `KCAL`, `IU`…).
- `nutrient.nutrient_class` and `nutrient.display_name` are **not** raw USDA columns — they
  were added downstream (`nutrient_class` groups nutrients as
  energy/macronutrient/lipid/amino_acid/mineral/vitamin/other; `display_name` is a friendly
  short label). Every other column is straight from FoodData Central.
- **Energy appears more than once per food** in `food_nutrient`: most foods carry both a
  kcal (`nutrient_id` 1008) and a kJ (1062) row, and some report only Atwater-derived energy.
  Pick one basis when summing.

## What was intentionally left out

- **App-created composite foods** — 2 synthetic rows (negative `fdc_id`, e.g. a sourdough
  starter blend) that a downstream project injected into the live `food`/`food_nutrient`
  tables were removed, so this is a pure USDA FoodData Central extract.
- **Non-USDA reference tables** in the same database — FDA Daily Values and
  National-Academies Dietary Reference Intakes — are a different source (not USDA
  FoodData Central) and are not included here.
- **Branded Foods and Survey/FNDDS** FoodData Central datasets were never loaded.

_Extracted 2026-07-16._
