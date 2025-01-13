---
title: "Reg Expression in JavaScript"
date: 2024-10-07
description: "A note starting from basics of Reg Expressions in javascript. Explains every matchers, flags and methods"
tags:
  - javascript
  - regex
---

# RegExpressiong Javascript

### Matchers

1. /string/ - use to match a word in the string. It is Case Sensitive matching.

2. pipe Operator(|) - it is a or operator (similar to typescript) where you can put different matchers in one experession, like creating a union in typescript.
   e.g. _/foo|bar|baz|/_ - this will find a match using(_//_) and will check for _foo_ , _bar_, _baz_.

3. wildcard operator (.) - way to return any _letter_ when combine in a matchers. Every `.` dot represents a letter.
   e.g. `/ba./` this will match any _3_ letters that starts with _ba_ while the third letter is any letter, like bar,baz will matched here.

4. predefined matching ([]) - it is like a pipe operator for _letters_. You can think of it as a value in the arrays but with no separator all the letters inside the bracket is considered as its own letter matcher so even if you put `[help]` this will find `h,e,l,p`. all the letters inside this bracket(array) will be matched.
   e.g. `/b[aeiou]r/` - this will match all vowels between b and r (bar,bir,ber,bor,bur)

   - range operator (-) - this can be used inside the predefined operator. What this do is will match a range of letters or numbers and/or combination of both instead of passing/typing it one by one.
     e.g. `/[f[a-z]o]/i` - this will match a case insensitive letter from a to z between f and o.
     e.g. `/b[a-e3-9]r` - this will match a letter from **a to e** and **3-9**.

   > **About Range Operator**,
   > notice that inside the predefined operator with multiple range operator you dont need a separator. regEx is smart enough to separate the expressions

5. negate characters set or match first character set (^) - the carat operator. is a way to negate or ignore the characters defined inside the bracket operator but if not used together with bracker operator it will become a first character matcher.
   e.g. `/f[^aeiou1-9]o/` - this will match all the letters except the vowels and the number 0.
   e.g. `/[c-z^aeiou0-9]r` - this will match letters starting from c to z but excludes vowels and _b_ and numbers . _b_ is not in the matching range
   e.g `/^he/i/` - the carat operator here is being used without the predefined operator thus it will check the string starts with _h_ instead.

6. match last character set ($) - while _^_ is being used to match the first character set of the string if not being used together with predefined matcher. the $ is use to match the end of the character set.

> **about ^ and $**, -->
> ^ must be use at the very first matcher to check for the first character set while $ is at the very last
> if the ^ and $ is used together it means that match that exact regEx. /^foo$/

7. shorthand operators

   - (**\w**) - it is the same to `/[a-z0-9_]/i` where it will match all the letters and numbers plus underscore.
     e.g. `/\w/g` will match all the occurences of letters, numbers and underscore
   - (**\W**) - is the opposite. it will match that is not letters,numbers or underscore.
     e.g. `/\W/g` will match all the occurences that is not letters, numbers and underscore
   - (**\d**) - is the matcher of only the numbers. _d_ stands for digits
     e.g. `/\d+/g` this will match all the numbers but will separate match the numbers with decimals. "5.00" will match 5 and 00 separately.
   - (**\D**) - is the opposite of the number shothands. this will match a none numbers.
     e.g. `/\D/g` this will all the letters, signs but not the numbers.
   - (**\s**) - this will match as all special characters including whitespce and line breaks.
     e.g. `/\s/g`
   - (**\S**) - this on other hand will match all special characters but not including the whitespce and line breaks.
   - (**\1**) - 1 can be any number base on the capture group available. this is a shorthand to repeat the existing capture group. the number passed here is the capture group sequence in the regEx. if there are multiple capture group like 3 of them you can repeat and choose the capture group.
     e.g. `/(\w+)\s(\d+)\s\2/` - this reads as the all the characters and a special then all the digits and special character and all the digits (\2) repeat the second capture group

8. plus operator (+) - matches one or more occurences of characters or group of characters.

   e.g. `/s+/g` - this will match all of the occurences of _s_.
   e.g. `/[0-9]+/g` - this will match all the numbers not just one number but the whole numbers, like, `"123 432 123"` will return these number in an array
   e.g. `/(foo+)/g` - this will match all the occurences of _foo_ and returns null if no matched found.

9. 0 or more times matcher (`*`) - the asterisk operator will match one or more occurences of a character. You can consider this as an optional matcher where there may be no match of that character at all.

   e.g. `/fo_/` - this will match the first occurence of _f_ and/or _fooooo_ . notice that it can match _f_ without _o_.

10. curly braces operator (**{}**) - is a indicator/constraint matcher. this will constraint the previous matcher
    on how many times it can match. it takes 2 arguments (like a minmax() in css) **{2,10}**.

- first argument is the min times it repeats
- comma(,) - is the separator of the min and max. _if there is no comma it will be read as the **Exact** value constraint_
- the second is the maximum. _if there is no second argument it will be read as infinite_.

  e.g. `/^[A-Za-z]{2,}\d*$/` - this matcher reads as the first character set must be a letter that is case insensitive and must be atleast _2_ (from the constraint) up to infinity and the end character can have a number (\*) so optional but number is only allowed in the end.

11. lazy matching (?) - by default regEx is doing a greedy match means it will get all the match possibe especially when used in a `*` operator but if you put a `?` will turn to lazy match and will only get the first occurence of that match.
    e.g `/<[a-z]*?>/i` - this will match the first occurence of a `<letter a to z>` like `<h1>` but if there is no `?` it will become `<h1>Hellow</h1>` it will match starting from `<` until the last occurence of `>` so the whole html tag.

    - (?) operator that is not being used with _ operator - if not being used together with _ operator it will become an **optional** operator. SO if \* will match 0 or more occurence, _?_ will match or not match the previous matcher but not occurences.

      e.g. `/hellow?/` - this will match the string "hello" and can also be "hellow" so the _w_ is optional.

12. look ahead - This is a check operation that will check the match base on the condition. It is inside the parenthesis and always start with ?. like so: **(?)** . It has two types positive look ahead **(?=)** and negative look head **(?!)**. This pattern is like a javascript pattern of _!_ for nagative and = operator.

    e.g. `/(?=\w{2})(?=\D*\d{2})` - this will check a match for atleast 2 characters including underscore and should have atleast 2 digits at the end.

13. Capture Group **()** - a regEx that is inside the parenthesis. It is a way to group a regEx together. It is useful with shorthands and also if you want to repeat the said expression.
    e.g `/(\w+)\s\1/` - will match all the character set and then a special character and then another character set

#### flag - is usually the expression that you're putting at the end of your expression to change the behavior of the expression.

- `i` - a ignore case flag. A flag that ignore where it is capital case or not. Just make sure to add it at the _end_ of your matcher. E.g. `/foo/i`
- `g` - a global flag. This flag will match _all_ the matched expressions of the string. E.g.`/bar/ig` this will match all bar in case **insensitive** and match all the result

> **About the Flag**,
> the flags can be combine

### RegEx methods.

> **Legends**
>
> `regExpVariable` - is the regEx  
> `string` is the string source

1. regExpVariable.test(string) - `test` is the method that returns a _boolean_ that will check if the regExp found the match or not.
2. string.match(regExpVariable) - `match` is the method that returns the matched expression.
3. string.replace(regExpVariable, stringtoReplace) - is a method to replace the matched regEx to a string indicated in the second parameter. The second argument can also use the capture group from the matched regEx.

   e.g. "hello world".replace(/(w+)/, "hi") - this will output "hi world" as the first capture group will get the hello and stops there since after hellow is a whitespace and replace it with "hi".
   e.g. "foo bar".replace(/(\w+)\s(\w+)/, "$2 $1") - this will return as "bar foo". the $ sign here will be like get the match capture group of n (where n is the number of capture group)

   > **Remember!** ,
   >
   > when using a capture group for the second parameter. the shorthand capture group repeat will not work as you might expect. like **/(\w+)\s\1/** and use it as "$2 $1" this will be considered as "$1 $1". since you just copy the first capture group regEx will also just copy the first capture group, thus you must create the second capture group to work as you expect.
