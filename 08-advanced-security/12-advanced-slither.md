# Advanced Slither: Custom Detectors

## Simple Definition
While Slither comes with dozens of built-in detectors, complex projects often have unique business logic vulnerabilities. Advanced Slither usage involves writing custom detectors in Python to find project-specific anti-patterns that generic tools miss.

## The Best Analogy
Think of built-in Slither detectors like a **standard metal detector** at an airport: it finds all metal (generic bugs). A custom detector is like a **specialized scanner calibrated to only beep for a specific type of rare coin** (your project's unique vulnerability, like a specific missing access control on a new function type).

## Code Example: Writing a Custom Slither Detector

```python
# custom_detector.py
from slither.detectors.abstract_detector import AbstractDetector, DetectorClassification

class MissingAccessControlOnMint(AbstractDetector):
    """
    Detects if a function named 'mint' or 'burn' lacks an access control modifier.
    """
    ARGUMENT = 'missing-mint-access-control'
    HELP = 'Functions named mint or burn should have access control (e.g., onlyRole)'
    IMPACT = DetectorClassification.HIGH
    CONFIDENCE = DetectorClassification.HIGH

    def _detect(self):
        results = []
        
        for contract in self.compilation_unit.contracts_derived:
            for function in contract.functions:
                # Check if function name is mint or burn
                if function.name in ['mint', 'burn']:
                    # Check if it has modifiers like 'onlyOwner' or 'onlyRole'
                    has_access_control = any(
                        mod.name in ['onlyOwner', 'onlyRole'] 
                        for mod in function.modifiers
                    )
                    
                    if not has_access_control:
                        info = [f"{function.name} in {contract.name} lacks access control\n"]
                        
                        # Add the function node to the result
                        results.append(self.generate_result(info))
                        
        return results
```

## How to Run Custom Detectors

```bash
# 1. Save the Python script in your project directory
# 2. Run Slither and point it to your custom detector
slither . --detect missing-mint-access-control --custom-detector custom_detector.py

# Output:
# missing-mint-access-control (HIGH):
# mint in MyToken lacks access control
# Reference: contracts/MyToken.sol#L15
```

## Integrating Slither into CI/CD (GitHub Actions)

```yaml
# .github/workflows/slither.yml
name: Slither Analysis

on: [push, pull_request]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run Slither
        uses: crytic/slither-action@v0.3.0
        with:
          # Fail the PR if any HIGH or CRITICAL issues are found
          fail-on: high
          slither-args: "--exclude naming-convention,unused-return --custom-detector scripts/custom_detector.py"
```

## Key Takeaways
- **Beyond Defaults:** Built-in detectors catch 80% of issues; custom detectors catch the 20% unique to your business logic.
- **Python-Based:** Slither detectors are written in Python, analyzing the Solidity AST (Abstract Syntax Tree).
- **CI/CD Enforcement:** Block Pull Requests automatically if new custom vulnerabilities are introduced.
- **Reduce False Positives:** Custom detectors can be finely tuned to your codebase, reducing noise compared to generic tools.
- **Open Source:** Share your custom detectors with the community to help others find similar patterns.