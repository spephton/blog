# Catching specific Errors without type information in TypeScript

Everyone knows how to catch a specific error in Typescript

```
try {
	blerk = something.that_might(throw);
}
catch (e) {
	if (e instanceof FartError) {
		console.log('oh no somone farted');
}
```

This will allow us to handle the specific error we anticipate without hiding errors we haven't anticipated. Groovy. And if you google "How to catch a specific error in TypeScript" this is basically what the results will tell you.

## The problem

What if you want to be responsible and only catch a specific error that you have a contingency plan for, but the code you're calling doesn't subclass Error as a way of making the error more specific. This is what I was dealing with:

```
import fs from 'node:fs/Promises';

try {
	await filecontents = fs.readFile('file_that_is_not_present.weasel');
}
catch (e) {
	// here, e is Error({
	//	message: ['Error: ENOENT: no such file or directory,... ']
	//	errno: -2,
	//	code: 'ENOENT',
	//	syscall: open,
	//	path: /Users/spephton/coooooode/project/file_that....
	// });
}
```

I'm new to TypeScript so I wasn't sure how to deal with this. `e` is unknown when we enter a catch block, so if you do this the compliler complains:

```
catch (e) {
	if (e.code === 'ENOENT') { // error TS18046: 'e' is of type 'unknown'.
		console.log('the filesystem passed wind');
	}
}
```

We can reasonbly expect that e will be an Error() when we enter the catch block so you might try:

```
catch (e) {
	if (e isinstance Error && e.code === 'ENOENT') {
		// error TS2339: Property 'code' does not exist on type 'Error'
		console.log('prrbt');
	}
}
```

The only thing we know from knowing e is an Error is that it has property Message, and parsing the type of error from that is ugly and bad. We're just interested in the code. So how do we test if the error has a property 'code'?

```
if ('code' in e && e.code === 'ENOENT') { 
	//    ^ error TS18046: 'e' is of type 'unknown'.
	console.log('you get the idea');
}
```
At this point I'm starting to feel like I've missed a trick. I'm using Zod in this project so I think about validating to a custom type HasCode but that sounds like ridiculous overkill. Typescript must have a solution to this that doesn't suck.

As I later worked out, the reason this throws a typescript error is that because if `e` is `unknown`, then `e` may be `null` and if you try to check if `'property' in null` at runtime things blow up. What you can safely do is check first whether e is the sort of thing that has properties i.e. is e an Object?

```
if (e instanceof Object && 'code' in e && e.code === 'ENOENT') {
	console.log('oh god that one was wet');
}
// >oh god that one was wet
```

This works great! First we, check that e is the sort of thing that has properties. Then, we know it has properties, so we see if 'code' is among them. Then, given we know it has properties, and that 'code' is one of them, we can safely access that property and use it for our comparison. Ahhh. 

Writing this post I worked out that you don't need to go so generic -- this also works:
```
if (e instanceof Error && 'code' in e && e.code === 'ENOENT') {
	console.log('ok i'm done');
}
// >ok i'm done
```

## TL;DR

If you want to specifically catch a generic Error with custom properties do something like the below:

```
import fs from 'node:fs/promises';

try {
    await fs.readFile('baka');
}
catch (e) {
    if (e instanceof Error && 'code' in e && e.code === 'ENOENT') {
        // do somehting
        console.log('Thanks for reading!!');
    }
}
// > Thanks for reading!!
```
Bye!