# Perfect backwards compatibility policy

**TL;DR**: if you write your code using `import narwhals.stable.v1 as nw`, then we promise to
never (intentionally) break your code.

Ever upgraded a package, only to find that it breaks all your tests because of an intentional
API change? Did you end up having to litter your code with statements such as the following?

```python
if parse_version(pdx.__version__) < parse_version('1.3.0'):
    df = df.brewbeer()
elif parse_version('1.3.0') <= parse_version(pdx.__version__) < parse_version('1.5.0'):
    df = df.brew_beer()
else:
    df = df.brew_drink('beer')
```

Now imagine multiplying that complexity over all the dataframe libraries you want to support...

Narwhals offers a simple solution, inspired by Rust editions.

## Narwhals' Stable API

Narwhals implements a subset of the Polars API. What will Narwhals do if/when Polars makes
a backwards-incompatible change? Would you need to update your Narwhals code?

To understand the solution, let's go through an example. Suppose that, hypothetically, in Polars 2.0,
`polars.Expr.cum_sum` was to be renamed to `polars.Expr.cumulative_sum`. In Narwhals, we
have `narwhals.Expr.cum_sum`. Does this mean that Narwhals will also rename its method,
and deprecate the old one? The answer is...no!

Narwhals offers a `stable` namespace, which allows you to write your code once and forget about
it. That is to say, if you write your code like this:

```python
import narwhals.stable.v1 as nw
from narwhals.typing import FrameT

@nw.narwhalify
def func(df: FrameT) -> FrameT:
    return df.with_columns(nw.col('a').cum_sum())
```

then we, in Narwhals, promise that your code will keep working, even in newer versions of Polars
after they have renamed their method. If you used
`narwhals.stable.v2`, then the method would be called `cumulative_sum` instead, and there may
be other breaking changes - but so long as your code uses `narwhals.stable.v1`, those breaking
changes won't affect you.

## `import narwhals as nw` or `import narwhals.stable.v1 as nw`?

Which should you use? In general we recommend:

- When prototyping, use `import narwhals as nw`, so you iterate quickly.
- Once you're happy with what you've got and want to release something production-ready and stable,
  replace `import narwhals as nw` with `import narwhals.stable.v1 as nw`.

## Exceptions

Are we really promising perfect backwards compatibility in all cases, without exceptions? Not quite.
There are some exceptions, which we'll now list. But we'll never intentionally break your code.
Anything currently in `narwhals.stable.v1` will not be changed or removed in future Narwhals versions.

Here are exceptions to our backwards compatibility policy:

- unambiguous bugs. If a function contains what is unambiguously a bug, then we'll fix it, without
  considering that to be a breaking change.
- radical changes in backends. Suppose that Polars was to do something really radical, like remove
  expressions, or pandas were to remove support for categorical data. At that point, we might
  need to rethink Narwhals. However, we expect such radical changes to be exceedingly unlikely.
- type hints, though we aim to keep these stable anyway.
