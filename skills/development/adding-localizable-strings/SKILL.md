---
name: adding-localizable-strings
description: Adds new human-readable strings that are translated into users' languages.
---

# Adding Localizable Strings

## Instructions

Copy this checklist and check off items as you complete them.

```
Task Progress:
- [ ] Add the strings to the English strings file.
- [ ] Run `yarn translate` to translate to other languages.
- [ ] Run the full test suite with `./gradlew test`.
```

### Step 1: Add the strings to the English strings file

You will usually want to add strings to `src/main/resources/i18n/Messages_en.properties`.

Each string may be preceded with a comment line (starting with `#`). The comment is provided to the translation service and can provide additional context about the string to help produce grammatically-correct translations. This is optional and doesn't need to be provided if the English string is a simple, unambiguous word or phrase.

Add strings in alphabetical order by key. But don't insert a string in between a comment line and the existing string the comment is about. For example, if you see

    stringX=Foo
    # Comment for string Z
    stringZ=Bar

and you are adding `stringY`, put it above string Z's comment, like

    stringX=Foo
    stringY=Why
    # Comment for string Z
    stringZ=Bar

Placeholders should be numbered starting with 0, enclosed in curly braces, like `{0}`.

Strings are rendered using Java's `MessageFormat`, which treats single quote characters (`'`) as special. If the English text contains single quotes, you must escape them by doubling them. For example, the word "can't" would need to be `can''t` in the file.

Stick to ASCII characters for punctuation.

### Step 2: Run "yarn translate" to translate to other languages

Only add strings to the English strings file. The strings files for other languages include metadata that you won't be able to generate yourself.

We have an automated translation tool that updates the files for other languages, including adding the right metadata.

Run `yarn translate` to update the non-English strings files with translations for newly-added and modified English strings.

### Step 3: Run the test suite

There are various tests that do sanity checking of the strings files. Run the full test suite to make sure you catch any problems.

If you're adding strings as part of a larger task, there's no need to run the full test suite multiple times; you can wait until you're done with the larger task.
