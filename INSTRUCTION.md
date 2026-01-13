# Toy Brick Assortment Optimization - Instructions

## Project Overview
Deliver insights from toy brick inventory data using mathematical optimization to determine which sets can be built from available parts.

### What You'll Build:
1. Data foundation with Rebrickable toy brick data
2. Optimization model using Gurobi solver
3. Natural language-ready data structures for analysis

### Value Proposition:
- **Inventory Optimization:** Maximize utilization of available toy brick parts
- **Decision Support:** Determine optimal set selection based on constraints

---

## Prerequisites

### Snowflake Setup
- Snowflake account with appropriate permissions
- Warehouse: `COMPUTE_WH` or `GEN2_SMALL`
- Role: `ACCOUNTADMIN` or role with CREATE DATABASE privileges

### Local Tools
- SnowCLI (v3.14.0 or later)
  ```bash
  pipx install snowflake-cli
  # or upgrade existing
  pipx upgrade snowflake-cli
  ```

### Gurobi License
- [Commercial License](https://www.gurobi.com/solutions/licensing/)
- Free trial available for small problems

---

## 3-Phase Implementation

### Phase 1 – Data Foundation

1. **Download Rebrickable Data**
   - Visit https://rebrickable.com/downloads/
   - Download CSV files (not gzipped):
     - themes.csv, sets.csv, inventories.csv, colors.csv
     - parts.csv, part_categories.csv
     - inventory_parts.csv, inventory_sets.csv
     - minifigs.csv, inventory_minifigs.csv
     - elements.csv, part_relationships.csv

2. **Create Database and Schema**
   ```bash
   snow sql -c demo -q "CREATE DATABASE IF NOT EXISTS TOY_BRICK_DB"
   snow sql -c demo -q "CREATE SCHEMA IF NOT EXISTS TOY_BRICK_DB.RAW_DATA"
   ```

3. **Create Stage**
   ```bash
   snow sql -c demo -q "
   CREATE OR REPLACE STAGE TOY_BRICK_DB.RAW_DATA.REBRICKABLE_STAGE
   FILE_FORMAT = (TYPE = CSV 
                  FIELD_DELIMITER = ',' 
                  SKIP_HEADER = 1 
                  FIELD_OPTIONALLY_ENCLOSED_BY = '\"'
                  NULL_IF = ('', 'NULL', 'null'));
   "
   ```

4. **Upload CSV Files**
   ```bash
   cd DATA/
   snow stage copy themes.csv @TOY_BRICK_DB.RAW_DATA.REBRICKABLE_STAGE -c demo
   snow stage copy sets.csv @TOY_BRICK_DB.RAW_DATA.REBRICKABLE_STAGE -c demo
   snow stage copy inventories.csv @TOY_BRICK_DB.RAW_DATA.REBRICKABLE_STAGE -c demo
   snow stage copy colors.csv @TOY_BRICK_DB.RAW_DATA.REBRICKABLE_STAGE -c demo
   snow stage copy parts.csv @TOY_BRICK_DB.RAW_DATA.REBRICKABLE_STAGE -c demo
   snow stage copy part_categories.csv @TOY_BRICK_DB.RAW_DATA.REBRICKABLE_STAGE -c demo
   snow stage copy inventory_parts.csv @TOY_BRICK_DB.RAW_DATA.REBRICKABLE_STAGE -c demo
   snow stage copy inventory_sets.csv @TOY_BRICK_DB.RAW_DATA.REBRICKABLE_STAGE -c demo
   snow stage copy minifigs.csv @TOY_BRICK_DB.RAW_DATA.REBRICKABLE_STAGE -c demo
   snow stage copy inventory_minifigs.csv @TOY_BRICK_DB.RAW_DATA.REBRICKABLE_STAGE -c demo
   snow stage copy elements.csv @TOY_BRICK_DB.RAW_DATA.REBRICKABLE_STAGE -c demo
   snow stage copy part_relationships.csv @TOY_BRICK_DB.RAW_DATA.REBRICKABLE_STAGE -c demo
   ```

5. **Create Tables and Load Data**
   - Run the SQL commands in `01_Prepare_Data_Snowflake.ipynb` (or see commented sections)
   - Tables are created with schemas matching CSV columns
   - COPY INTO commands load data from stage

### Phase 2 – Optimization Model Setup

1. **Run Notebook 02 (Small Example)**
   - Uses 4 owned sets as starting inventory
   - Considers 50 candidate sets
   - Demonstrates Gurobi optimization basics

2. **Run Notebook 03 (Large Scale)**
   - Uses 100+ owned sets
   - Considers 500+ candidate sets
   - Includes heuristic comparison

### Phase 3 – Analysis & Iteration

1. **Compare Strategies**
   - Gurobi optimization vs greedy heuristics
   - Maximize parts used vs maximize set count

2. **Sensitivity Analysis**
   - Identify binding constraints (bottleneck parts)
   - What-if scenarios

---

## Technical Components

### Data Model
```
themes (1) ──┬── sets (N)
             │
             └── sets ──┬── inventories ──┬── inventory_parts ──── parts
                        │                 │
                        │                 └── inventory_minifigs ── minifigs
                        │
                        └── inventory_sets
```

### Key Views
- `v_set_parts`: Denormalized view of all set-part relationships
- `v_part_color_inventory`: Aggregated inventory by part-color

### Optimization Tables
- `part_color_pairs`: Unique part-color combinations
- `set_part_requirements`: Requirements matrix for optimization

---

## Success Metrics

✅ **Data Loaded:** 12 tables with 1.7M+ rows total
✅ **Complex Query Support:** Views join 6+ tables efficiently
✅ **Optimization Ready:** Part-color requirements matrix created

---

## Output Transformation

**Traditional:** "You can build 15 sets from your inventory"

**Enhanced:** "You can build 15 sets using 85% of your 10,000 pieces. The bottleneck parts are Red 2x4 Bricks (fully utilized). Consider acquiring 20 more to unlock 3 additional sets."

---

## Customization Points

1. **Owned Sets:** Modify the list of sets you own
2. **Candidate Sets:** Filter by theme, year, part count
3. **Objective:** Maximize parts used OR maximize set count
4. **Constraints:** Add budget limits, theme preferences, etc.

---

## File Structure

```
GUROBI/
├── 01_Prepare_Data_Snowflake.ipynb      # Data loading notebook
├── 02_Optimization_Model_Small_Snowflake.ipynb  # Small example
├── 03_Optimization_Model_Large_Snowflake.ipynb  # Large scale
├── README.md                            # Project documentation
├── INSTRUCTION.md                       # This file
├── prompt.txt                           # Project context
├── .gitignore                           # Git ignore rules
├── DATA/                                # CSV data files (not in git)
│   ├── themes.csv
│   ├── sets.csv
│   └── ... (12 files total)
└── OLD/                                 # Archive folder
```

---

## Troubleshooting

### SnowCLI Connection Issues
```bash
snow connection list  # Check available connections
snow connection test -c demo  # Test connection
```

### Data Loading Errors
- Check CSV file encoding (should be UTF-8)
- Verify NULL handling in FILE_FORMAT
- Use ON_ERROR = CONTINUE for partial loads

### Gurobi License
- Ensure GUROBI_HOME environment variable is set
- Check license file location
- For Snowflake notebooks, Gurobi must be available in the environment

---

## References

- [Rebrickable Downloads](https://rebrickable.com/downloads/)
- [Databricks Toy Brick Solution](https://github.com/databricks-industry-solutions/Toy-Brick-Assortment)
- [Gurobi Documentation](https://www.gurobi.com/documentation/)
- [Snowflake Notebooks (Private Preview)](https://docs.snowflake.com/en/user-guide/ui-snowsight/notebooks)
