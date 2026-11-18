# Xeption for Python

An exception that carries its data. The Python port of [Xeption](https://github.com/The-Standard-Organization/Xeption),
the library [The Standard](https://github.com/hassanhabib/The-Standard) uses to collect validation
errors before breaking the circuit, and to compare exceptions by value in tests.

The C# library decides the behaviour; this package decides how it reads in Python. It has no
runtime dependencies.

```bash
python -m pip install xeption
```

## Collecting validation errors

A validation routine upserts every failure it finds into one exception, then raises it once:

```python
from xeption import Xeption


class InvalidStudentException(Xeption):
    def __init__(self) -> None:
        super().__init__(message="Invalid student. Please correct the errors and try again.")


def validate_student(student: dict) -> None:
    invalid_student_exception = InvalidStudentException()

    if not student.get("name"):
        invalid_student_exception.upsert_data_list("name", "Text is required")

    if not student.get("email"):
        invalid_student_exception.upsert_data_list("email", "Text is required")

    invalid_student_exception.throw_if_contains_errors()
```

`data` maps each key to the list of messages collected for it, so the caller sees every error at
once rather than the first.

## Wrapping an exception

A foundation localizes a native exception by wrapping it. The inner exception is kept as
`inner_exception` and is also the `__cause__`, so tracebacks show the chain:

```python
class FailedStudentStorageException(Xeption):
    def __init__(self, inner_exception: BaseException) -> None:
        super().__init__(
            message="Failed student storage error occurred, contact support.",
            inner_exception=inner_exception)
```

`add_data` copies another exception's data in, which is how a dependency's validation details
travel up with the local exception.

## Comparing exceptions in tests

`same_exception_as` is equivalence, not identity: the same type, message and data, all the way down
the chain of inner exceptions. A native inner exception carries no data, so its type and arguments
are what make it the same.

```python
assert actual_exception.same_exception_as(expected_exception)
```

## API

| Python | C# |
|---|---|
| `Xeption(message="", inner_exception=None)` | `Xeption()`, `Xeption(message)`, `Xeption(message, innerException)` |
| `upsert_data_list(key, value)` | `UpsertDataList(key, value)` |
| `add_data(data)` | `AddData(IDictionary)` |
| `throw_if_contains_errors()` | `ThrowIfContainsErrors()` |
| `same_exception_as(other)` | `SameExceptionAs(otherException)` |

### Not ported yet

These parts of the C# library have no Python counterpart in this version:

- the constructors that take a data dictionary, and `AddData(key, params values)`;
- `DataEquals` and `DataEqualsWithDetail`, and the `out message` overload of `SameExceptionAs`
  that explains a mismatch;
- `AggregateException` comparison (Python's `ExceptionGroup`);
- `IsFrom` and `IsNotFrom`;
- the FluentAssertions extensions.

## The Standard

This library is part of [The Standard](https://github.com/hassanhabib/The-Standard)
family and is built to its rules for naming, layering and testing.

Questions, ideas and contributions are welcome in
[The Standard Community](https://discord.gg/vdPZ7hS52X) on Discord.

## In other languages

| Language | Repository | Package |
|---|---|---|
| C# | [Xeption](https://github.com/The-Standard-Organization/Xeption) | [Xeption on NuGet](https://www.nuget.org/packages/Xeption) |
| Python | [Xeption-Python](https://github.com/hassanhabib/Xeption-Python) | [xeption on PyPI](https://pypi.org/project/xeption/) |

## Contributing

See [CONTRIBUTING.md](https://github.com/hassanhabib/Xeption-Python/blob/main/CONTRIBUTING.md).

## License

[The Standard Software License (TSSL) v1.1](https://github.com/hassanhabib/Xeption-Python/blob/main/LICENSE.txt).
