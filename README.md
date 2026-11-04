# DNA Sequence Matching with A* Search

A Python project exploring whether **A* search can efficiently align DNA sequences** and identify which original sequence a mutated sample came from.

I built this project for **FIT1061: Introduction to Artificial Intelligence**. It adapts A* from a pathfinding problem to DNA edit distance and compares search efficiency, optimality, and origin-matching accuracy.

## The Idea

I start with short reference fragments from the *E. coli* K-12 MG1655 genome and simulate three generations:

- **Set A:** Original reference sequences.
- **Set B:** Sequences created by mutating Set A once.
- **Set C:** Sequences created by mutating Set B again.

After shuffling the B and C samples, the program compares each sample with five possible references from Set A. It predicts the source with the **smallest Levenshtein edit distance** (the fewest substitutions, insertions, and deletions).

The project asks two questions: **Can a better heuristic reduce the work A* does? And does the closest DNA match always identify the true original?**

## How It Works

For two sequences, each A* state `(i, j)` records how many characters have been processed from each one.

The search can move:

- **Diagonally:** Match (cost 0) or substitute (cost 1).
- **Along the first sequence:** Delete a character (cost 1).
- **Along the second sequence:** Insert a character (cost 1).

I compared four search methods:

| Method | Priority / Idea |
|---|---|
| A* — Zero heuristic | `h = 0` (baseline) |
| A* — Length heuristic | Difference in remaining sequence lengths |
| A* — Base-count heuristic | Larger of the remaining-length difference and half the total A/T/C/G count difference (rounded up) |
| Weighted A* | `f = g + 1.5h`, using the base-count heuristic |

The base-count heuristic is a lower bound because one substitution can correct at most two base-count differences.

I also use `suffix_counts()` to prepare the remaining base counts for each position and `make_alignment()` to show gaps in the chosen alignment.

The Python **Levenshtein** library provides an independent exact-distance check. The notebook also tests 12 edge-case pairs and verifies the reconstructed alignments.

## Experiments

The notebook uses 45 reference fragments organised into groups of five, with intended sequence lengths of **30, 60, and 100**.

I tested **5%, 15%, and 30%** mutation probabilities, using substitution-only and mixed substitution/insertion/deletion modes.

Using a fixed random seed of `42`, the notebook generates:

- **54 trials**
- **540 modified DNA samples**
- **2,700 DNA-pair comparisons per search method**

### Experiment 1 — Search Efficiency and Correctness

| Method | Average Nodes Expanded | Average Runtime | Exact Distances |
|---|---:|---:|---:|
| A* — Zero | 3,302.88 | 7.481 ms | 2,700 / 2,700 |
| A* — Length | 1,499.67 | 4.021 ms | 2,700 / 2,700 |
| A* — Base-count | 1,495.71 | 4.919 ms | 2,700 / 2,700 |
| Weighted A* (1.5) | 1,185.90 | 4.256 ms | 2,555 / 2,700 |

**Key findings:**

- Base-count A* expanded **54.72% fewer nodes** than zero A* while preserving the minimum edit distance in every tested comparison.
- Weighted A* expanded **20.71% fewer nodes** than base-count A*, but returned a non-optimal distance in **145 of 2,700 comparisons**.
- Length A* was faster than base-count A* despite expanding slightly more nodes, showing that more complex heuristics can increase runtime.

### Experiment 2 — Finding the Original DNA

The program compared every modified sample with all five candidate references and checked its prediction against the **origin index stored during data generation**.

Results:

- **538 / 540** predicted origins were correct (**99.63%**).
- Generation B: **270 / 270** correct.
- Generation C: **268 / 270** correct.
- The notebook recorded **2 tied (ambiguous) predictions**.

**Key finding:** A* can calculate the correct minimum edit distance and still select the wrong original reference.

After mutations, another reference can become more similar than the one a sample actually came from.

This shows that finding the closest DNA match does not always guarantee correct identification of its origin.

## Files in This Repository

```text
├── README.md
├── H2_comprehensive_project_implementation(7).ipynb
└── d1_helpers.py
```

- **`H2_comprehensive_project_implementation(7).ipynb`** — Contains the DNA simulation, A* and Weighted A* implementations, tests, experiments, visualisations and project report.
- **`d1_helpers.py`** — Helper module provided for the FIT1061 D1 task. My notebook imports its `PriorityQueue` and `_reconstruct` utilities. The DNA-specific search algorithm and heuristics are implemented in my notebook.

## How to Run

1. Download or clone the repository.
2. Keep the notebook and `d1_helpers.py` in the **same folder**.
3. Install Python and Jupyter, along with the required packages:

```bash
pip install jupyter numpy matplotlib python-Levenshtein
```

4. Open `H2_comprehensive_project_implementation(7).ipynb` in Jupyter Notebook.
5. Run the cells in order.

The reference DNA text is embedded in the notebook, so a separate genome download is not required to reproduce the recorded experiments.

## Limitations and Future Work

This is an **algorithm experiment, not a biological ancestry test**.

Sets B and C are generated using simplified Python mutations rather than realistic bacterial reproduction or mutation frequencies.

Each trial uses only five candidate references, many of which are relatively different. This makes identifying the original sequence easier.

Tied minimum distances are recorded, but the current program selects the first tied reference.

**Data-cleaning note:** The notebook extracts fragments from copied FASTA text containing line breaks. Some extracted sequences therefore contain newline characters. Cleaning the input and rerunning the experiments would be an important improvement.

Future improvements could include:

- Cleaning and validating the DNA input.
- Testing more similar original DNA sequences.
- Running experiments with multiple random seeds.
- Comparing A* against a dynamic-programming edit-distance algorithm.
- Reporting tied predictions as ambiguous rather than selecting one automatically.

## Data Source

*E. coli* K-12 MG1655 reference genome.

NCBI accession: **NC_000913.3**

## Acknowledgement

This project was developed as part of FIT1061: Introduction to Artificial Intelligence.

The `d1_helpers.py` module was provided as part of the FIT1061 D1 learning materials. It is not my original work. Its redistribution should follow the university's rules on sharing teaching materials.
