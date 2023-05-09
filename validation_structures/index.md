# Validation Structures


## Definitions

Validators -
  Functions which take an input and give us a validation result.

Validation Structs -
  Structures that contain configuration, and/or functions, required for  performing validation on a given input.

Input Struct -
  Structures that contain validation structures used to perform validation on  given inputs.

Input Filters Struct - 
  A structure that contains key-value pair entries for "Input" field structs.  These structures should also be where logic for running validation for a set of inputs is performed (whether validation is asynchronous or not).

## Validators

### Caveats

- These are mostly for "hybrid" Functional + Object-Oriented platforms - Some programming environments contain strict rules over what can be shared across threads reliably so in those environments it can be difficult to share structures across threads that may contain non thread-safe fields (rust is one of these languages) - This can limit what is possible for "usable from anywhere" structures, and/or, libraries.

### Pseudo-Ideal Implementation(s)

#### Generally Ideal Implementation:

Implementation would contain:

- Validators - functions that take an optional value and returns a tuple containing a validation state enum and an optional message (in the case of 'failed' validation, etc.).
- Inputs - Structures that would contain validators, and/or, filters for performing input filtering/validation.
- InputsFilters - Structure containing inputs, and general options, for performing input filtering, and/or validation, on it's set of inputs.


#### Example (legacy) Javascript Example

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

// @todo

interface NotEmptyValidatorOptions {
  // @todo	
}

```

### Validator Structs

These are like Validators except they contain all their validation/messaging properties directly on themselves.  Additionally, their validation/validator method(s) exist directly on themselves as well:

*figure 3* - Note: the implementation below is only partially validated (and implemented) so it may require further adjustments for actual use.
```rust

use crate::number::NumberInputConstraints;
use crate::text::TextInputConstraints;
use std::fmt::{Debug, Display};

pub type ValidationMessage = String;
pub type ValidationResultError = (ValidationResultEnum, ValidationMessage);
pub type ValidationResult = Result<(), ValidationResultError>;

pub trait InputConstraints<T: Clone + Debug + Display + PartialEq>: Debug {
  fn validate(&self, x: T) -> Result<(), ValidationResultError>;
}

pub enum Constraints<'a, T> {
  TextInput(TextInputConstraints<'a>),
  NumberInput(NumberInputConstraints<'a, T>),
}

#[derive(PartialEq, Debug)]
pub enum ValidationResultEnum {
  CustomError,
  PatternMismatch,
  RangeOverflow,
  RangeUnderflow,
  StepMismatch,
  TooLong,
  TooShort,
  TypeMismatch, 
  Valid,
  ValueMissing,
}

```

Example (partial) struct declaration *figure 4*
```rust
use crate::types::ValidationResultEnum::{
  CustomError, RangeOverflow, RangeUnderflow, Valid, ValueMissing,
};
use crate::types::{InputConstraints, ValidationResultEnum, ValidationResultError};
use std::borrow::Borrow;
use std::fmt::{Debug, Display, Formatter};
use std::sync::Arc;

#[derive(Clone)]
pub struct NumberInputConstraints<'a, T> {
  pub min: Option<T>,
  pub max: Option<T>,
  pub step: Option<T>,
  pub required: bool,
  pub custom:
    Option<Arc<&'a (dyn Fn(&NumberInputConstraints<T>, T) -> bool + Send + Sync)>>,

  pub range_underflow:
    Arc<&'a (dyn Fn(&NumberInputConstraints<T>, T) -> String + Send + Sync)>,
  pub range_overflow:
    Arc<&'a (dyn Fn(&NumberInputConstraints<T>, T) -> String + Send + Sync)>,
  pub value_missing:
    Arc<&'a (dyn Fn(&NumberInputConstraints<T>, T) -> String + Send + Sync)>,
  pub custom_error:
    Arc<&'a (dyn Fn(&NumberInputConstraints<T>, T) -> String + Send + Sync)>,
}
```

### Example implementations

- TextInput
- NumberInput
- etc.

Pros: 

Fewer structures to deal with - TextInput contains the properties required for dealing with any string type (there are 'custom*' and 'pattern' fields declared here which can be used for enforcing value types of different formats).

Cons:

- Object-oriented-like approach (fields in structs can be malleable depending on implementation).

Quick note: notice how this is the same as if we had used a builder pattern struct, and/or validator getter, for acquiring a validator;  Validators, and/or Validator Structs, are really the same thing (with caveats only on their implementations). 

