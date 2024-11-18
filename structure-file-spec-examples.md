# Structure File Examples, Validation Rules, and Hook System

## Extended Examples

### Pattern Examples

#### API Endpoint Pattern
```yaml
# .structure/patterns/api-endpoint.yaml
pattern-name: "REST API Endpoint"
version: "1.0.0"
description: "Standard REST endpoint implementation pattern"
applies-to:
  - "/api/**/*.ts"
  - "/api/**/*.js"
rules:
  - name: "Authentication"
    description: "All endpoints must use authentication middleware"
    reference: "/api/middleware/auth.ts"
    validation:
      required-imports: ["@/middleware/auth"]
      required-patterns: ["authMiddleware("]
  
  - name: "Request Validation"
    description: "All input must be validated using Zod schemas"
    reference: "/api/schemas/validation.ts"
    validation:
      required-imports: ["zod"]
      file-patterns:
        - "export const schema = z."
        - "validate(req.body, schema)"

  - name: "Error Handling"
    description: "Must use standard error response format"
    reference: "/api/utils/errors.ts"
    validation:
      required-imports: ["@/utils/errors"]
      required-patterns: ["handleError("]

examples:
  - path: "/api/v1/users/create.ts"
    purpose: "Reference implementation"
    highlights:
      - line: 12
        description: "Authentication middleware usage"
      - line: 15
        description: "Schema validation"
```

#### React Component Pattern
```yaml
# .structure/patterns/react-component.yaml
pattern-name: "React Component"
version: "1.0.0"
description: "Standard React component structure"
applies-to:
  - "/src/components/**/*.tsx"
  - "/src/features/**/*.tsx"
rules:
  - name: "Component Structure"
    description: "Standard component organization"
    template: |
      import React from 'react'
      import styles from './Component.module.css'
      
      interface Props {
        // Props definition
      }
      
      export const Component: React.FC<Props> = () => {
        return (
          // JSX
        )
      }
    validation:
      required-patterns:
        - "interface Props"
        - "React.FC<Props>"

  - name: "Styling"
    description: "CSS modules for styling"
    validation:
      required-imports: [".module.css$"]
      forbidden-patterns: ["style={{"]

  - name: "Testing"
    description: "Must have accompanying test file"
    validation:
      required-files: ["${filename}.test.tsx"]

examples:
  - path: "/src/components/Button/Button.tsx"
    purpose: "Basic component example"
  - path: "/src/features/UserProfile/UserProfile.tsx"
    purpose: "Complex feature component"
```

#### Database Model Pattern
```yaml
# .structure/patterns/database-model.yaml
pattern-name: "Database Model"
version: "1.0.0"
description: "Prisma model with standard patterns"
applies-to:
  - "/src/models/**/*.prisma"
rules:
  - name: "Timestamps"
    description: "All models must include created/updated timestamps"
    validation:
      required-fields:
        - "createdAt DateTime @default(now())"
        - "updatedAt DateTime @updatedAt"

  - name: "Soft Delete"
    description: "Support soft deletion"
    validation:
      required-fields:
        - "deletedAt DateTime?"

  - name: "Auditing"
    description: "Track who created/updated records"
    validation:
      required-fields:
        - "createdBy String?"
        - "updatedBy String?"

examples:
  - path: "/src/models/user.prisma"
    purpose: "Standard user model"
```

### Template Examples

#### API Service Template
```yaml
# .structure/templates/api-service.yaml
template-name: "API Service"
version: "1.0.0"
description: "Complete API service template"
variables:
  - name: "SERVICE_NAME"
    description: "Name of the service (e.g., 'users')"
    required: true
    validation: "^[a-z][a-z0-9-]*$"
  
  - name: "METHODS"
    description: "HTTP methods to implement"
    type: "array"
    default: ["GET", "POST", "PUT", "DELETE"]
    allowed-values: ["GET", "POST", "PUT", "PATCH", "DELETE"]

  - name: "DATABASE"
    description: "Include database integration"
    type: "boolean"
    default: true

structure:
  - path: "/src/services/${SERVICE_NAME}"
    files:
      - name: "controller.ts"
        template: "templates/service/controller.ts"
        if: "true"
      
      - name: "service.ts"
        template: "templates/service/service.ts"
        if: "true"
      
      - name: "repository.ts"
        template: "templates/service/repository.ts"
        if: "DATABASE"
      
      - name: "schema.ts"
        template: "templates/service/schema.ts"
        if: "true"
      
      - name: "types.ts"
        template: "templates/service/types.ts"
        if: "true"
      
      - name: "__tests__/${SERVICE_NAME}.test.ts"
        template: "templates/service/test.ts"
        if: "true"
      
      - name: "__tests__/${SERVICE_NAME}.e2e.ts"
        template: "templates/service/e2e.ts"
        if: "true"
```

## Detailed Hook System

### Hook Types

#### Pre-Commit Hook
```bash
#!/bin/bash
# .structure/hooks/pre-commit

# Validate structure files
echo "Validating structure files..."
structure-validate ./structure.yaml

# Check pattern compliance
echo "Checking pattern compliance..."
structure-check-patterns

# Verify all referenced paths exist
echo "Verifying paths..."
structure-verify-paths

exit $?
```

#### Post-Update Hook
```bash
#!/bin/bash
# .structure/hooks/post-update

# Regenerate documentation
echo "Updating documentation..."
structure-generate-docs

# Update IDE configuration
echo "Updating IDE config..."
structure-update-ide-config

# Notify team of changes
echo "Notifying team..."
structure-notify-team

exit $?
```

#### Custom Validation Hook
```bash
#!/bin/bash
# .structure/hooks/custom-validate

# Check custom business rules
echo "Checking custom rules..."

# Example: Ensure all API endpoints have documentation
check_api_docs() {
  local apis=$(find ./src/api -name "*.ts")
  for api in $apis; do
    local doc_file="${api/\.ts/.md}"
    if [[ ! -f "$doc_file" ]]; then
      echo "Error: Missing documentation for $api"
      return 1
    fi
  done
}

check_api_docs
exit $?
```

### Hook Configuration

```yaml
# .structure/config.yaml
hooks:
  enabled: true
  timeout: 30
  environment:
    NODE_ENV: "development"
    PATH: "/usr/local/bin:$PATH"
  
  pre-commit:
    enabled: true
    parallel: false
    ignore-errors: false
  
  post-update:
    enabled: true
    parallel: true
    ignore-errors: true
    
  custom-validate:
    enabled: true
    timeout: 60
    cwd: "./src"
```

## Validation Rules

### Path Validation

```yaml
# Example validation rules for paths
validation:
  paths:
    # Ensure paths exist
    must-exist: true
    
    # Allow patterns in paths
    allow-patterns: true
    
    # Path naming conventions
    naming:
      pattern: "^[a-z0-9-_/]+$"
      max-length: 100
      
    # Directory structure rules
    directories:
      max-depth: 5
      required:
        - "/src"
        - "/tests"
      forbidden:
        - "/temp"
        - "/old"
```

### Pattern Validation

```yaml
# Example validation rules for patterns
validation:
  patterns:
    # Pattern naming
    naming:
      pattern: "^[A-Z][a-zA-Z0-9-]+$"
      max-length: 50
    
    # Pattern versioning
    versioning:
      required: true
      format: "semver"
    
    # Pattern dependencies
    dependencies:
      circular: false
      max-depth: 3
    
    # Pattern applicability
    applies-to:
      required: true
      min-paths: 1
      valid-patterns:
        - "**/*.ts"
        - "**/*.tsx"
        - "**/*.js"
        - "**/*.jsx"
```

### Template Validation

```yaml
# Example validation rules for templates
validation:
  templates:
    # Template naming
    naming:
      pattern: "^[A-Z][a-zA-Z0-9-]+$"
      max-length: 50
    
    # Variable validation
    variables:
      naming:
        pattern: "^[A-Z][A-Z0-9_]+$"
      required:
        - description
        - type
      
    # Template generation
    generation:
      max-files: 50
      max-directory-depth: 5
      forbidden-paths:
        - "/node_modules"
        - "/.git"
```

### Content Validation

```yaml
# Example validation rules for content
validation:
  content:
    # File size limits
    size:
      max: "1MB"
      warn-at: "500KB"
    
    # Content patterns
    patterns:
      required:
        - pattern: "^#!/usr/bin/env"
          files: "**/*.sh"
        - pattern: "Copyright"
          files: "**/*"
      
    # Encoding validation
    encoding:
      required: "UTF-8"
      
    # Line endings
    line-endings:
      style: "LF"
      mixed: false
```
