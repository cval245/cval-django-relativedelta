# django-relativedelta

A Django field for the [`dateutil.relativedelta.relativedelta`](http://dateutil.readthedocs.io/en/stable/relativedelta.html) class,
which conveniently maps to the [PostgresQL `INTERVAL` type](https://www.postgresql.org/docs/current/static/datatype-datetime.html#DATATYPE-INTERVAL-INPUT).

The standard [Django `DurationField`](https://docs.djangoproject.com/en/1.10/ref/models/fields/#durationfield)
maps to [Python's `datetime.timedelta`](https://docs.python.org/3/library/datetime.html#datetime.timedelta), which
has support for days and weeks, but not for years and months.  And if you try to read an `INTERVAL` that contains
months anyway, information is lost because each month gets converted to 30 days.

You should use this package when you need to store payment intervals
(which tend to be monthly or quarterly), publication intervals (which
can be weekly but also monthly) and so on, or when you simply don't
know what the intervals are going to be and want to offer some
flexibility.

If you want to use more advanced recurring dates, you should consider
using [django-recurrence](https://github.com/django-recurrence/django-recurrence)
instead.  This maps to the [`dateutil.rrule.rrule`](http://dateutil.readthedocs.io/en/stable/rrule.html)
class, but it doesn't use native database field types, so you can't
perform arithmetic on them within the database.

## Usage

Using the field is straightforward.  You can add the field to your
model like so:

```python
from django.db import models
from relativedeltafield import RelativeDeltaField

class MyModel(models.Model):
	rdfield=RelativeDeltaField()
```

Then later, you can use it:

```python
from dateutil.relativedelta import relativedelta

rd = relativedelta(months=2,days=1,hours=6)
my_model = MyModel(rdfield=rd)
my_model.save()
```

Or, alternatively, you can use a string with the
[ISO8601 "format with designators" time interval syntax](https://www.postgresql.org/docs/current/static/datatype-datetime.html#DATATYPE-INTERVAL-INPUT):

```python
from dateutil.relativedelta import relativedelta

my_model = MyModel(rdfield='P2M1DT6H')
my_model.save()
```

For convenience, a standard Python `datetime.timedelta` object is
also accepted:

```python
from datetime import timedelta

td = timedelta(days=62,hours=6)
my_model = MyModel(rdfield=td)
my_model.save()
```

After a `full_clean()`, the object will always be converted to a
_normalized_ `relativedelta` instance.  It is highly recommended
you use the [django-fullclean](https://github.com/fish-ball/django-fullclean)
app to always force `full_clean()` on `save()`, so you can be
sure that after a `save()`, your fields are both normalized
and validated.


## Limitations and pitfalls

Because this field is backed by an `INTERVAL` column, it neither
supports the relative `weekday`, `leapdays`, `yearday` and `nlyearday`
arguments, nor the absolute arguments `year`, `month`, `day`, `hour`,
`second` and `microsecond`.

The `microseconds` field is converted to a fractional `seconds` value,
which might lead to some precision loss due to floating-point
representation.

Databases other than PostgreSQL are not supported.

For consistency reasons, when a relativedelta object is assigned to a
RelativedeltaField, it automatically calls `normalized()` on
`full_clean`.  This ensures that the database representation is as
similar to the relativedelta as possible (for instance, fractional
days are always converted to hours).
