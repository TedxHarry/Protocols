# JSON MASTERY - DAY 5: ADVANCED PATTERNS & MASTERY PROJECT

## Welcome to Day 5 - The Final Day!

Today you'll master advanced JSON concepts and build a production-grade automation system!

**Today's Goal:** Achieve complete JSON mastery and build a real-world project

---

## Table of Contents
- [Morning Session: Advanced Concepts](#morning-session-advanced-concepts)
  - [Part 1: JSON Schema Validation](#part-1-json-schema-validation-45-minutes)
  - [Part 2: Complex Nested Patterns](#part-2-complex-nested-patterns-45-minutes)
  - [Part 3: Performance Optimization](#part-3-performance-optimization-30-minutes)
- [Afternoon Session: Production Best Practices](#afternoon-session-production-best-practices)
  - [Part 4: Error Handling Strategies](#part-4-error-handling-strategies-60-minutes)
  - [Part 5: Testing & Validation](#part-5-testing--validation-60-minutes)
- [Evening Session: Capstone Project](#evening-session-capstone-project)
  - [Part 6: Build Complete Automation System](#part-6-build-complete-automation-system-120-minutes)
- [JSON Mastery Certification](#json-mastery-certification)

---

## Morning Session: Advanced Concepts

### Part 1: JSON Schema Validation (45 minutes)

**JSON Schema = Rules that define valid JSON structure**

Think of it as a "blueprint" or "template" that JSON must follow.

---

#### What is JSON Schema?

**Example: Person Schema**
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["name", "age"],
  "properties": {
    "name": {
      "type": "string",
      "minLength": 1
    },
    "age": {
      "type": "integer",
      "minimum": 0,
      "maximum": 150
    },
    "email": {
      "type": "string",
      "format": "email"
    }
  }
}
```

**What this schema enforces:**
- Must be an object
- Must have "name" and "age" (required)
- name must be string with at least 1 character
- age must be integer between 0-150
- email (if present) must be valid email format

---

#### Valid vs Invalid Data

**✓ Valid JSON (matches schema):**
```json
{
  "name": "John Smith",
  "age": 30,
  "email": "john@company.com"
}
```

**✗ Invalid JSON:**
```json
{
  "name": "",           // Too short
  "age": 200,           // Over maximum
  "email": "not-email"  // Invalid format
}
```

---

#### SailPoint Transform Schema

**Schema for Static Transform:**
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["name", "type", "attributes"],
  "properties": {
    "name": {
      "type": "string",
      "pattern": "^[a-zA-Z][a-zA-Z0-9_]*$",
      "description": "Attribute name (alphanumeric, starts with letter)"
    },
    "type": {
      "type": "string",
      "enum": ["static"],
      "description": "Must be 'static'"
    },
    "attributes": {
      "type": "object",
      "required": ["value"],
      "properties": {
        "value": {
          "type": "string",
          "description": "Static value to return"
        }
      }
    }
  }
}
```

---

#### Schema for Concat Transform
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["name", "type", "attributes"],
  "properties": {
    "name": {
      "type": "string"
    },
    "type": {
      "type": "string",
      "enum": ["concat"]
    },
    "attributes": {
      "type": "object",
      "required": ["values"],
      "properties": {
        "values": {
          "type": "array",
          "minItems": 2,
          "items": {
            "type": "object",
            "required": ["type", "attributes"],
            "properties": {
              "type": {
                "type": "string",
                "enum": ["static", "identityAttribute", "accountAttribute", "reference"]
              },
              "attributes": {
                "type": "object"
              }
            }
          }
        }
      }
    }
  }
}
```

---

#### Validating JSON Against Schema

**Using Python:**
```python
import json
from jsonschema import validate, ValidationError

# Define schema
schema = {
    "type": "object",
    "required": ["name", "type", "attributes"],
    "properties": {
        "name": {"type": "string"},
        "type": {"type": "string"},
        "attributes": {"type": "object"}
    }
}

# JSON to validate
data = {
    "name": "email",
    "type": "static",
    "attributes": {
        "value": "test@company.com"
    }
}

# Validate
try:
    validate(instance=data, schema=schema)
    print("✓ Valid JSON!")
except ValidationError as e:
    print(f"✗ Invalid: {e.message}")
```

---

#### Using Online Schema Validators

**Tools:**
- https://www.jsonschemavalidator.net/
- https://jsonschemalint.com/

**Steps:**
1. Paste your schema on left
2. Paste your JSON on right
3. Click "Validate"
4. See errors or success

---

#### Why Use Schemas?

**Benefits:**
```
✓ Catch errors before API calls
✓ Document expected structure
✓ Auto-generate validation code
✓ Ensure consistency
✓ Self-documenting APIs
```

**Example: Pre-API Validation**
```python
def create_transform_safe(transform_json):
    """Create transform with validation"""
    
    # Validate against schema first
    try:
        validate(instance=transform_json, schema=TRANSFORM_SCHEMA)
    except ValidationError as e:
        print(f"Invalid transform: {e.message}")
        return None
    
    # If valid, make API call
    response = requests.post(
        f"{BASE_URL}/transforms",
        headers=HEADERS,
        json=transform_json
    )
    
    return response.json()
```

**This prevents invalid API calls!**

---

### Practice Exercise 1: Schema Validation

**Task:** Create a schema for a FirstValid transform

**Requirements:**
- Must have name (string)
- Must have type = "firstValid"
- Must have attributes object
- attributes must have values array
- values array must have at least 2 items
- Must have ignoreErrors (boolean)

<details>
<summary>Click to reveal answer</summary>
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["name", "type", "attributes"],
  "properties": {
    "name": {
      "type": "string"
    },
    "type": {
      "type": "string",
      "enum": ["firstValid"]
    },
    "attributes": {
      "type": "object",
      "required": ["values", "ignoreErrors"],
      "properties": {
        "values": {
          "type": "array",
          "minItems": 2
        },
        "ignoreErrors": {
          "type": "boolean"
        }
      }
    }
  }
}
```

</details>

---

### Part 2: Complex Nested Patterns (45 minutes)

**Mastering deeply nested JSON structures**

---

#### Pattern 1: Recursive Transforms

**Transform that calls itself (conditional chains):**
```json
{
  "name": "lifecycleState",
  "type": "conditional",
  "attributes": {
    "expression": "status == 'Leave'",
    "positiveCondition": "Leave of Absence",
    "negativeCondition": {
      "type": "conditional",
      "attributes": {
        "expression": "status == 'Active'",
        "positiveCondition": "Active",
        "negativeCondition": {
          "type": "conditional",
          "attributes": {
            "expression": "status == 'Terminated'",
            "positiveCondition": "Terminated",
            "negativeCondition": "Unknown"
          }
        }
      }
    }
  }
}
```

**Nesting depth: 3 levels**

---

#### Pattern 2: Multi-Stage Chains

**Transform with 6+ chained operations:**
```json
{
  "name": "email",
  "type": "trim",
  "attributes": {
    "input": {
      "type": "identityAttribute",
      "attributes": {
        "name": "firstname"
      }
    },
    "next": {
      "type": "replace",
      "attributes": {
        "regex": " ",
        "replacement": "",
        "next": {
          "type": "lower",
          "next": {
            "type": "concat",
            "attributes": {
              "values": [
                {
                  "type": "reference"
                },
                {
                  "type": "static",
                  "attributes": {
                    "value": "."
                  }
                },
                {
                  "type": "trim",
                  "attributes": {
                    "input": {
                      "type": "identityAttribute",
                      "attributes": {
                        "name": "lastname"
                      }
                    },
                    "next": {
                      "type": "replace",
                      "attributes": {
                        "regex": " ",
                        "replacement": "",
                        "next": {
                          "type": "lower"
                        }
                      }
                    }
                  }
                },
                {
                  "type": "static",
                  "attributes": {
                    "value": "@company.com"
                  }
                }
              ],
              "next": {
                "type": "substring",
                "attributes": {
                  "begin": 0,
                  "end": 50
                }
              }
            }
          }
        }
      }
    }
  }
}
```

**Flow:**
```
Trim(firstname) 
→ Replace spaces 
→ Lower 
→ Concat(
    result + 
    "." + 
    Trim(lastname) → Replace spaces → Lower +
    "@company.com"
  ) 
→ Substring(0, 50)
```

---

#### Pattern 3: Array of Complex Objects

**Identity Profile with multiple transforms:**
```json
{
  "name": "Employees",
  "identityAttributeConfig": {
    "attributeTransforms": [
      {
        "identityAttributeName": "displayName",
        "transformDefinition": {
          "type": "concat",
          "attributes": {
            "values": [...]
          }
        }
      },
      {
        "identityAttributeName": "email",
        "transformDefinition": {
          "type": "firstValid",
          "attributes": {
            "values": [...]
          }
        }
      },
      {
        "identityAttributeName": "lifecycleState",
        "transformDefinition": {
          "type": "conditional",
          "attributes": {
            "expression": "...",
            "positiveCondition": "...",
            "negativeCondition": {
              "type": "conditional",
              "attributes": {...}
            }
          }
        }
      }
    ]
  }
}
```

**Structure:**
```
Profile
└─ attributeTransforms (array)
   ├─ Transform 1 (object)
   │  └─ transformDefinition (complex nested)
   ├─ Transform 2 (object)
   │  └─ transformDefinition (complex nested)
   └─ Transform 3 (object)
      └─ transformDefinition (complex nested)
```

---

#### Navigating Deeply Nested JSON

**Strategy: JSON Path Notation**
```
Root object
$.identityAttributeConfig
$.identityAttributeConfig.attributeTransforms
$.identityAttributeConfig.attributeTransforms[0]
$.identityAttributeConfig.attributeTransforms[0].transformDefinition
$.identityAttributeConfig.attributeTransforms[0].transformDefinition.attributes
```

**Using JSONPath in Python:**
```python
from jsonpath_ng import parse

data = {
  "identityAttributeConfig": {
    "attributeTransforms": [
      {
        "identityAttributeName": "email",
        "transformDefinition": {
          "type": "static",
          "attributes": {
            "value": "test@company.com"
          }
        }
      }
    ]
  }
}

# Find all transform types
jsonpath_expr = parse('$.identityAttributeConfig.attributeTransforms[*].transformDefinition.type')
matches = [match.value for match in jsonpath_expr.find(data)]
print(matches)  # ['static']
```

---

#### Building Complex JSON Programmatically

**Don't write huge JSON manually - build it programmatically:**
```python
def create_concat_transform(name, *values):
    """Helper to create concatenation transforms"""
    return {
        "name": name,
        "type": "concat",
        "attributes": {
            "values": list(values)
        }
    }

def identity_attr(name):
    """Helper for identity attribute reference"""
    return {
        "type": "identityAttribute",
        "attributes": {
            "name": name
        }
    }

def static_value(value):
    """Helper for static value"""
    return {
        "type": "static",
        "attributes": {
            "value": value
        }
    }

# Now build complex transform easily
email_transform = create_concat_transform(
    "email",
    identity_attr("firstname"),
    static_value("."),
    identity_attr("lastname"),
    static_value("@company.com")
)

print(json.dumps(email_transform, indent=2))
```

**Output:**
```json
{
  "name": "email",
  "type": "concat",
  "attributes": {
    "values": [
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "firstname"
        }
      },
      {
        "type": "static",
        "attributes": {
          "value": "."
        }
      },
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "lastname"
        }
      },
      {
        "type": "static",
        "attributes": {
          "value": "@company.com"
        }
      }
    ]
  }
}
```

**Much easier than writing JSON manually!**

---

### Part 3: Performance Optimization (30 minutes)

**Making your JSON operations faster and more efficient**

---

#### Performance Tip 1: Minimize API Calls

**❌ Bad: Multiple individual calls**
```python
# Slow: 100 API calls
for transform in transforms:
    response = requests.post(url, json=transform)
```

**✓ Good: Batch operations**
```python
# Fast: 1 API call (if supported)
response = requests.post(batch_url, json={"transforms": transforms})

# Or use threading for parallel requests
from concurrent.futures import ThreadPoolExecutor

def create_transform(transform):
    return requests.post(url, json=transform)

with ThreadPoolExecutor(max_workers=10) as executor:
    results = executor.map(create_transform, transforms)
```

---

#### Performance Tip 2: Stream Large JSON

**❌ Bad: Load entire file into memory**
```python
# Memory intensive for large files
with open("large.json", "r") as f:
    data = json.load(f)  # Loads entire file
```

**✓ Good: Stream processing**
```python
import ijson

# Stream large JSON file
with open("large.json", "r") as f:
    # Process items one at a time
    for item in ijson.items(f, "transforms.item"):
        process_transform(item)
        # Only one item in memory at a time
```

---

#### Performance Tip 3: Cache Expensive Operations

**❌ Bad: Repeated API calls for same data**
```python
# Called 1000 times, makes 1000 API calls
def get_transform(transform_id):
    response = requests.get(f"{url}/{transform_id}")
    return response.json()
```

**✓ Good: Cache results**
```python
from functools import lru_cache

@lru_cache(maxsize=128)
def get_transform(transform_id):
    response = requests.get(f"{url}/{transform_id}")
    return response.json()

# First call: API request
transform = get_transform("id1")

# Second call: Returns cached result (instant!)
transform = get_transform("id1")
```

---

#### Performance Tip 4: Efficient JSON Parsing

**❌ Bad: Parse same JSON multiple times**
```python
json_string = '{"name": "test"}'

# Parsing 3 times
name = json.loads(json_string)["name"]
type = json.loads(json_string)["type"]
attrs = json.loads(json_string)["attributes"]
```

**✓ Good: Parse once, reuse**
```python
json_string = '{"name": "test"}'

# Parse once
data = json.loads(json_string)

# Access multiple times
name = data["name"]
type = data.get("type")
attrs = data.get("attributes", {})
```

---

#### Performance Tip 5: Use Generators for Large Datasets

**❌ Bad: Load everything at once**
```python
def get_all_transforms():
    response = requests.get(url)
    return response.json()  # Returns huge list

transforms = get_all_transforms()  # All in memory
for transform in transforms:
    process(transform)
```

**✓ Good: Use generator**
```python
def get_transforms_generator():
    offset = 0
    limit = 100
    
    while True:
        response = requests.get(f"{url}?limit={limit}&offset={offset}")
        batch = response.json()
        
        if not batch:
            break
        
        for transform in batch:
            yield transform  # Yield one at a time
        
        offset += limit

# Only one transform in memory at a time
for transform in get_transforms_generator():
    process(transform)
```

---

## Afternoon Session: Production Best Practices

### Part 4: Error Handling Strategies (60 minutes)

**Building robust, production-ready JSON systems**

---

#### Error Handling Layers

**Layer 1: JSON Syntax Validation**
```python
import json

def safe_json_load(json_string):
    """Safely load JSON with error handling"""
    try:
        return json.loads(json_string)
    except json.JSONDecodeError as e:
        print(f"Invalid JSON syntax: {e}")
        print(f"Error at line {e.lineno}, column {e.colno}")
        return None
```

---

#### Layer 2: Schema Validation
```python
from jsonschema import validate, ValidationError

def validate_transform(transform_json):
    """Validate transform against schema"""
    try:
        validate(instance=transform_json, schema=TRANSFORM_SCHEMA)
        return True
    except ValidationError as e:
        print(f"Schema validation failed: {e.message}")
        print(f"Failed at path: {' -> '.join(str(p) for p in e.path)}")
        return False
```

---

#### Layer 3: API Error Handling
```python
import requests
from requests.exceptions import RequestException

def create_transform_with_retry(transform, max_retries=3):
    """Create transform with retry logic"""
    
    for attempt in range(max_retries):
        try:
            response = requests.post(
                f"{BASE_URL}/transforms",
                headers=HEADERS,
                json=transform,
                timeout=10
            )
            
            # Check status code
            if response.status_code == 201:
                return response.json()
            elif response.status_code == 400:
                print(f"Bad request: {response.json()}")
                return None  # Don't retry bad requests
            elif response.status_code == 429:
                # Rate limited, wait and retry
                wait_time = int(response.headers.get('Retry-After', 60))
                print(f"Rate limited, waiting {wait_time}s...")
                time.sleep(wait_time)
                continue
            else:
                print(f"Unexpected status: {response.status_code}")
                response.raise_for_status()
                
        except RequestException as e:
            print(f"Request failed (attempt {attempt + 1}/{max_retries}): {e}")
            if attempt < max_retries - 1:
                time.sleep(2 ** attempt)  # Exponential backoff
            else:
                raise
    
    return None
```

---

#### Layer 4: Business Logic Validation
```python
def validate_transform_business_rules(transform):
    """Validate business rules"""
    errors = []
    
    # Rule 1: Name must be alphanumeric
    if not transform["name"].replace("_", "").isalnum():
        errors.append("Name must be alphanumeric")
    
    # Rule 2: Static transforms must have non-empty value
    if transform["type"] == "static":
        if not transform["attributes"].get("value"):
            errors.append("Static transform must have value")
    
    # Rule 3: Concat must have at least 2 values
    if transform["type"] == "concat":
        if len(transform["attributes"].get("values", [])) < 2:
            errors.append("Concat must have at least 2 values")
    
    return errors
```

---

#### Complete Error Handling Pipeline
```python
def create_transform_production(transform_json):
    """Production-grade transform creation with full error handling"""
    
    # Step 1: Validate JSON structure
    if not isinstance(transform_json, dict):
        print("Error: Transform must be a JSON object")
        return None
    
    # Step 2: Validate required fields
    required_fields = ["name", "type", "attributes"]
    missing_fields = [f for f in required_fields if f not in transform_json]
    if missing_fields:
        print(f"Error: Missing required fields: {missing_fields}")
        return None
    
    # Step 3: Validate against schema
    if not validate_transform(transform_json):
        print("Error: Schema validation failed")
        return None
    
    # Step 4: Validate business rules
    business_errors = validate_transform_business_rules(transform_json)
    if business_errors:
        print(f"Error: Business rule violations: {business_errors}")
        return None
    
    # Step 5: Create via API with retry
    try:
        result = create_transform_with_retry(transform_json)
        if result:
            print(f"✓ Successfully created: {transform_json['name']}")
            return result
        else:
            print(f"✗ Failed to create: {transform_json['name']}")
            return None
    except Exception as e:
        print(f"✗ Unexpected error: {e}")
        return None
```

---

#### Logging Best Practices
```python
import logging
from datetime import datetime

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler(f'transform_operations_{datetime.now().strftime("%Y%m%d")}.log'),
        logging.StreamHandler()
    ]
)

logger = logging.getLogger(__name__)

def create_transform_with_logging(transform):
    """Create transform with comprehensive logging"""
    
    logger.info(f"Creating transform: {transform['name']}")
    logger.debug(f"Transform JSON: {json.dumps(transform, indent=2)}")
    
    try:
        response = requests.post(url, json=transform, headers=headers)
        
        logger.info(f"API response status: {response.status_code}")
        
        if response.status_code == 201:
            logger.info(f"✓ Successfully created: {transform['name']}")
            logger.debug(f"Response: {response.json()}")
            return response.json()
        else:
            logger.error(f"✗ Failed to create: {transform['name']}")
            logger.error(f"Status: {response.status_code}")
            logger.error(f"Response: {response.text}")
            return None
            
    except Exception as e:
        logger.exception(f"Exception while creating transform: {transform['name']}")
        raise
```

---

### Part 5: Testing & Validation (60 minutes)

**Ensuring your JSON operations work correctly**

---

#### Unit Testing JSON Functions
```python
import unittest
import json

class TestTransformGeneration(unittest.TestCase):
    
    def test_create_static_transform(self):
        """Test static transform creation"""
        transform = create_static_transform("test", "value")
        
        self.assertEqual(transform["name"], "test")
        self.assertEqual(transform["type"], "static")
        self.assertEqual(transform["attributes"]["value"], "value")
    
    def test_create_concat_transform(self):
        """Test concat transform creation"""
        transform = create_concat_transform(
            "email",
            identity_attr("firstname"),
            static_value("@company.com")
        )
        
        self.assertEqual(transform["type"], "concat")
        self.assertEqual(len(transform["attributes"]["values"]), 2)
    
    def test_validate_transform_schema(self):
        """Test schema validation"""
        valid_transform = {
            "name": "test",
            "type": "static",
            "attributes": {"value": "test"}
        }
        
        invalid_transform = {
            "name": "test"
            # Missing type and attributes
        }
        
        self.assertTrue(validate_transform(valid_transform))
        self.assertFalse(validate_transform(invalid_transform))

if __name__ == '__main__':
    unittest.main()
```

---

#### Integration Testing with API
```python
class TestTransformAPI(unittest.TestCase):
    
    @classmethod
    def setUpClass(cls):
        """Set up test fixtures"""
        cls.test_transforms = []
    
    def test_create_transform(self):
        """Test creating transform via API"""
        transform = {
            "name": f"test_{int(time.time())}",
            "type": "static",
            "attributes": {"value": "test"}
        }
        
        response = requests.post(
            f"{BASE_URL}/transforms",
            headers=HEADERS,
            json=transform
        )
        
        self.assertEqual(response.status_code, 201)
        result = response.json()
        self.assertIn("id", result)
        
        # Store for cleanup
        self.test_transforms.append(result["id"])
    
    def test_get_transform(self):
        """Test getting transform via API"""
        # Create transform first
        transform = {...}
        create_response = requests.post(url, json=transform, headers=headers)
        transform_id = create_response.json()["id"]
        
        # Get it back
        get_response = requests.get(
            f"{BASE_URL}/transforms/{transform_id}",
            headers=HEADERS
        )
        
        self.assertEqual(get_response.status_code, 200)
        result = get_response.json()
        self.assertEqual(result["name"], transform["name"])
    
    @classmethod
    def tearDownClass(cls):
        """Clean up test transforms"""
        for transform_id in cls.test_transforms:
            requests.delete(
                f"{BASE_URL}/transforms/{transform_id}",
                headers=HEADERS
            )
```

---

#### Test Data Fixtures

**Create reusable test data:**
```python
# test_fixtures.py

STATIC_TRANSFORM = {
    "name": "testStatic",
    "type": "static",
    "attributes": {
        "value": "test value"
    }
}

CONCAT_TRANSFORM = {
    "name": "testConcat",
    "type": "concat",
    "attributes": {
        "values": [
            {
                "type": "identityAttribute",
                "attributes": {"name": "firstname"}
            },
            {
                "type": "static",
                "attributes": {"value": " "}
            },
            {
                "type": "identityAttribute",
                "attributes": {"name": "lastname"}
            }
        ]
    }
}

FIRSTVALID_TRANSFORM = {
    "name": "testFirstValid",
    "type": "firstValid",
    "attributes": {
        "values": [
            {
                "type": "identityAttribute",
                "attributes": {"name": "email1"}
            },
            {
                "type": "identityAttribute",
                "attributes": {"name": "email2"}
            },
            {
                "type": "static",
                "attributes": {"value": "default@company.com"}
            }
        ],
        "ignoreErrors": True
    }
}
```

---

#### Validation Checklist

**Before deploying to production:**
```
Pre-Deployment Checklist:

□ JSON Syntax
  □ Valid JSON (no syntax errors)
  □ Properly formatted (indentation, no trailing commas)
  
□ Schema Validation
  □ Passes schema validation
  □ All required fields present
  □ Correct data types
  
□ Business Rules
  □ Names follow conventions
  □ Values within acceptable ranges
  □ No circular references
  
□ Testing
  □ Unit tests pass
  □ Integration tests pass
  □ Tested with sample data
  □ Edge cases covered
  
□ Documentation
  □ Purpose documented
  □ Dependencies documented
  □ Known limitations noted
  
□ API Operations
  □ Creates successfully
  □ Updates work as expected
  □ Can be deleted cleanly
  □ Error handling tested
  
□ Performance
  □ Tested with production volume
  □ No memory leaks
  □ Acceptable response time
  
□ Security
  □ No hardcoded credentials
  □ Proper access controls
  □ Audit logging enabled
```

---

## Evening Session: Capstone Project

### Part 6: Build Complete Automation System (120 minutes)

**Final Project: Transform Management System**

---

#### Project Requirements

Build a complete system that:
```
1. Transform Library Management
   - Store transform templates
   - Version control
   - Search and filter

2. Validation Pipeline
   - JSON syntax validation
   - Schema validation
   - Business rules validation

3. API Integration
   - Create transforms
   - Update transforms
   - Delete transforms
   - Backup/restore

4. Deployment Automation
   - Dev → Prod promotion
   - Rollback capability
   - Change tracking

5. Reporting
   - Generate transform inventory
   - Deployment logs
   - Error reports
```

---

#### Project Structure
```
sailpoint-transform-manager/
├── config/
│   ├── config.json           # API credentials, tenant info
│   └── schemas/
│       ├── static.json        # Transform schemas
│       ├── concat.json
│       └── conditional.json
├── templates/
│   ├── email.json             # Transform templates
│   ├── username.json
│   └── lifecycle.json
├── src/
│   ├── __init__.py
│   ├── api_client.py          # SailPoint API wrapper
│   ├── validator.py           # Validation functions
│   ├── transform_builder.py  # Build transforms programmatically
│   ├── deployer.py            # Deployment functions
│   └── reporter.py            # Reporting functions
├── tests/
│   ├── test_validator.py
│   ├── test_api_client.py
│   └── test_transform_builder.py
├── logs/
│   └── .gitkeep
├── backups/
│   └── .gitkeep
├── requirements.txt
└── main.py                    # CLI interface
```

---

#### Component 1: API Client

**File: `src/api_client.py`**
```python
import requests
import json
from typing import Dict, List, Optional

class SailPointAPIClient:
    """Client for SailPoint ISC API operations"""
    
    def __init__(self, tenant: str, token: str):
        self.tenant = tenant
        self.base_url = f"https://{tenant}.api.identitynow.com/v3"
        self.headers = {
            "Authorization": f"Bearer {token}",
            "Content-Type": "application/json"
        }
    
    def get_all_transforms(self) -> List[Dict]:
        """Get all transforms"""
        response = requests.get(
            f"{self.base_url}/transforms",
            headers=self.headers
        )
        response.raise_for_status()
        return response.json()
    
    def get_transform(self, transform_id: str) -> Dict:
        """Get specific transform by ID"""
        response = requests.get(
            f"{self.base_url}/transforms/{transform_id}",
            headers=self.headers
        )
        response.raise_for_status()
        return response.json()
    
    def create_transform(self, transform: Dict) -> Dict:
        """Create new transform"""
        response = requests.post(
            f"{self.base_url}/transforms",
            headers=self.headers,
            json=transform
        )
        response.raise_for_status()
        return response.json()
    
    def update_transform(self, transform_id: str, updates: Dict) -> Dict:
        """Update existing transform"""
        response = requests.patch(
            f"{self.base_url}/transforms/{transform_id}",
            headers=self.headers,
            json=updates
        )
        response.raise_for_status()
        return response.json()
    
    def delete_transform(self, transform_id: str) -> bool:
        """Delete transform"""
        response = requests.delete(
            f"{self.base_url}/transforms/{transform_id}",
            headers=self.headers
        )
        return response.status_code == 204
    
    def search_transforms(self, filter_query: str) -> List[Dict]:
        """Search transforms with filter"""
        response = requests.get(
            f"{self.base_url}/transforms?filters={filter_query}",
            headers=self.headers
        )
        response.raise_for_status()
        return response.json()
```

---

#### Component 2: Validator

**File: `src/validator.py`**
```python
import json
from jsonschema import validate, ValidationError
from typing import Dict, List, Tuple

class TransformValidator:
    """Validate transforms against schemas and business rules"""
    
    def __init__(self, schemas_dir: str):
        self.schemas = self._load_schemas(schemas_dir)
    
    def _load_schemas(self, schemas_dir: str) -> Dict:
        """Load all schema files"""
        schemas = {}
        # Load static.json, concat.json, etc.
        # Return dict of {type: schema}
        return schemas
    
    def validate_syntax(self, json_string: str) -> Tuple[bool, str]:
        """Validate JSON syntax"""
        try:
            json.loads(json_string)
            return True, "Valid JSON syntax"
        except json.JSONDecodeError as e:
            return False, f"Invalid JSON: {e}"
    
    def validate_schema(self, transform: Dict) -> Tuple[bool, str]:
        """Validate against transform schema"""
        transform_type = transform.get("type")
        
        if transform_type not in self.schemas:
            return False, f"Unknown transform type: {transform_type}"
        
        try:
            validate(instance=transform, schema=self.schemas[transform_type])
            return True, "Schema validation passed"
        except ValidationError as e:
            return False, f"Schema validation failed: {e.message}"
    
    def validate_business_rules(self, transform: Dict) -> Tuple[bool, List[str]]:
        """Validate business rules"""
        errors = []
        
        # Rule 1: Name conventions
        name = transform.get("name", "")
        if not name[0].isalpha():
            errors.append("Name must start with a letter")
        
        if not name.replace("_", "").isalnum():
            errors.append("Name must be alphanumeric (underscores allowed)")
        
        # Rule 2: Type-specific validations
        if transform.get("type") == "static":
            if not transform.get("attributes", {}).get("value"):
                errors.append("Static transform must have non-empty value")
        
        if transform.get("type") == "concat":
            values = transform.get("attributes", {}).get("values", [])
            if len(values) < 2:
                errors.append("Concat must have at least 2 values")
        
        return len(errors) == 0, errors
    
    def validate_complete(self, transform: Dict) -> Tuple[bool, List[str]]:
        """Run complete validation pipeline"""
        errors = []
        
        # Check syntax (if string)
        # Check schema
        valid_schema, schema_msg = self.validate_schema(transform)
        if not valid_schema:
            errors.append(schema_msg)
        
        # Check business rules
        valid_rules, rule_errors = self.validate_business_rules(transform)
        if not valid_rules:
            errors.extend(rule_errors)
        
        return len(errors) == 0, errors
```

---

#### Component 3: Transform Builder

**File: `src/transform_builder.py`**
```python
from typing import Dict, List

class TransformBuilder:
    """Programmatically build transforms"""
    
    @staticmethod
    def static(name: str, value: str) -> Dict:
        """Build static transform"""
        return {
            "name": name,
            "type": "static",
            "attributes": {
                "value": value
            }
        }
    
    @staticmethod
    def identity_attr(name: str) -> Dict:
        """Build identity attribute reference"""
        return {
            "type": "identityAttribute",
            "attributes": {
                "name": name
            }
        }
    
    @staticmethod
    def static_value(value: str) -> Dict:
        """Build static value for concat"""
        return {
            "type": "static",
            "attributes": {
                "value": value
            }
        }
    
    @staticmethod
    def concat(name: str, *values) -> Dict:
        """Build concatenation transform"""
        return {
            "name": name,
            "type": "concat",
            "attributes": {
                "values": list(values)
            }
        }
    
    @staticmethod
    def first_valid(name: str, *values, ignore_errors=True) -> Dict:
        """Build firstValid transform"""
        return {
            "name": name,
            "type": "firstValid",
            "attributes": {
                "values": list(values),
                "ignoreErrors": ignore_errors
            }
        }
    
    @staticmethod
    def add_chain(transform: Dict, next_transform_type: str, **kwargs) -> Dict:
        """Add next transform to chain"""
        transform["attributes"]["next"] = {
            "type": next_transform_type,
            **kwargs
        }
        return transform
```

---

#### Component 4: Deployer

**File: `src/deployer.py`**
```python
import json
from datetime import datetime
from typing import Dict, List
import logging

class TransformDeployer:
    """Deploy transforms with backup and rollback"""
    
    def __init__(self, api_client, validator, backup_dir: str):
        self.api = api_client
        self.validator = validator
        self.backup_dir = backup_dir
        self.logger = logging.getLogger(__name__)
    
    def backup_current_state(self) -> str:
        """Backup all current transforms"""
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        backup_file = f"{self.backup_dir}/backup_{timestamp}.json"
        
        transforms = self.api.get_all_transforms()
        
        with open(backup_file, "w") as f:
            json.dump(transforms, f, indent=2)
        
        self.logger.info(f"Backed up {len(transforms)} transforms to {backup_file}")
        return backup_file
    
    def deploy_transform(self, transform: Dict, validate=True) -> Dict:
        """Deploy single transform with validation"""
        
        if validate:
            valid, errors = self.validator.validate_complete(transform)
            if not valid:
                raise ValueError(f"Validation failed: {errors}")
        
        # Check if exists
        existing = self.api.search_transforms(f"name eq '{transform['name']}'")
        
        if existing:
            # Update existing
            self.logger.info(f"Updating existing transform: {transform['name']}")
            result = self.api.update_transform(existing[0]["id"], transform)
        else:
            # Create new
            self.logger.info(f"Creating new transform: {transform['name']}")
            result = self.api.create_transform(transform)
        
        return result
    
    def deploy_batch(self, transforms: List[Dict], backup=True) -> Dict:
        """Deploy multiple transforms with rollback on failure"""
        
        if backup:
            backup_file = self.backup_current_state()
        
        deployed = []
        failed = []
        
        for transform in transforms:
            try:
                result = self.deploy_transform(transform)
                deployed.append(result)
                self.logger.info(f"✓ Deployed: {transform['name']}")
            except Exception as e:
                failed.append({
                    "transform": transform["name"],
                    "error": str(e)
                })
                self.logger.error(f"✗ Failed: {transform['name']} - {e}")
        
        return {
            "deployed": deployed,
            "failed": failed,
            "backup_file": backup_file if backup else None
        }
    
    def rollback(self, backup_file: str) -> bool:
        """Rollback to backup state"""
        self.logger.info(f"Rolling back to {backup_file}")
        
        with open(backup_file, "r") as f:
            backup_transforms = json.load(f)
        
        # Get current state
        current = self.api.get_all_transforms()
        current_names = {t["name"] for t in current}
        backup_names = {t["name"] for t in backup_transforms}
        
        # Delete transforms not in backup
        to_delete = current_names - backup_names
        for name in to_delete:
            transforms = self.api.search_transforms(f"name eq '{name}'")
            if transforms:
                self.api.delete_transform(transforms[0]["id"])
                self.logger.info(f"Deleted: {name}")
        
        # Restore backup transforms
        for transform in backup_transforms:
            transform.pop("id", None)  # Remove ID
            self.deploy_transform(transform, validate=False)
        
        self.logger.info("Rollback complete")
        return True
```

---

#### Component 5: CLI Interface

**File: `main.py`**
```python
import argparse
import json
import logging
from src.api_client import SailPointAPIClient
from src.validator import TransformValidator
from src.transform_builder import TransformBuilder
from src.deployer import TransformDeployer

def setup_logging():
    logging.basicConfig(
        level=logging.INFO,
        format='%(asctime)s - %(levelname)s - %(message)s'
    )

def load_config():
    with open("config/config.json", "r") as f:
        return json.load(f)

def main():
    setup_logging()
    config = load_config()
    
    # Initialize components
    api = SailPointAPIClient(config["tenant"], config["token"])
    validator = TransformValidator("config/schemas")
    deployer = TransformDeployer(api, validator, "backups")
    
    # CLI argument parser
    parser = argparse.ArgumentParser(description="SailPoint Transform Manager")
    subparsers = parser.add_subparsers(dest="command")
    
    # List command
    subparsers.add_parser("list", help="List all transforms")
    
    # Create command
    create_parser = subparsers.add_parser("create", help="Create transform")
    create_parser.add_argument("file", help="JSON file with transform")
    
    # Deploy command
    deploy_parser = subparsers.add_parser("deploy", help="Deploy transforms")
    deploy_parser.add_argument("directory", help="Directory with transforms")
    
    # Backup command
    subparsers.add_parser("backup", help="Backup all transforms")
    
    # Rollback command
    rollback_parser = subparsers.add_parser("rollback", help="Rollback to backup")
    rollback_parser.add_argument("backup_file", help="Backup file to restore")
    
    args = parser.parse_args()
    
    # Execute commands
    if args.command == "list":
        transforms = api.get_all_transforms()
        for t in transforms:
            print(f"{t['name']} ({t['type']})")
    
    elif args.command == "create":
        with open(args.file, "r") as f:
            transform = json.load(f)
        result = deployer.deploy_transform(transform)
        print(f"Created: {result['name']} (ID: {result['id']})")
    
    elif args.command == "deploy":
        # Load all JSON files from directory
        import os
        transforms = []
        for filename in os.listdir(args.directory):
            if filename.endswith(".json"):
                with open(os.path.join(args.directory, filename), "r") as f:
                    transforms.append(json.load(f))
        
        result = deployer.deploy_batch(transforms)
        print(f"Deployed: {len(result['deployed'])}")
        print(f"Failed: {len(result['failed'])}")
    
    elif args.command == "backup":
        backup_file = deployer.backup_current_state()
        print(f"Backup saved to: {backup_file}")
    
    elif args.command == "rollback":
        deployer.rollback(args.backup_file)
        print("Rollback complete")

if __name__ == "__main__":
    main()
```

---

#### Usage Examples

**List all transforms:**
```bash
python main.py list
```

**Create a transform:**
```bash
python main.py create templates/email.json
```

**Deploy all transforms from directory:**
```bash
python main.py deploy templates/
```

**Backup current state:**
```bash
python main.py backup
```

**Rollback to backup:**
```bash
python main.py rollback backups/backup_20240206_120000.json
```

---

#### Testing the System

**File: `tests/test_api_client.py`**
```python
import unittest
from src.api_client import SailPointAPIClient

class TestAPIClient(unittest.TestCase):
    
    def setUp(self):
        self.client = SailPointAPIClient("test-tenant", "test-token")
    
    def test_get_all_transforms(self):
        # Test with mock
        transforms = self.client.get_all_transforms()
        self.assertIsInstance(transforms, list)
    
    # More tests...
```

---

## JSON Mastery Certification

### Final Assessment

**Complete this assessment to earn your JSON Mastery Certificate!**

---

#### Question 1: JSON Syntax (10 points)

Fix all errors in this JSON:
```json
{
  'name': "John Smith",
  "age": 30,
  email: "john@company.com",
  "skills": ["Python", "JavaScript",],
  "active": True,
}
```

<details>
<summary>Click to reveal answer</summary>
```json
{
  "name": "John Smith",
  "age": 30,
  "email": "john@company.com",
  "skills": ["Python", "JavaScript"],
  "active": true
}
```

**Errors fixed:**
1. Single quotes on 'name' → double quotes
2. Missing quotes on email key
3. Trailing comma in skills array
4. Capital T in True → lowercase
5. Trailing comma after active

</details>

---

#### Question 2: SailPoint Transform (20 points)

Write complete JSON for a transform that:
- Name: "primaryEmail"
- Tries personal_email first
- Falls back to work_email
- Falls back to firstname.lastname@company.com (lowercase)
- Final fallback: "noemail@company.com"

<details>
<summary>Click to reveal answer</summary>
```json
{
  "name": "primaryEmail",
  "type": "firstValid",
  "attributes": {
    "values": [
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "personal_email"
        }
      },
      {
        "type": "identityAttribute",
        "attributes": {
          "name": "work_email"
        }
      },
      {
        "type": "concat",
        "attributes": {
          "values": [
            {
              "type": "identityAttribute",
              "attributes": {
                "name": "firstname"
              }
            },
            {
              "type": "static",
              "attributes": {
                "value": "."
              }
            },
            {
              "type": "identityAttribute",
              "attributes": {
                "name": "lastname"
              }
            },
            {
              "type": "static",
              "attributes": {
                "value": "@company.com"
              }
            }
          ],
          "next": {
            "type": "lower"
          }
        }
      },
      {
        "type": "static",
        "attributes": {
          "value": "noemail@company.com"
        }
      }
    ],
    "ignoreErrors": true
  }
}
```

</details>

---

#### Question 3: API Operations (20 points)

Write Python code to:
1. Get all transforms of type "static"
2. Update each one to append " - Updated" to the value
3. Handle errors gracefully
4. Log all operations

<details>
<summary>Click to reveal answer</summary>
```python
import requests
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def update_all_static_transforms(base_url, headers):
    """Update all static transforms"""
    
    try:
        # Get all static transforms
        response = requests.get(
            f"{base_url}/transforms?filters=type eq 'static'",
            headers=headers
        )
        response.raise_for_status()
        transforms = response.json()
        
        logger.info(f"Found {len(transforms)} static transforms")
        
        # Update each one
        for transform in transforms:
            try:
                transform_id = transform["id"]
                current_value = transform["attributes"]["value"]
                new_value = f"{current_value} - Updated"
                
                update_data = {
                    "attributes": {
                        "value": new_value
                    }
                }
                
                response = requests.patch(
                    f"{base_url}/transforms/{transform_id}",
                    headers=headers,
                    json=update_data
                )
                response.raise_for_status()
                
                logger.info(f"✓ Updated: {transform['name']}")
                
            except Exception as e:
                logger.error(f"✗ Failed to update {transform['name']}: {e}")
        
    except Exception as e:
        logger.error(f"Failed to get transforms: {e}")

# Usage
update_all_static_transforms(BASE_URL, HEADERS)
```

</details>

---

#### Question 4: Complete Project (50 points)

**Build a complete transform deployment system that:**

1. Loads transforms from a directory
2. Validates each transform
3. Backs up current state
4. Deploys to SailPoint
5. Rolls back on error
6. Generates deployment report

**Submit your code for review**

---

### Scoring
```
90-100 points: JSON Master ⭐⭐⭐
80-89 points: JSON Expert ⭐⭐
70-79 points: JSON Proficient ⭐
60-69 points: JSON Competent
<60 points: Review material and retake
```

---

## Day 5 Summary

### What You Learned Today

**Morning:**
- ✅ JSON Schema validation
- ✅ Complex nested patterns
- ✅ Performance optimization
- ✅ JSONPath navigation

**Afternoon:**
- ✅ Multi-layer error handling
- ✅ Production logging
- ✅ Unit and integration testing
- ✅ Validation pipelines

**Evening:**
- ✅ Complete automation system
- ✅ CLI interface design
- ✅ Deployment workflows
- ✅ Rollback strategies

---

### Complete 5-Day Journey

**Day 1:** JSON fundamentals, syntax rules, reading JSON
**Day 2:** Writing JSON, fixing errors, SailPoint transforms
**Day 3:** SailPoint ISC structures, bulk operations, patterns
**Day 4:** SailPoint API, CRUD operations, automation
**Day 5:** Advanced patterns, production practices, capstone

---

### Skills Mastered

You can now:
```
✓ Read any JSON structure
✓ Write valid JSON from scratch
✓ Fix JSON syntax errors instantly
✓ Understand SailPoint transform JSON
✓ Use SailPoint API with confidence
✓ Build automation scripts
✓ Validate with schemas
✓ Handle errors gracefully
✓ Test JSON systems
✓ Deploy to production
✓ Build complete automation platforms
✓ Version control configurations
✓ Promote across environments
```

---

## Final Certification
```
═══════════════════════════════════════════════════════════
           JSON MASTERY CERTIFICATION
═══════════════════════════════════════════════════════════

This certifies that

    [YOUR NAME]

has successfully completed the comprehensive JSON Mastery
training program and has demonstrated expertise in:

✓ JSON Syntax and Structure
✓ SailPoint ISC JSON Configuration
✓ REST API Operations
✓ Automation and Scripting
✓ Production Best Practices
✓ Advanced Patterns and Optimization

Completed: [DATE]
Level: JSON Master

═══════════════════════════════════════════════════════════
```

---

## What's Next?

**You've mastered JSON! Now you can:**

1. **Return to Identity Lifecycle Management (ILM)**
   - You now have the JSON skills needed
   - ILM uses JSON extensively
   - API-driven ILM automation

2. **Build Advanced Automation**
   - CI/CD pipelines for SailPoint
   - Multi-tenant management
   - Configuration as code

3. **Contribute to Community**
   - Share your JSON patterns
   - Help others learn
   - Build open-source tools

4. **Professional Advancement**
   - SailPoint certifications
   - DevOps for identity
   - Platform engineering

---

## Resources

**Official Documentation:**
- SailPoint API Docs: https://developer.sailpoint.com/
- JSON Schema: https://json-schema.org/

**Tools:**
- JSONLint: https://jsonlint.com/
- VS Code: https://code.visualstudio.com/
- Postman: https://www.postman.com/

**Community:**
- SailPoint Developer Community
- Stack Overflow (tag: sailpoint)
- GitHub SailPoint projects

---

**Congratulations on completing JSON Mastery! You're now a JSON expert ready to automate anything in SailPoint ISC!**

**Ready to continue with Identity Lifecycle Management (ILM)?**

---

*End of JSON Mastery - Day 5*
*End of Complete 5-Day JSON Mastery Program*
