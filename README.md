# Structure File Specification v1.0.0

## Table of Contents
- [Introduction](#introduction)
- [Specification](#specification)
- [Schema](#schema)
- [File Format](#file-format)
- [Comments](#comments)
- [Key-Value Pairs](#key-value-pairs)
- [Required Fields](#required-fields)
- [Optional Fields](#optional-fields)
- [File Discovery](#file-discovery)
- [Types](#types)
- [Examples](#examples)

## Introduction

This document serves as the formal specification for structure files, a standardized format for helping AI tools understand and navigate codebases. This is version 1.0.0 of the specification.

A structure file is a configuration file that describes the layout, patterns, and key aspects of a software project. It is designed to be both human-readable and machine-parseable, with a focus on providing context to AI tools.

## Specification

- Structure files MUST be encoded in UTF-8
- Structure files MUST use either YAML or JSON format
- Line endings MAY be either LF or CRLF
- File names MUST be either `structure.yaml`, `structure.yml`, or `structure.json`
- File extension MUST match the format used
- Files MUST be placed in the root directory of the project they describe
- All paths within the file MUST be relative to the file's location
- Version numbers MUST follow semantic versioning

## Schema

A structure file is a document that contains key-value pairs organized in a hierarchical structure. The root level MUST contain all required fields and MAY contain any number of optional fields.

## File Format

Structure files use YAML or JSON because:
1. They are widely supported
2. They have clear specifications
3. They support comments (YAML)
4. They handle nested structures well

### YAML Example
```yaml
project-name: "Example Project"
description: "An example project"
version: "1.0.0"
key-directories:
  - path: /src
    purpose: "Source code"
```

### JSON Example
```json
{
  "project-name": "Example Project",
  "description": "An example project",
  "version": "1.0.0",
  "key-directories": [
    {
      "path": "/src",
      "purpose": "Source code"
    }
  ]
}
```

## Comments

Comments are supported in YAML format using the `#` character. JSON format does not support comments.

```yaml
# This is a top-level comment
project-name: "Example Project"  # This is an inline comment
```

## Key-Value Pairs

- Keys MUST contain only ASCII letters, digits, and hyphens
- Keys MUST start with a letter
- Keys MUST be case-sensitive
- Keys MUST be unique within their table or array
- Values MUST be of the specified type for each field

## Required Fields

The following fields MUST be present in every structure file:

### project-name
- Type: String
- Description: The name of the project
- Constraints: MUST be non-empty

### description
- Type: String
- Description: A brief description of the project
- Constraints: MUST be non-empty

### version
- Type: String
- Description: The version of the project
- Constraints: MUST follow semantic versioning (MAJOR.MINOR.PATCH)

### key-directories
- Type: Array of Directory Objects
- Description: List of important directories in the project
- Constraints: MUST contain at least one directory

#### Directory Object
- Type: Object
- Required Fields:
  - `path`: String (relative path to directory)
  - `purpose`: String (description of directory's purpose)
- Optional Fields:
  - `patterns`: Array of strings
  - `key_files`: Array of File Objects

## Optional Fields

The following fields MAY be present:

### maintainers
- Type: Array of Maintainer Objects
- Description: List of project maintainers

#### Maintainer Object
- Type: Object
- Required Fields:
  - `name`: String
- Optional Fields:
  - `github`: String
  - `email`: String

### dependencies
- Type: Array of Dependency Objects
- Description: List of project dependencies

#### Dependency Object
- Type: Object
- Required Fields:
  - `library`: String
  - `version`: String
- Optional Fields:
  - `usage`: String

### prompts
- Type: Array of Prompt Objects
- Description: Guidance for AI tools

#### Prompt Object
- Type: Object
- Required Fields:
  - `context`: String
  - `instruction`: String
- Optional Fields:
  - `validation`: String

### security
- Type: Object
- Description: Security-related configurations

#### Security Object Fields
- Optional Fields:
  - `required-reviews`: Integer
  - `restricted-paths`: Array of Restricted Path Objects

### integrations
- Type: Object
- Description: Integration configurations

### metrics
- Type: Object
- Description: Project metrics and tracking

## File Discovery

### Basic Discovery
1. Tools MUST look for structure files in the following order:
   - `structure.yaml`
   - `structure.yml`
   - `structure.json`

2. Tools MUST only use the first file found

### Dotfile Directory

Projects MAY use a `.structure` directory in the root project folder for enhanced configuration. When present, this directory follows these rules:

#### Directory Structure
```
.structure/
├── structure.yaml      # Main structure file
├── patterns/          # Pattern definitions
│   ├── api.yaml
│   └── frontend.yaml
├── templates/         # Template definitions
│   ├── endpoint.yaml
│   └── component.yaml
└── hooks/            # Custom behavior hooks
    ├── pre-commit
    └── post-update
```

#### Required Structure
- The `.structure` directory MUST contain a main structure file if used
- The main structure file MUST follow all rules from the base specification

#### Optional Directories

##### patterns/
- Contains YAML files defining reusable patterns
- Each file MUST describe a single pattern or related set of patterns
- Files MUST use `.yaml` or `.yml` extension
- Pattern files MUST follow this format:

```yaml
pattern-name: "API Endpoint Pattern"
version: "1.0.0"
description: "Standard REST endpoint implementation"
applies-to:
  - "/api/**/*.ts"
  - "/api/**/*.js"
rules:
  - name: "Authentication"
    description: "All endpoints must use authentication middleware"
    reference: "/api/middleware/auth.ts"
  - name: "Validation"
    description: "All input must be validated"
    reference: "/api/middleware/validate.ts"
examples:
  - path: "/api/v1/users/create.ts"
    purpose: "Reference implementation"
```

##### templates/
- Contains YAML files defining project templates
- Each file MUST describe a single template
- Template files MUST follow this format:

```yaml
template-name: "REST Endpoint"
version: "1.0.0"
description: "Template for new REST endpoints"
variables:
  - name: "RESOURCE_NAME"
    description: "Name of the resource (e.g., 'users')"
    required: true
  - name: "METHODS"
    description: "HTTP methods to implement"
    type: "array"
    default: ["GET", "POST"]
structure:
  - path: "/api/v1/${RESOURCE_NAME}"
    files:
      - name: "controller.ts"
        template: "templates/endpoint/controller.ts"
      - name: "service.ts"
        template: "templates/endpoint/service.ts"
      - name: "test.ts"
        template: "templates/endpoint/test.ts"
```

##### hooks/
- Contains executable scripts that run at specified points
- Scripts MUST be executable
- Scripts MUST return 0 for success, non-zero for failure
- Standard hooks include:
  - `pre-commit`: Run before structure file changes are committed
  - `post-update`: Run after structure file is updated
  - `pre-validate`: Run before structure validation
  - `post-validate`: Run after structure validation

#### Configuration

The `.structure` directory MAY contain a `config.yaml` file for directory-specific settings:

```yaml
version: "1.0.0"
settings:
  strict-mode: true     # Enforce all optional rules
  validate-paths: true  # Verify all paths exist
  hooks:
    enabled: true      # Enable hook execution
    timeout: 30        # Maximum hook execution time (seconds)
patterns:
  enabled:             # Explicitly enable specific patterns
    - "api.yaml"
    - "frontend.yaml"
  ignore:              # Patterns to ignore
    - "legacy.yaml"
templates:
  path: "./templates"  # Custom template path
  variables:           # Global template variables
    COMPANY: "Acme Inc"
    LICENSE: "MIT"
```

#### Extension Points

The `.structure` directory provides several extension points:

1. **Pattern Extensions**
```yaml
# patterns/custom.yaml
extends: "api.yaml"    # Base pattern to extend
override:              # Rules to override
  - name: "Authentication"
    description: "Use OAuth instead of JWT"
    reference: "/api/middleware/oauth.ts"
add:                   # New rules to add
  - name: "Logging"
    description: "Add structured logging"
    reference: "/api/middleware/logger.ts"
```

2. **Template Extensions**
```yaml
# templates/custom-endpoint.yaml
extends: "endpoint.yaml"
variables:
  add:
    - name: "TEAM"
      description: "Owning team"
      required: true
structure:
  add:
    - path: "/api/v1/${RESOURCE_NAME}/docs"
      files:
        - name: "readme.md"
          template: "templates/docs/readme.md"
```

#### Integration

Structure files in the root directory MUST reference the `.structure` directory if used:

```yaml
project-name: "Example Project"
description: "Project using .structure directory"
version: "1.0.0"
key-directories:
  - path: /src
    purpose: "Source code"
structure-config:
  path: ".structure"
  patterns:
    - "api"
    - "frontend"
  templates:
    - "endpoint"
    - "component"
```

#### Validation Rules

1. If `.structure` directory exists:
   - It MUST contain a valid main structure file
   - All referenced patterns MUST exist
   - All referenced templates MUST exist
   - All hooks MUST be executable

2. Pattern files MUST:
   - Have unique pattern names
   - Contain valid path patterns
   - Reference existing files

3. Template files MUST:
   - Have unique template names
   - Define all required variables
   - Contain valid path templates
   - Reference existing template files

4. Hooks MUST:
   - Return valid exit codes
   - Complete within configured timeout
   - Be executable by the current user

## Types

### String
- MUST be UTF-8 encoded
- MAY be empty unless otherwise specified
- MAY use single or double quotes in YAML
- MUST use double quotes in JSON

### Integer
- MUST be whole numbers
- MAY be positive or negative
- MUST NOT include decimal points

### Boolean
- MUST be `true` or `false` in lowercase

### Array
- MAY be empty unless otherwise specified
- MUST contain elements of the same type
- MUST use consistent indentation in YAML

### Object
- MAY contain any number of key-value pairs
- MUST contain all required fields for its type
- MAY contain optional fields

### Path
- Type: String
- MUST be relative to structure file location
- MUST use forward slashes (/) as separators
- MUST NOT end with a slash
- MUST start with a slash for absolute paths from project root

## Examples

### Minimal Valid Structure File
```yaml
project-name: "Example Project"
description: "A minimal example"
version: "1.0.0"
key-directories:
  - path: /src
    purpose: "Source code"
```

### Complete Example
```yaml
project-name: "Complex Project"
description: "A full-featured example"
version: "2.1.0"
key-directories:
  - path: /src
    purpose: "Source code"
    patterns:
      - "All code follows ESLint config"
    key_files:
      - path: /src/main.ts
        purpose: "Application entry point"
  - path: /tests
    purpose: "Test suite"
maintainers:
  - name: "Adam Whitlock"
    github: "@alloydwhitlock"
    email: "whitlock@example.com"
dependencies:
  - library: "typescript"
    version: "4.9.0"
    usage: "Development language"
prompts:
  - context: "new-feature"
    instruction: "Follow patterns in /src/features"
    validation: "Must include tests"
security:
  required-reviews: 2
  restricted-paths:
    - path: /security
      teams: ["security-team"]
integrations:
  documentation:
    tool: "TypeDoc"
    path: "/docs"
metrics:
  tracking-since: "2024-01-01"
  contributors: 15
```

This specification is versioned. Changes that break backward compatibility MUST increment the major version number. New features that don't break backward compatibility MUST increment the minor version number. Bug fixes and clarifications MUST increment the patch version number.
