# Repository to report openapi-enum Bug

## Bug that occurred while developing
[JSON specification of ENUMS](https://json-schema.org/understanding-json-schema/reference/generic.html?highlight=enum#enumerated-values)


```yaml
# File: openapi.yaml
firstBug:
  type: integer
  format: int32
  enum:
    - 0
    - 25
    - 50
    - 75
    - 100
  default: 50
```

results in 

(some generated comments have been removed for clarity)

```javascript
// File: CoolParameters.js (generated)

/**
 * @member {module:model/CoolParameters.FirstBugEnum} firstBug
 * @default FirstBugEnum.50
 */
CoolParameters.prototype['firstBug'] = FirstBugEnum.50;

/* . . . */

/**
 * Allowed values for the <code>firstBug</code> property.
 * @enum {Number}
 * @readonly
 */
CoolParameters['FirstBugEnum'] = {
    "0": 0,
    "25": 25,
    "50": 50,
    "75": 75,
    "100": 100
};
```

Babel which is already included in the generated module throws a `BABEL_PARSE_ERROR` with the code `MissingSemicolon`


An easy way how this could be fixed, when generated is the following;
```javascript
CoolParameters.prototype['firstBug'] = FirstBugEnum["50"];
```