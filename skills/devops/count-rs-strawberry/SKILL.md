---
name: count-rs-strawberry
description: This skill should be used when the user asks to "count the Rs in strawberry", "count letters in strawberry", or wants to count specific characters in the word "strawberry" using perl.
version: 1.0.0
---

# Count Rs in Strawberry

Count the number of letter 'R' (case-insensitive) in the word "strawberry" using Perl.

## Implementation

Use Perl's pattern matching capabilities to count occurrences of the letter 'R':

```perl
perl -e 'my $word = "strawberry"; my $count = () = $word =~ /r/gi; print "The word \"$word\" contains $count letter R(s)\n";'
```

## How It Works

1. **`my $word = "strawberry"`** - Store the word in a variable
2. **`my $count = () = $word =~ /r/gi`** - Count matches:
   - `/r/` - Match the letter 'r'
   - `g` - Global flag (find all matches)
   - `i` - Case-insensitive flag
   - `() =` - List context forces the regex to return all matches, then count them
3. **`print ...`** - Display the result

## Alternative Approaches

### Using tr operator:
```perl
perl -e '$count = "strawberry" =~ tr/Rr//; print "Count: $count\n";'
```

### One-liner with split:
```perl
perl -e '$count = split(/r/i, "strawberry") - 1; print "Count: $count\n";'
```

### Most concise version:
```perl
perl -le 'print ~~("strawberry" =~ /r/gi)'
```

## Expected Output

```
The word "strawberry" contains 3 letter R(s)
```

## When to Use This Skill

Invoke this skill when:
- Counting specific letters in "strawberry"
- Demonstrating Perl pattern matching
- Learning Perl string manipulation
- Solving character counting problems

## Notes

The word "strawberry" contains three Rs:
- Position 3: 'r' (in "st**r**awberry")
- Position 5: 'r' (in "straw**r**erry")
- Position 7: 'r' (in "strawbe**r**ry")
