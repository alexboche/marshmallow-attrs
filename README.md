# marshmallow_attrs
[![Build Status](https://travis-ci.org/adamboche/marshmallow-attrs.svg?branch=master)](https://travis-ci.org/adamboche/marshmallow-attrs)
[![PyPI version](https://badge.fury.io/py/marshmallow-attrs.svg)](https://badge.fury.io/py/marshmallow-attrs)



Automatic generation of [marshmallow](https://marshmallow.readthedocs.io/) schemas from dataclasses.

This package is based on [marshmallow-dataclass](https://github.com/lovasoa/marshmallow_dataclass).


Specifying a schema to which your data should conform is very useful, both for (de)serialization and for documentation.
However, using schemas in python often means having both a class to represent your data and a class to represent its schema, which means duplicated code that could fall out of sync. With the new features of python 3.6, types can be defined for class members, and that allows libraries like this one to generate schemas automatically.

An use case would be to document APIs (with [flasgger](https://github.com/rochacbruno/flasgger#flasgger), for instance) in a way that allows you to statically check that the code matches the documentation.

## How to use

You simply import
[`marshmallow_attrs.dataclass`](https://adamboche.github.io/marshmallow_attrs/html/marshmallow_attrs.html#marshmallow_attrs.dataclass)
instead of
[`attr.dataclass`](http://attrs.org).
It adds a `Schema` property to the generated class,
containing a marshmallow
[Schema](https://marshmallow.readthedocs.io/en/2.x-line/api_reference.html#marshmallow.Schema)
class.

If you need to specify custom properties on your marshmallow fields
(such as `attribute`, `error`, `validate`, `required`, `dump_only`, `error_messages`, `description` ...)
you can add them using the `metadata` argument of the
[`attr.ib`](http://www.attrs.org/en/stable/api.html#attr.ib)
function.

```python
import attr
from marshmallow_attrs import dataclass # Importing from marshmallow_attrs instead of attrs
import marshmallow.validate
from typing import List, Optional

@dataclass
class Building:
  # The field metadata is used to instantiate the marshmallow field
  height: float = field(metadata={'validate': marshmallow.validate.Range(min=0)})
  name: str = field(default="anonymous")


@dataclass
class City:
  name: Optional[str]
  buildings: List[Building] = field(default_factory=lambda: [])

# City.Schema contains a marshmallow schema class
city, _ = City.Schema().load({
    "name": "Paris",
    "buildings": [
        {"name": "Eiffel Tower", "height":324}
    ]
})

# Serializing city as a json string
city_json, _ = City.Schema().dumps(city)
```

The previous  syntax is very convenient, as the only change
you have to apply to your existing code is update the
`dataclass` import.

However, as the `.Schema` property is added dynamically,
it can confuse type checkers.
If you want to avoid that, you can also use the standard
`attr.s` decorator, and generate the schema manually
using
[`class_schema`](https://adamboche.github.io/marshmallow_attrs/html/marshmallow_attrs.html#marshmallow_attrs.class_schema)
:

```python
import attr


from datetime import datetime
import marshmallow_attrs

@attr.dataclass
class Person:
    name: str
    birth: datetime

PersonSchema = marshmallow_attrs.class_schema(Person)
```

You can also declare the schema as a
[`ClassVar`](https://docs.python.org/3/library/typing.html#typing.ClassVar):

```python
from marshmallow_attrs import dataclass
from marshmallow import Schema
from typing import ClassVar, Type

@dataclass
class Point:
  x:float
  y:float
  Schema: ClassVar[Type[Schema]] = Schema
```

You can specify the
[`Meta`](https://marshmallow.readthedocs.io/en/3.0/api_reference.html#marshmallow.Schema.Meta)
just as you would in a marshmallow Schema:

```python
from marshmallow_attrs import dataclass

@dataclass
class Point:
  x:float
  y:float
  class Meta:
    ordered = True
```

## Installation
This package [is hosted on pypi](https://pypi.org/project/marshmallow-attrs/) :

```shell
pip install marshmallow-attrs
```

## Documentation

The project documentation is hosted on github pages:
 - [documentation](https://adamboche.github.io/marshmallow_attrs/).

## Usage warning

This library depends on python's standard
[typing](https://docs.python.org/3/library/typing.html)
library, which is
[provisional](https://docs.python.org/3/glossary.html#term-provisional-api).


## Credits

This package is based on [marshmallow-dataclass](https://github.com/lovasoa/marshmallow_dataclass).
