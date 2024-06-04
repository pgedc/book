# PG_CATCH when holding locks

Imagine you have a scenario where you acquire and hold some lwlocks before
`PG_TRY/PG_CATCH` block, and then you try to release the locks in `PG_CATCH` as
part of your cleanup. Innocent enough?

However, you are likely to get this:

```
TRAP: failed Assert("InterruptHoldoffCount > 0")
```

That’s because `errfinish` sets it to zero, but releasing lwlocks requires the
count to be greater than zero, as you can see above.

However, this is what `errfinish` implementation tells us:

```c
/*
* We do some minimal cleanup before longjmp'ing so that handlers can
* execute in a reasonably sane state.
*
* Reset InterruptHoldoffCount in case we ereport'd from inside an
* interrupt holdoff section.  (We assume here that no handler will
* itself be inside a holdoff section.  If necessary, such a handler
* could **save and restore InterruptHoldoffCount** for itself, but this
* should make life easier for most.)
*/
InterruptHoldoffCount = 0;
```

So, we can perhaps use this as an invitation to save `InterruptHoldoffCount`
before going into `PG_TRY`, even though it doesn't feel perfect – if you really
need this.
