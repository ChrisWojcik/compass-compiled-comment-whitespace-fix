#compass-compiled-comment-whitespace-fix

When I compile stylesheets using compass, it ignores whitespace in my authored scss 
document and removes some extra linebreaks between the css comments I use for documentation 
to divide the file up into sections. It makes no practical difference except that it 
annoys me. This little snippet helps me fix that.

NOTE: This fix is specific to the formatting of my comments,but you may be able to
modify the regular expression to fit your needs.

NOTE: I'm a total ruby noob, so there may be a better way to do this. I also run 
Windows. I don't think that should affect how sass treats linebreaks, but I could 
be wrong.

## The Problem

Small sample of what I'm talking about:

**My Authored SCSS File:**
```css
/* ==========================================================================
Base styles: opinionated defaults
========================================================================== */

html,
button,
input,
select,
textarea {
    color: #222;
}
```

**What Sass Outputs (expanded):**
```css
/* ==========================================================================
Base styles: opinionated defaults
========================================================================== */
html,
button,
input,
select,
textarea {
  color: #222;
}
```

ARGH!! There's no blank line after the divider comment! How hideous!

But who cares, I should minify the CSS file for production use anyway, you say. 
Yeah...I know, but maybe I don't wanna!

## The (Hacky) Solution
I add this code snippet in at the bottom of my config.rb file. Don't overwrite 
all the other options, obviously. This code will execute any time a stylesheet is
compiled. Again, this particular snippet will only work if you use the style of 
comment divider seen above. It's the style used by the HTML5 Boilerplate, so you 
may use it or something similar already.

```ruby
on_stylesheet_saved do |file|
    if File.exists?(file)
        text = File.read(file)
        replace = text.gsub(/(\/\* ==.*?\*\/)/m, '\1' + "\n")
        File.open(file, "w") { |file| file.puts replace }
    end
end
```

## The Explanation
The snippet works as follows:

1. Uses the built in on_stylesheet_saved hook so it runs whenever a stylesheet is 
compiled.

2. Reads the stylesheet file and places its contents in a variable called text.

3. Uses a regular expression to replace any pattern which matches one of the comment 
blocks with the exact same text PLUS a newline.

4. Opens the same file and writes the new contents.

## The Regular Expression
```ruby
/(\/\* ==.*?\*\/)/m
```

Regex is confusing sometimes (at least to me), so here's what this one does. The forward 
slashes at either end define the pattern. The "m" is a flag for multiline mode 
so patterns stretching across multiple lines can be matched.

The "/*" and "*\" which define a multi-line comment in css must be escaped because 
those are also special characters in regex. In between I specify one space and then 
two "=" signs so as not to match EVERY multi-line comment. ".*?" means zero or more of any 
character, but "non-greedily" meaning it will match the smallest possible pattern. 
(Otherwise we'd grab nearly the entire file between the first and last comment blocks).

The entire pattern is wrapped in parentheses which "captures" it. gsub can then reference 
the captured text with '\1' so that we can append a newline character onto the exact same text.

Again, this requires that you document your code according to this convention. But 
that's life. You may also be able to play around with the regex to fit your needs.

## References
* [Similar-ish solution to a similar problem that inspired me](http://css-tricks.com/compass-compiling-and-wordpress-themes/)
* [An open issue on github discussing how sass treats whitespace when compiling](https://github.com/nex3/sass/issues/161)