---
name: json-yaml
description: Process JSON, YAML, CSV, and XML data (jq, yq, awk).
tags: [json, yaml, data, cross-platform]
---
# JSON / YAML / Data Processing

Transform and query structured data using `jq`, `yq`, and standard Unix tools.

## JSON with jq

Pretty print:
```
exec: cat /path/to/data.json | jq .
```

Extract field:
```
exec: cat /path/to/data.json | jq '.field.nested'
```

Extract from array:
```
exec: cat /path/to/data.json | jq '.[0].name'
```

Filter array:
```
exec: cat /path/to/data.json | jq '[.items[] | select(.status == "active")]'
```

Map / transform:
```
exec: cat /path/to/data.json | jq '[.users[] | {name: .name, email: .email}]'
```

Count items:
```
exec: cat /path/to/data.json | jq '.items | length'
```

Sort by field:
```
exec: cat /path/to/data.json | jq '.items | sort_by(.date) | reverse'
```

Group by:
```
exec: cat /path/to/data.json | jq '[.items[] | {key: .category, value: .}] | group_by(.key)'
```

Merge objects:
```
exec: jq -s '.[0] * .[1]' file1.json file2.json
```

Modify value in place:
```
exec: jq '.version = "2.0.0"' /path/to/package.json > /tmp/tmp.json && mv /tmp/tmp.json /path/to/package.json
```

JSON to CSV:
```
exec: cat /path/to/data.json | jq -r '.[] | [.name, .email, .age] | @csv'
```

## YAML with yq

Read field:
```
exec: yq '.metadata.name' /path/to/config.yaml
```

Set field:
```
exec: yq -i '.spec.replicas = 3' /path/to/deployment.yaml
```

Convert YAML to JSON:
```
exec: yq -o=json '.' /path/to/config.yaml
```

Convert JSON to YAML:
```
exec: yq -P '.' /path/to/data.json
```

Merge YAML files:
```
exec: yq eval-all '. as $item ireduce({}; . * $item)' base.yaml override.yaml
```

List array items:
```
exec: yq '.items[].name' /path/to/config.yaml
```

## CSV Processing

View CSV with column alignment:
```
exec: column -t -s',' /path/to/data.csv | head -20
```

Extract specific column (awk):
```
exec: awk -F',' '{print $1, $3}' /path/to/data.csv
```

Filter rows:
```
exec: awk -F',' '$3 > 100 {print $0}' /path/to/data.csv
```

Count unique values in a column:
```
exec: awk -F',' '{print $2}' /path/to/data.csv | sort | uniq -c | sort -rn
```

CSV to JSON (with jq):
```
exec: python3 -c "import csv, json, sys; r=csv.DictReader(open('$FILE')); print(json.dumps(list(r), indent=2))"
```

## XML (with xmllint or xq)

Format XML:
```
exec: xmllint --format /path/to/data.xml
```

XPath query:
```
exec: xmllint --xpath '//element/@attr' /path/to/data.xml
```

XML to JSON (with yq):
```
exec: yq -p=xml -o=json '.' /path/to/data.xml
```

## TOML (with yq v4+)

Read TOML:
```
exec: yq -p=toml '.' /path/to/config.toml
```

TOML to JSON:
```
exec: yq -p=toml -o=json '.' /path/to/config.toml
```

## Inline JSON from String

Parse inline:
```
exec: echo '{"name":"test","value":42}' | jq '.name'
```

Build JSON:
```
exec: jq -n --arg name "test" --arg val "42" '{name: $name, value: ($val | tonumber)}'
```

## Notes

- `jq` is usually pre-installed on macOS; install with `apt install jq` (Linux) or `brew install jq`.
- `yq` (Mike Farah's version) handles YAML/JSON/XML/TOML. Install: `brew install yq` or `snap install yq`.
- `xmllint` is part of `libxml2-utils` on Linux, pre-installed on macOS.
- All tools work cross-platform (macOS, Linux, Windows WSL).
- For very large files, consider streaming with `jq --stream` or `yq --stream`.
