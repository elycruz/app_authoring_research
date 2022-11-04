# Validation Structures


## Definitions

Validators
  Functions which take an input and give us a validation result.

Validation Structs
  Objects/Structures the contain configuration required for running validation and functions for performing validation.

Input Struct
  Structures that contain validation structions/validators for a given input.

InputsFilter
  A structure that contains key-value pair entries for "Input" fields structs.  These structures should also be where logic for running validation for a set of inputs is performed (whether validation is asynchronous or not).

## Validators

### Caveats

- These are mostly for "hybrid" Functional + Object Oriented platforms - Some programming environments contain strict rules over what can be shared across threads reliably so in those environments it can be difficult to share structs/objects across threads that may contain non thread-safe fields/values (rust is one of these) - This can limit what is possible for "usable from anywhere" structures/libraries etc.

### Ideal Implementation(s)

A Validator should (ideally) be made up of validator options, and
 a validator getter:

Example (*figure 1*):

```typescript

/**
 * Error/Validation message getter.
 */
type MessageGetter<T = any> = (x?: T, options?: ValidatorOptions<T>) => string;

/**
 * The "right hand side" value of the `MessageTemplates` type structure - * Either a string or a message getter.
 */
type MessageTemplateRValue<T = any> = string | MessageGetter<T>;

/**
 * A struct for keeping validation (error/invalid value) message getter functions, and/or static error message strings.
 *
 */
interface MessageTemplates<T = any> {
  [index: string]: MessageTemplateRValue<T>;
}

/**
 * All composed (defined/returned) Validator functions' option structures should implement/include this interface.
 */
interface ValidatorOptions<T = any> {
  messageTemplates?: MessageTemplates<T>;
  valueObscured?: boolean;
  valueObscurer?: Unary<T, ToString>;
}

/**
 * Return value of (composed) validator functions.
 */
interface ValidatorResult<T = any> {
  result: boolean;
  messages?: string[];
  value?: T;
}

/**
 * (composed) Validator function type (Note: validator function should be generated from a validator getter (which takes required validator options).
 */
type Validator<T = any> = (valueToValidator?: T) => ValidatorResult<T>;

```

### Example Validators 

Types which would be returned by their respective validator getter functions (e.g., `getNotEmptyValidator(...)`).

- NotEmptyValidator
- RegexValidator
- LengthValidator
- DigitValidator
- RangeValidator
- EmailValidator
- etc.

Example implementation (*figure 2* - Uses types defined in *figure 1*):

```typescript

interface NotEmptyValidator {
}

```




### Validator Structs

These are like Validator

Consolidated:

- TextInput
- NumberInput
- ArrayInput
- ObjectInput
- etc.

Pros: 

- Less structures to deal with.

Cons: 

- Object oriented like approach (fields in structs) etc.


