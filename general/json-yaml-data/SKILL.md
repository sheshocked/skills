---
name: json-yaml-data
description: 
category: general
tags: [json-yaml-data]
---

## When to Use
Work with data formats: JSON/YAML/TOML parsing, validation, transformation, schema.

## jq Transformations
```bash
# Extract field
echo '{"name":"test","value":42}' | jq '.name'

# Filter array
echo '[{"id":1},{"id":2}]' | jq '.[] | select(.id > 1)'

# Transform
echo '{"items":[1,2,3]}' | jq '.items | map(. * 2)'

# Merge objects
jq -s '.[0] * .[1]' file1.json file2.json
```

## Python Schema Validation
```python
from pydantic import BaseModel, validator

class Config(BaseModel):
    host: str = "0.0.0.0"
    port: int = 8080
    debug: bool = False

    @validator('port')
    def port_range(cls, v):
        if not 1 <= v <= 65535:
            raise ValueError('port must be 1-65535')
        return v

# Validate
config = Config(**raw_dict)
```

## YAML Tips
```yaml
# Multi-line strings
literal: |
  Line 1
  Line 2
folded: >
  This is a very long
  line that folds

# Anchors & aliases
defaults: &defaults
  timeout: 30
  retries: 3

production:
  <<: *defaults
  host: prod.example.com
```

## Pitfalls
- **JSON trailing commas**: Not valid JSON (but many parsers accept)
- **YAML indentation**: Use 2 spaces consistently
- **TOML vs YAML**: TOML for config, YAML for data
- **Schema first**: Define schema before writing data

## Verification
- Validate with jsonschema / pydantic
- Check YAML syntax with yamllint
- Verify TOML with toml CLI