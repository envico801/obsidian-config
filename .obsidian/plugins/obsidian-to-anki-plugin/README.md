# Changes made

The changes were made in [main.js](https://github.com/envico801/obsidian-config/blob/main/.obsidian/plugins/obsidian-to-anki-plugin/main.js) and serve to work with the [conversion script](https://github.com/envico801/obsidian-to-anki-card-converter) and/or fix indentation problems.

These 3 changes go together

1. This avoids selecting whitespaces that are possibly behind the start of a code block.

    ```javascript
    // @#@# text = text.replace(/(?:^|\n)(?: {0,3})(```+|~~~+)(?: *)([^\s`~]*)\n([\s\S]*?)\n(?: {0,3})\1/g, function (wholeMatch, delim, language, codeblock) {
    text = text.replace(/(```+|~~~+)(?: *)([^\s`~]*)\n([\s\S]*?)\n*\1/g, function (wholeMatch, delim, language, codeblock) {
    ```

2. This is to avoid adding 2 new lines in front of a captured code block, to keep the previous ident (for example if we are in a list with indentation).

    ```javascript
    // @#@# return '\n\n¨G' + (globals.ghCodeBlocks.push({text: wholeMatch, codeblock: codeblock}) - 1) + 'G\n\n';
    return '¨G' + (globals.ghCodeBlocks.push({text: wholeMatch, codeblock: codeblock}) - 1) + 'G\n\n';
    ```

3. This modifies how lists are selected, it allows to select everything with the same indentation, including code blocks.

    ```javascript
    // @#@# text = text.replace(/(\n\n|^\n?)(( {0,3}([*+-]|\d+[.])[ \t]+)[^\r]+?(¨0|\n{2,}(?=\S)(?![ \t]*(?:[*+-]|\d+[.])[ \t]+)))/gm,
    text = text.replace(/(\n\n|^\n?)(( {0,3}([*+-]|\d+[.])[ \t]+)[^\r]+?(¨0|\n{2,}(((\s{3,}¨G\dG\n{1,}\s{3,}[\s\S]*?)(?=(\n^\S)))|(?=\S))(?![ \t]*(?:[*+-]|\d+[.])[ \t]+)))/gm,
    ```

This prevents the plugin from highlighting any part of the text, for now I don't use it. 

```javascript
// @#@# const HIGHLIGHT_REGEXP = /==(.*?)==/g;
const HIGHLIGHT_REGEXP = /@#@#@#@/g;
```

This changes the way mathematical expressions are selected, they now use the `§` symbol instead of the `$` symbol *(conflicts with javascript 💀)*.

```javascript
// const OBS_INLINE_MATH_REGEXP = /(?<!\$)\$((?=[\S])(?=[^$])[\s\S]*?\S)\$/g;
const OBS_INLINE_MATH_REGEXP = /(?<!\§)\§((?=[\S])(?=[^$])[\s\S]*?\S)\§/g;
```
```javascript
// const OBS_DISPLAY_MATH_REGEXP = /\$\$([\s\S]*?)\$\$/g;
const OBS_DISPLAY_MATH_REGEXP = /\§\§([\s\S]*?)\§\§/g;
```

Check the [configuration folder](https://github.com/envico801/obsidian-config/tree/main/Configuration%20of%20individual%20plugins/obsidian-to-anki-plugin) to know what changes to take into account.
