---
layout: post
title: "CSV Surgeon: Automatically Repair Broken CSV Files"
date: 2026-02-11
description: "CSV Surgeon is a CLI tool that automatically detects and repairs encoding issues, delimiter mismatches, malformed quotes, and embedded linebreaks in CSV files."
author: AlphaOfTech
faq:
  - q: "What is CSV Surgeon?"
    a: "CSV Surgeon is a command-line tool that automatically detects and repairs broken CSV files. It handles encoding issues, delimiter mismatches, malformed quotes, and structural problems using statistical analysis and intelligent algorithms."
  - q: "How do I install CSV Surgeon?"
    a: "Install via pip: pip install csv-surgeon. Requires Python 3.8 or later. Works on Linux, macOS, and Windows."
  - q: "Why does broken CSV data matter?"
    a: "Data analysts, journalists, and business users regularly receive CSV exports from multiple sources (APIs, legacy systems, database exports) that have encoding issues, wrong delimiters, or structural problems. Manual fixes are time-consuming and error-prone, while existing tools require technical knowledge or can't be automated."
  - q: "How is CSV Surgeon different from csvkit or OpenRefine?"
    a: "CSV Surgeon automatically detects and fixes issues without requiring you to know what's wrong. csvkit requires technical knowledge and manual diagnosis. OpenRefine is GUI-only and not scriptable. RepairMyCSV works only in browsers. CSV Surgeon works from the command line, requires no configuration for 90% of cases, and can be integrated into automated data pipelines."
---

**TL;DR:** CSV Surgeon is a command-line tool that automatically repairs broken CSV files with encoding issues, delimiter mismatches, malformed quotes, and embedded linebreaks. Install with `pip install csv-surgeon`, run `csv-surgeon repair broken.csv`, and get clean data in seconds.

You download a CSV export from your company's legacy reporting system. You try to open it in Excel. Instead of a clean spreadsheet, you see:

- Gibberish characters where names should be: `Ren√©` instead of `René`
- Quote marks scattered randomly through your data
- Rows that should be single records split across multiple lines
- A delimiter that might be comma, semicolon, or something else entirely

This is the reality for data analysts, journalists, and business users who receive CSV files from multiple sources. Each system exports data slightly differently. Some use semicolons instead of commas. Others export with Latin-1 encoding but your tools expect UTF-8. Legacy systems produce malformed quotes that break parsing.

Existing solutions don't solve this well:

- **csvkit** (13.7k GitHub stars) requires you to know what's wrong and how to fix it. You need to manually specify encodings, delimiters, and repair strategies.
- **OpenRefine** (11k stars) is a powerful GUI tool, but it's not scriptable and requires manual interaction for each file.
- **RepairMyCSV** is web-based, which means you can't automate it or integrate it into data pipelines.
- **Excel and Google Sheets** force you to manually find and fix each issue.

I built CSV Surgeon to solve this: **one command that automatically detects and repairs everything.**

## What is CSV Surgeon?

CSV Surgeon is a CLI tool that uses statistical analysis and intelligent algorithms to:

1. **Detect file encoding** using multi-pass analysis with confidence scoring (UTF-8, UTF-16, Latin-1, CP1252)
2. **Infer the delimiter** by analyzing coefficient of variation across candidates (comma, semicolon, tab, pipe)
3. **Repair malformed quotes** using a state machine parser with context-aware pattern matching
4. **Reconstruct split records** by calculating column count consistency and merging lines until structure is restored
5. **Validate output** and provide detailed diagnostics with confidence scores

Here's the simplest usage:

```bash
pip install csv-surgeon
csv-surgeon repair broken.csv
```

The tool analyzes your file, detects issues automatically, repairs them, and outputs clean CSV to stdout:

```
Detected encoding: ISO-8859-1 (confidence: 0.89)
Detected delimiter: ; (confidence: 0.95)
Repaired 3 malformed quote pairs
Reconstructed 2 records with embedded linebreaks
Repaired 127 rows successfully

id;name;email;address
1;John Doe;john@example.com;123 Main St
2;Jane Smith;jane@example.com;456 Oak Ave
```

## Why does automatic CSV repair matter?

Every data pipeline starts with data ingestion. When your CSV files are broken, you have three options:

1. **Manually fix each file** — time-consuming and error-prone
2. **Write custom parsing code** — brittle and hard to maintain
3. **Use a tool that requires configuration** — you need to diagnose the problem first

CSV Surgeon eliminates these steps. It's designed for the 90% case where you just need the file to work. You don't need to know whether your encoding is ISO-8859-1 or CP1252. You don't need to count delimiters manually to figure out if it's semicolons or commas.

The tool uses **statistical delimiter inference**: it analyzes coefficient of variation (CV = standard deviation / mean) for each candidate delimiter across all lines. The delimiter with the lowest CV is most likely correct. This approach works even when data contains multiple special characters.

For encoding detection, CSV Surgeon uses **multi-pass detection with chardet** and a fallback chain: UTF-8 → UTF-16 → Latin-1 → CP1252. Each detection includes a confidence score, so you can see how certain the tool is about its choice.

## Real-world examples

### Analyze before repairing

If you want to see what's wrong before making changes:

```bash
csv-surgeon analyze messy.csv
```

Output:

```
File: messy.csv (234 KB, 1,523 lines)

Encoding Analysis:
  Detected: UTF-8 with errors (confidence: 0.73)
  Issues: 12 invalid byte sequences at lines: 45, 89, 234...

Delimiter Analysis:
  Comma: CV=0.12 (likely correct)
  Semicolon: CV=0.87
  Tab: CV=2.34

Structure Issues:
  Expected columns: 7
  Inconsistent rows: 23 (lines: 34, 56, 78...)
  Unclosed quotes: 5 pairs
  Embedded linebreaks: 8 records

Recommendation: Run with --force-encoding utf-8 and default repair
```

### Repair with output file

```bash
csv-surgeon repair data.csv -o cleaned.csv --encoding utf-8
```

This forces UTF-8 output encoding even if the input uses something else.

### PII sanitization during repair

```bash
csv-surgeon repair input.csv --delimiter , --sanitize-pii
```

Output:

```
Using forced delimiter: ,
Detected encoding: UTF-8 (confidence: 0.99)
Repaired 12 rows successfully
Sanitized PII: 34 emails, 12 phone numbers

id,name,email,phone
1,John Doe,[REDACTED_EMAIL],[REDACTED_PHONE]
2,Jane Smith,[REDACTED_EMAIL],[REDACTED_PHONE]
```

This is useful when you need to share sample data without exposing personal information.

## How CSV Surgeon works

The repair process runs in five stages:

1. **Encoding Detection**: Multi-pass detection with chardet, using a fallback chain and confidence scoring
2. **Delimiter Inference**: Statistical analysis of coefficient of variation across candidate delimiters
3. **Quote Repair**: State machine parser that tracks quote context and repairs using delimiter-aware heuristics
4. **Record Reconstruction**: For files with embedded linebreaks, calculates mode of column counts and merges rows until consistency is achieved
5. **Validation**: Verifies column count consistency and data type coherence across all rows

Each stage produces confidence scores and diagnostic information. If a stage fails with high confidence, the tool reports the issue rather than making guesses.

## Stream processing for large files

CSV Surgeon handles files larger than 1GB using chunk-based reading. It doesn't load the entire file into memory. This makes it suitable for production data pipelines where file sizes are unpredictable.

```bash
csv-surgeon repair huge_export.csv -o cleaned.csv
```

The tool processes data incrementally, maintaining state for quote repair and record reconstruction across chunks.

## Frequently Asked Questions

### What is CSV Surgeon?

CSV Surgeon is a command-line tool that automatically detects and repairs broken CSV files. It handles encoding issues, delimiter mismatches, malformed quotes, and structural problems using statistical analysis and intelligent algorithms.

### How do I install CSV Surgeon?

Install via pip:

```bash
pip install csv-surgeon
```

Requires Python 3.8 or later. Works on Linux, macOS, and Windows.

### Why does broken CSV data matter?

Data analysts, journalists, and business users regularly receive CSV exports from multiple sources (APIs, legacy systems, database exports) that have encoding issues, wrong delimiters, or structural problems. Manual fixes are time-consuming and error-prone, while existing tools require technical knowledge or can't be automated.

### How is CSV Surgeon different from csvkit or OpenRefine?

CSV Surgeon automatically detects and fixes issues without requiring you to know what's wrong. csvkit requires technical knowledge and manual diagnosis. OpenRefine is GUI-only and not scriptable. RepairMyCSV works only in browsers. CSV Surgeon works from the command line, requires no configuration for 90% of cases, and can be integrated into automated data pipelines.

---

CSV Surgeon is open source (MIT License) and available on GitHub: [https://github.com/Intellirim/csv-surgeon](https://github.com/Intellirim/csv-surgeon)

Try it out:

```bash
pip install csv-surgeon
csv-surgeon repair your_broken_file.csv
```

Star the repo if you find it useful, or open an issue if you encounter edge cases. Contributions welcome.