# Repository to report openapi-enum Bug

## Setup & Reproduction

1) install dependencies with `npm install`

2) generate API Code with `npm run generate-api` 

   - or instead use the following command 
```
openapi-generator-cli generate -o ./api -i ./openapi.yaml -g javascript && cd api && npm install && npm audit fix --force
```

________________________
# Bug that occurred
## Case 1: Bug that occurred while developing

According to the OpenAPI specification, the `enum` keyword is used to specify a set of acceptable values for a parameter or property. Since the OpenAPI specification allows for the use of JSON objects to describe API operations and parameters which can include both string values and numerical values, my Team and I assumed that the `enum` keyword in the OpenAPI specification can be used also with numerical values. As written in the [JSON specification ](https://json-schema.org/understanding-json-schema/reference/generic.html?highlight=enum#enumerated-values).

After we compiled this following code with the openapi-generator via openapi-generator-cli to javascript, we ran into some issues.

```yaml
# from File: openapi.yaml
firstCase:
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

If we now take a look into the files of the generated module, we see this code-snippet

```javascript
// from File: api/src/model/CoolParameters.js (generated)

/**
 * @member {module:model/CoolParameters.FirstCaseEnum} firstCase
 * @default FirstCaseEnum.50
 */
CoolParameters.prototype['firstCase'] = FirstCaseEnum.50;

/* . . . */

/**
 * Allowed values for the <code>firstCase</code> property.
 * @enum {Number}
 * @readonly
 */
CoolParameters['FirstCaseEnum'] = {

    /**
     * value: 0
     * @const
     */
    "0": 0,

    /**
     * value: 50
     * @const
     */
    "50": 50,

    /**
     * value: 100
     * @const
     */
    "100": 100
};

```

While building the generated Code, Babel -which is already included in the generated module- throws a `BABEL_PARSE_ERROR` with the code `MissingSemicolon`. Which makes sense, since the generated Code in the first codeline is not ECMAScript compatible.


An easy way how this could be fixed, would be maybe to generate the code as following:
```javascript
CoolParameters.prototype['firstCase'] = FirstCaseEnum['50'];
```

# Other Test Cases we experimented with
After this incident we asked ourselves if there could be also other test-cases that could result in a similiar issue. So we defined in the included yaml file other parameters with other potential cases, that could result into issues while developing. 

To get to the parameters that throw an error through babel, just comment out the various previous parameters in the yaml file that throw an error.

## Case 2: secondCase
```yaml
# from File: openapi.yaml
secondCase:
  enum:
  - 5
  - "5"
  default: 5
```
Babel throws `BABEL_PARSE_ERROR` with the code `MissingSemicolon`. 
Same error as described above. The generated code is the following:

```JavaScript
// from File: api/src/model/CoolParameters.js (generated)

/**
 * Allowed values for the <code>secondCase</code> property.
 * @enum {Number}
 * @readonly
 */
CoolParameters['SecondCaseEnum'] = {

    /**
     * value: 5
     * @const
     */
    "5": 5,

    /**
     * value: 5
     * @const
     */
    "5": 5
};
```

## Case 3: thirdCase

```yaml
# from File: openapi.yaml
thirdCase:
  enum:
  - 5
  - "5"
  default: "5"
```
compiles, but gives the following code as result, which could lead to some issues

```JavaScript
// from File: api/src/model/CoolParameters.js (generated)

/**
 * Allowed values for the <code>secondCase</code> property.
 * @enum {Number}
 * @readonly
 */
CoolParameters['thirdCaseEnum'] = {

    /**
     * value: 5
     * @const
     */
    "5": 5,

    /**
     * value: 5
     * @const
     */
    "5": 5
};
```


## Case 4: fourthCase
So we tried the same with `null` and `"null"`
```yaml
# from File: openapi.yaml
fourthCase:
  enum:
    - null
    - "null"
  default: null
```
Which results in 
```javascript
// from File: api/src/model/CoolParameters.js (generated)

CoolParameters['FourthCaseEnum'] = {

    /**
     * value: "null"
     * @const
     */
    "null": "null",

    /**
     * value: "null"
     * @const
     */
    "null": "null"
};
```
Since there is a `null` value this could also lead to some issues. If that is not that easy fixable: maybe the code generator could throw an exception which indicates that there are duplicate values.

# Experiments with TypeScript Generator

The TypeScript generator apparently does not generate Case 4 and the openapi-tools code generator throws an exception. It looks like its because of the `null` type, which the compiler can't compile into an TypeScript enum.

Case 1 - 3 compiles beautiful without any troubles. But there is probably no default value set (at least I didn't see where it could potentially be set). This could be the reason, why the issue of the javascript generator does not apply to the TypeScript Code.

## potential workaround
* Compile into TypeScript.
* then compile TypeScript code into JavaScript. <br>
NOT TESTED YET.