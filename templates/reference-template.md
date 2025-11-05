# [Component/API/Configuration Name] Reference

> **Document Type**: Reference (Information-Oriented)  
> **Category**: [API/Configuration/Architecture/CLI/etc.]  
> **Version**: [Version number]  
> **Last Updated**: [Date]

## Overview

[1-2 sentence neutral description of what this reference documents]

**Quick Facts**:
- **Type**: [Component type]
- **Location**: [Where it exists in the system]
- **Status**: [Stable/Beta/Deprecated]
- **Since Version**: [X.X]

---

## Synopsis / Quick Reference

[For APIs/CLIs: Show basic usage syntax]
[For configurations: Show basic structure]

```[language]
# Basic usage pattern
[show syntax or structure]
```

---

## Description

[Comprehensive, neutral description of the component/API/configuration. Be thorough and precise.]

### Key Characteristics
- [Characteristic 1]
- [Characteristic 2]
- [Characteristic 3]

---

## [For APIs: Endpoints / For CLIs: Commands / For Configs: Parameters]

### [Endpoint/Command/Parameter Name 1]

**Signature/Syntax**:
```[language]
[Exact syntax]
```

**Description**: [What it does]

**Parameters**:

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `param1` | string | Yes | - | [Description] |
| `param2` | integer | No | 0 | [Description] |
| `param3` | boolean | No | false | [Description] |

**Returns**: [Return type and description]

**Example**:
```[language]
# Example usage
[realistic example with context]
```

**Response** (if applicable):
```[language]
{
  "field1": "value",
  "field2": 123
}
```

**Possible Errors**:
- `ERROR_CODE_1`: [When this occurs and what it means]
- `ERROR_CODE_2`: [When this occurs and what it means]

---

### [Endpoint/Command/Parameter Name 2]

[Repeat structure for each item]

---

## Configuration Options (if applicable)

### [Configuration Section 1]

| Option | Type | Required | Default | Description | Valid Values |
|--------|------|----------|---------|-------------|--------------|
| `option1` | string | Yes | - | [Description] | [Valid values or range] |
| `option2` | integer | No | 100 | [Description] | 1-1000 |

**Example Configuration**:
```[format]
[Show example configuration]
```

---

## Data Types / Schemas (if applicable)

### [Type Name 1]

**Structure**:
```[language]
{
  "field1": "type",
  "field2": "type",
  "nestedObject": {
    "subfield1": "type"
  }
}
```

**Field Descriptions**:
- **field1** (`type`): [Description]
- **field2** (`type`): [Description]
- **nestedObject** (`object`): [Description]
  - **subfield1** (`type`): [Description]

**Validation Rules**:
- [Rule 1]
- [Rule 2]

---

## Constants / Enumerations (if applicable)

### [Enum Name]

| Value | Description | Use Case |
|-------|-------------|----------|
| `VALUE_1` | [Description] | [When to use] |
| `VALUE_2` | [Description] | [When to use] |

---

## Behavior / Constraints

### [Behavior Aspect 1]
[Detailed description of specific behavior]

### [Behavior Aspect 2]
[Description of limitations, constraints, or special cases]

### Performance Considerations
- [Performance characteristic 1]
- [Performance characteristic 2]

### Limits
- [Limit 1]
- [Limit 2]

---

## Examples

### Example 1: [Common Use Case]

[Brief context]

```[language]
# Complete working example
[code]
```

**Output**:
```
[Expected output]
```

---

### Example 2: [Another Use Case]

[Brief context]

```[language]
# Complete working example
[code]
```

---

## Error Reference (if applicable)

### [ERROR_CODE_1]

**Message**: "[Error message text]"  
**Cause**: [What causes this error]  
**Resolution**: [How to fix it]

### [ERROR_CODE_2]

[Repeat pattern]

---

## Version History

### Version [X.X]
- [Change 1]
- [Change 2]

### Version [X.Y]
- [Change 1]

---

## Deprecation Notice (if applicable)

> **⚠️ Deprecated**: This [component/endpoint/parameter] is deprecated as of version [X.X] and will be removed in version [Y.Y].
> 
> **Migration Path**: Use [alternative] instead. See [link to migration guide].

---

## Related References

- [Link to related reference doc 1]
- [Link to related reference doc 2]

## See Also

- **Tutorial**: [Link to tutorial using this component]
- **How-To**: [Link to practical guide]
- **Explanation**: [Link to conceptual explanation]

---

**Maintained By**: [Team/Person]  
**Last Reviewed**: [Date]  
**Version**: [X.X]
