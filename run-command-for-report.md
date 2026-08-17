## Promptfoo CLI Cheat Sheet
**1. Run a single test file:**
```bash
npx promptfoo eval -c 01-knowledge-grounding.yaml
2. Run multiple specific test files at once:

bash


npx promptfoo eval -c 01-knowledge-grounding.yaml 02-guardrails.yaml 03-accuracy.yaml
3. Run ALL test files in a folder together:

bash


npx promptfoo eval -c 0*.yaml
4. View results in the interactive browser UI (Localhost):

bash


npx promptfoo view
5. Export an HTML Report (Standalone web page):

bash


# Export single test to HTML
npx promptfoo eval -c 03-accuracy.yaml -o accuracy_report.html
# Export ALL tests to a single HTML report
npx promptfoo eval -c 0*.yaml -o full_report.html
6. Export a CSV Report (For Excel / Google Sheets):

bash


npx promptfoo eval -c 03-accuracy.yaml -o results.csv
7. Export a JSON Report (For automation pipelines):

bash


npx promptfoo eval -c 03-accuracy.yaml -o results.json
8. Run tests without caching (Force a fresh run):

bash


npx promptfoo eval -c 01-knowledge-grounding.yaml --no-cache
9. Initialize a brand new Promptfoo project in a folder:

