# Catching specific generic Errors in TypeScript

>*You can jump to the TL;DR to skip the discussion (and the  toilet humour, because mentally I am five).*

Everyone knows how to catch a specific error in TypeScript:

```typescript
try {
	blerk = something.thatMight(throw);
}
catch (e) {
	if (e instanceof FartError) {
		console.log('oh no somone farted');
	}
	else {
		throw e;
	}
}
```

This will allow us to handle the specific error we anticipate without hiding errors we haven't anticipated. Groovy. And if you google "How to catch a specific error in Typescript" this is [basically what the results will tell you](https://timmousk.com/blog/typescript-try-catch/#how-to-catch-a-specific-error).

## The problem

What if you want to be responsible and only catch a specific error that you have a contingency plan for, but the code you're calling doesn't inherit from Error as a way of making the error more specific. This is what I was dealing with:

```typescript
import fs from 'node:fs/Promises';

try {
	await f = fs.readFile('notafile.weasel');
}
catch (e) {
	// here, e is Error({
	//	message: ['Error: ENOENT: no such file or directory,... ']
	//	errno: -2,
	//	code: 'ENOENT',
	//	syscall: 'open',
	//	path: '/code/notafile.weasel'
	// });
}
```

I'm new to TypeScript so I wasn't sure how to deal with this. `e` is `unknown` when we enter a catch block, so if you do this the compliler complains:

```typescript
catch (e) {
	if (e.code === 'ENOENT') { 
		// error TS18046: 'e' is of type 'unknown'.
		console.log('the filesystem passed wind');
	}
}
```

We can reasonbly expect that `e` will be an `Error()` when we enter the catch block so you might try:

```typescript
catch (e) {
	if (e isinstance Error && e.code === 'ENOENT') {
		// error TS2339: Property 'code' does not exist on type 'Error'
		console.log('prrbt');
	}
}
```

The only thing we know from knowing e is an Error is that it has property Message, and parsing the type of error from that is ugly and bad. We're just interested in `e.code`. So how do we test if the error has a property `code`?

```typescript
if ('code' in e && e.code === 'ENOENT') { 
	//    ^ error TS18046: 'e' is of type 'unknown'.
	console.log('you get the idea');
}
```
At this point I'm starting to feel like I've missed a trick. I'm using [Zod](https://github.com/colinhacks/zod) in this project, so I think about validating the error to a custom type, `HasCode`, but that sounds like ridiculous overkill. Typescript must have a solution to this that doesn't suck.

As I later worked out, the reason this throws a Typescript error is that because if `e` is `unknown`, then `e` may be `null` (or a string, or...) and if you try to check if `'property' in null` at runtime, things blow up. What you can safely do is check first whether `e` is the sort of thing that has properties i.e. is `e` an `Object`?

```typescript
if (e instanceof Object && 'code' in e && e.code === 'ENOENT') {
	console.log('oh god that one was wet');
}
// > oh god that one was wet
```

This works great! First we, check that e is the sort of thing that has properties. Then, we know it has properties, so we see if 'code' is among them. Then, given we know it has properties, and that 'code' is one of them, we can safely access that property and use it for our comparison. Ahhh. 

Writing this post, I worked out that you don't need to go so generic -- this also works:
```typescript
if (e instanceof Error && 'code' in e && e.code === 'ENOENT') {
	console.log("ok i'm done");
}
// > ok i'm done
```

This is the type-safe way to do it. You could cast `e` as `Any` when it is declared in the catch statement:

```typescript
catch (e: Any) {...}
```

but this is forbidden in `strict` type checking mode for good reason (even though it was the default behaviour in earlier versions of TypeScript) because it completely disables type-checking. 

## TL;DR

If you want to specifically catch a generic Error with custom properties do something like the below:

```typescript
import fs from 'node:fs/promises';

try {
    await fs.readFile('baka');
}
catch (e) {
    if (e instanceof Error && 'code' in e && e.code === 'ENOENT') {
        console.log('Thanks for reading!!');
    }
}
// > Thanks for reading!!
```
Bye!
