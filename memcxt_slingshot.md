# MemoryContext slingshot

There are cases when we'd like to have a callback, but Postgres doesn't provide
one. Fortunately, in some cases, there is still a way to do this.

When `MemoryContext` is deleted or reset, you can use this as a "slingshot" to
inject your callback there.

Examples of such use:

* Upon `PostmasterContext` deletion, initialize the backend
* When transaction is committed **and** [visible to other backends](https://github.com/postgres/postgres/blob/8fea1bd5411b793697a4c9087c403887e050c4ac/src/backend/access/transam/xact.c#L2401-L2431), `TopTransactionContext` is reset.
  Transaction callbacks don't get us that far.

```c
MemoryContextCallback *cb = MemoryContextAlloc(/* TargetMemoryContext */, sizeof(*cb));
cb->func = /* callback function */;
cb->arg = /* function argument */
MemoryContextRegisterResetCallback(/* TargetMemoryContext */, cb);
```

> [!CAUTION]
> Newly registered callbacks will be called before those registered earlier, effectively reversing the order in which the callbacks are called.
