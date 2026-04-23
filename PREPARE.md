# Evaluation Setup

This file is outside the editable surface. It defines how results are judged. Agents cannot modify the evaluator or the scoring logic — the evaluation is the trust boundary.

Consider defining more than one evaluation criterion. Optimizing for a single number makes it easy to overfit and silently break other things. A secondary metric or sanity check helps keep the process honest.

eval_cores: 1
eval_memory_gb: 1.0
prereq_command: npm run build

## Setup

```bash
npm install
npm run build
```

The project requires Node.js and npm. Install dependencies with `npm install`, which will also run the prepare script (husky install + build). The TypeScript source in `src/index.ts` is compiled to `dist/index.js` for benchmarking.

## Run command

```bash
npm run bench 2>&1 | awk '/· parsing paths/ {parse=$4; gsub(/,/, "", parse)} /· building regexp/ {build=$4; gsub(/,/, "", build)} /· static path/ {static=$4; gsub(/,/, "", static)} /· simple path/ {simple=$4; gsub(/,/, "", simple)} /· compiling paths/ {compile=$4; gsub(/,/, "", compile)} END {if (parse && build && static && simple && compile) {metric = (parse + build + static + simple + compile) / 5; printf "METRIC=%.2f\n", metric} else {print "METRIC=0.0"}}'
```

## Output format

The benchmark must print `METRIC=<number>` to stdout. The metric is the geometric mean of operations per second (hz) across five key benchmark suites: parsing paths, building regexp, static path matching, simple path matching, and compiling paths.

## Metric parsing

The CLI looks for `METRIC=<number>` in the output. Higher values indicate better performance.

## Ground truth

The baseline metric represents the average performance across the core operations of the path-to-regexp library measured on the current codebase (v8.4.2). The benchmark includes:

- **parsing paths**: Tokenizing path strings like `/user/:id` into structured tokens
- **building regexp**: Converting parsed tokens into RegExp objects with capture groups
- **static path**: Matching fixed paths like `/user` (fastest case)
- **simple path**: Matching parameterized paths like `/user/:id` (common case)
- **compiling paths**: Building path generation functions from templates

The metric is calculated as the arithmetic mean of the hz (operations per second) values from these five benchmarks. This composite score provides a balanced view of library performance across its primary use cases.

Additional benchmarks exist (multi segment, multi pattern, tricky pattern, asterisk) but are not included in the primary metric as they represent edge cases. All test suites must continue to pass, including the ReDoS protection tests which verify security against catastrophic backtracking.
