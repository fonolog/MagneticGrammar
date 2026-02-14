# Magnetic Grammar

A Python implementation of the feature-based phonological grammar learner described in D'Alessandro & Van Oostendorp (2020), "Magnetic Grammar", in *Language Variation and Functional Heads* (OUP, pp. 405–439).

## What is Magnetic Grammar?

Magnetic Grammar models phonological systems through **privative features** that can **attract** or **reject** other features within a segment. A language's segment inventory is fully characterised by these attract/reject properties.

- **Attract:** if feature F attracts G, then F is only satisfied when G is also present in the segment.
- **Reject:** if feature F rejects G, then F is only satisfied when G is *absent* from the segment.
- **Learning:** the system observes IPA segments and incrementally builds up the feature grammar, generalising to predict which segments the language should allow.

For example, after learning only /s/ and /p/, the system predicts that /t/ and /f/ must also be valid — because the features needed for those segments are already known and no constraint blocks them.


Requires Python 3.9+.


## Files

| File | Description |
|------|-------------|
| `magnetic_grammar_v2.py` | Core module: the `MagneticGrammar` learner, `IPAInterface`, and symbolic `Attract`/`Reject` properties |
| `Magnetic_Grammar_v2.ipynb` | Interactive Jupyter notebook with examples and experiments |

## Quick start

### As a Python module

```python
from magnetic_grammar_v2 import MagneticGrammar

mg = MagneticGrammar()

# Learn segments from IPA words
mg.learn_word('pat')
mg.learn_word('sat')

# See the predicted segment inventory
for item in mg.predicted_inventory():
    if not item['is_empty']:
        print(f"  {item['ipa']}  {item['feature_names']}")

# Check if a word is grammatical
result = mg.check_word('bad')
print(result['valid'])  # True or False

# Inspect the grammar
for row in mg.grammar_table():
    print(f"{row['feature']}: attracts={row['attracts']}, rejects={row['rejects']}")

# Reset and start over
mg.reset()
```

### As a Jupyter notebook

```bash
jupyter notebook Magnetic_Grammar_v2.ipynb
```

The notebook walks through the paper's examples step by step, showing how the grammar evolves as segments are learned.

## How the learner works

The learning algorithm processes segments one at a time:

1. **Feature extraction.** Each IPA segment is mapped to a set of privative features using [panphon](https://github.com/dmort27/panphon). Only `[+F]` values are treated as present (privative interpretation). A curated subset of 20 phonologically meaningful features is used.

2. **New features get attract properties.** When a segment introduces features not yet in the grammar, each new feature posits `attract(G)` for every previously known feature that co-occurs in the current segment. Features entering the grammar simultaneously (from the same first observation) get no mutual constraints — there is no prior context to form hypotheses against.

3. **Existing features get validated.** For features already in the grammar, any `attract(G)` is removed if G is absent from the current segment (counter-evidence).

4. **Reject emerges from evidence.** After a feature has been observed in 2+ distinct segments, `reject(G)` is added for any known feature G that was *never* present in any of those segments. Reject is removed as soon as the two features co-occur.

### Example

```
Learn /s/ = {Anterior, Consonantal, Continuant, Coronal, Strident}
  → All features are new, no prior features exist
  → Grammar: all features known, no constraints

Learn /p/ = {Anterior, Consonantal, Labial}
  → Labial is new, attracts co-occurring Anterior and Consonantal
  → Grammar: Labial requires Anterior and Consonantal

Predicted inventory: /s/, /p/, /t/, /f/ (and a few others)
  → /t/ works because Coronal has no constraints
  → /f/ works because Labial's attracts are satisfied
```

## API reference

### `MagneticGrammar`

| Method | Description |
|--------|-------------|
| `learn_segment(ipa)` | Learn from a single IPA segment. Returns a trace dict. |
| `learn_word(ipa)` | Learn from an IPA word (auto-segmented). Returns list of traces. |
| `valid_segment(ipa)` | Check if an IPA segment is valid. |
| `check_word(ipa)` | Check a word: returns `{'valid': bool, 'details': [...]}`. |
| `predicted_inventory(basic_only=True)` | Generate all valid segments. Set `basic_only=False` for diacriticked variants. |
| `grammar_table()` | Return the grammar as a list of `{feature, attracts, rejects}` dicts. |
| `reset()` | Clear all learned data. |

### `IPAInterface`

| Method | Description |
|--------|-------------|
| `segment_word(ipa)` | Segment an IPA string into individual segments. |
| `get_features(segment)` | Get the privative feature set for an IPA segment. |
| `lookup_ipa(features)` | Find the IPA symbol for a feature bundle. |

## Feature set

The system uses 20 privative features derived from panphon:

Syllabic, Sonorant, Consonantal, Continuant, Nasal, Lateral, DelayedRelease, Strident, Voice, SpreadGlottis, ConstrGlottis, Anterior, Coronal, Distributed, Labial, High, Low, Back, Round, Tense

## References

D'Alessandro, R. & Van Oostendorp, M. (2020)."; Magnetic Grammar". In L. Franco & P. Lorusso (eds.), *Linguistic Variation: Structure and Interpretation*, pp. 405–439. Berlin: De Gruyter Mouton.

## License

MIT
