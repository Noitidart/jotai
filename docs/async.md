This doc describes about the behavior with async.

## Some notes

- You need to wrap components with `<Suspense>` inside `<Provider>`.
- You can have as many `<Suspense>` as you need.
- If the `read` function of an atom returns a promise, the atom will suspend.
- This applies to dependent atoms too.
- If a primitive atom has a promise as the initial value, it will suspend at the first use (when Provider doesn't have it.)
- You can create a `read` function so that it works asynchronously, but does not return a promise. In such a case, the atom won't suspend.
- If the `write` function of an atom returns a promise, the atom will suspend. There's no way to know as of now if an atom suspends because of `read` or `write`.
- You can create a `write` function so that it works asynchronously, but does not return a promise. In such a case, the atom won't suspend. (Caveat: This means you can't catch async errors. So, it's not recommended.)

### Understand how to implement async atoms that trigger `Suspense` fallback


Recap for anyone who runs into this (correct me if I'm wrong):

**An atom will trigger the `Suspense` fallback only if:**

    * the atom's write arg (2nd one) is async

    * the awaited call is made directly and not from inside another containing function


This _will_ trigger the `Suspense` fallback:

```ts
const writeAtom = atom(null, async (get, set) => {
   const response = await new Promise<string>((resolve, _reject) => {
    setTimeout(() => {
      resolve("some returned value");
    }, 2000);
  });
  set(somePrimitiveAtom, "The returned value is: " + response);
})
```

This _will not_ trigger the `Suspense` fallback:

```ts
const writeAtom = atom(null, (get, set) => {
   const getResponse = async () => {
      const response = await new Promise<string>((resolve, _reject) => {
        setTimeout(() => {
          resolve("some returned value");
        }, 2000);
      });
      set(somePrimitiveAtom, "The returned value is: " + response);
   }
  getResponse();
})
```

But **_both_** of the above will still properly set `somePrimitiveAtom` to the correct values.
