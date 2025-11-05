# Company Name Semantic Matching

This project provides a workflow for **identifying whether two company names refer to the same real-world organization**, even when the text differs due to abbreviations, suffix variations, punctuation, or alternate naming conventions.

The approach combines:
- **Vector-based language embeddings**
- **Fine-tuned similarity modeling**
- **Synthetic training data generation to reinforce domain-specific patterns**

This allows the model to correctly distinguish relationships such as:
- **"Acme Holdings LLC"** ≈ **"Acme, Inc."** (same company)
- **"Acme Logistics"** ≠ **"Acme Capital"** (similar prefixes, different entities)
- **"Blue River Solutions"** ≠ **"Red River Solutions"** (similar structure, different identities)

---

## Project Structure

| File | Purpose |
|------|---------|
| `build-training-data.ipynb` | Generates training examples, including positive and hard negative pairs, with controlled suffix/prefix variation logic. |
| `fine-tune-model.ipynb` | Fine-tunes a sentence-transformer model on the generated dataset to improve similarity scoring. |
| `fuzzy-semantic-matching.ipynb` | Runs the inference workflow to match a list of company names against an existing reference list. |

---

## How It Works

1. **Training Data Generation**
   - Takes a list of known company names and automatically creates:
     - Similar pairs (representing the same entity)
     - Dissimilar pairs (representing different entities with realistic overlaps)
   - Special emphasis is placed on:
     - Corporate suffixes (`LLC`, `Inc`, `Ltd`, `Co.`)
     - Prefix variations
     - Entity-type semantics (e.g., *Capital*, *Logistics*, *Ventures*)

2. **Model Fine-Tuning**
   - A base multilingual / business-domain sentence-transformer is fine-tuned.
   - The model learns that some words should be weighted *less* (e.g., `LLC`) while others *matter more* (e.g., `Capital` vs `Logistics`).

3. **Semantic Matching Workflow**
   - Each company name is embedded into a vector space.
   - Cosine similarity determines closeness.
   - A similarity threshold is used to classify matches.

---

## Input Data Requirements

Before running any training or matching workflows, you must provide a file named **`companies.xlsx`** in the project’s working directory.

### Expected File Format

| Column Name   | Description |
|---------------|-------------|
| `ID`          | A unique identifier for each company (string or integer). |
| `CompanyName` | The company name as it should be interpreted by the model. |

Example:

| ID  | CompanyName           |
|-----|-----------------------|
| 101 | Acme Holdings LLC     |
| 102 | Blue River Logistics  |
| 103 | Acme Capital Partners |

### Workflow

1. Place your `companies.xlsx` file at the root of the project.
2. Open and run the notebook **`build-training-data.ipynb`**:
   - This script will generate a training dataset containing:
     - Positive match pairs (same company under different naming patterns)
     - Hard negative pairs (similar structure but different entities)
     - Suffix and prefix-aware variations (`Inc`, `LLC`, `Ltd`, etc.)
3. After the training data is generated, run **`fine-tune-model.ipynb`** to create your domain-optimized similarity model.

Once fine-tuning completes, the resulting model can be used in the **semantic matching** notebook to score and compare company names.


---

## Example Usage

```python
result_df = semantic_match_company_lists(
    df_left=companies_to_match,
    df_right=reference_companies,
    left_name_col="CompanyName",
    right_name_col="CompanyName",
    left_id_col="CompanyID",
    right_id_col="CompanyID",
    n_top=3,
    threshold=0.75,
    output_mode="rows",
    model_name="finetuned-company-matcher"
)
