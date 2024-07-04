# Notes on `dshash`

## When to use it?

Unlike [`dynahash`](https://doxygen.postgresql.org/dynahash_8c_source.html),
[`dshash`](https://doxygen.postgresql.org/dshash_8c_source.html) is always in
shared memory (dynahash _can_ be put in shared memory) it allows the hashtable
to grow (though be careful, it can't shrink).

## Differences with `dynahash`

While the behaviours below are documented in `dshash`, it's worth highlighting
them as they are notably different from those of `dynahash`.


### Sequence iteration

`dshash_seq_term` **must** be called when the scan is finished (in dynahash,
its `hash_seq_term` counterpart must only be called if the scan is
abandoned before the termination of the scan).

> [!NOTE]
> Postgres versions before 15 do not have support for sequential scans. Current
> version's comments suggest that it still doesn't: "Future versions may
> support iterators" but this appears to be untrue considering presence of
> `dshash_seq_` family of functions

### Release entry locks

When you are done with an entry returned by `dshash_find` or similar functions,
make sure you release the lock held by entry using `dshash_release_lock`.
