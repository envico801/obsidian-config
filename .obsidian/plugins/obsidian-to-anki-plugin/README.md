# Changes made

The changes were made in main.js and serve to work with the conversion script and/or fix indentation problems.

```javascript
// @#@# text = text.replace(/(?:^|\n)(?: {0,3})(```+|~~~+)(?: *)([^\s`~]*)\n([\s\S]*?)\n(?: {0,3})\1/g, function (wholeMatch, delim, language, codeblock) {
text = text.replace(/(```+|~~~+)(?: *)([^\s`~]*)\n([\s\S]*?)\n(?: {0,3})\1/g, function (wholeMatch, delim, language, codeblock) {
```

```javascript
// @#@# return '\n\n¨G' + (globals.ghCodeBlocks.push({text: wholeMatch, codeblock: codeblock}) - 1) + 'G\n\n';
return '¨G' + (globals.ghCodeBlocks.push({text: wholeMatch, codeblock: codeblock}) - 1) + 'G\n\n';
```

```javascript
// @#@# text = text.replace(/(\n\n|^\n?)(( {0,3}([*+-]|\d+[.])[ \t]+)[^\r]+?(¨0|\n{2,}(?=\S)(?![ \t]*(?:[*+-]|\d+[.])[ \t]+)))/gm,
text = text.replace(/(\n\n|^\n?)(( {0,3}([*+-]|\d+[.])[ \t]+)[^\r]+?(¨0|\n{2,}(((\s{3,}¨G\dG\n{1,}\s{3,}[\s\S]*?)(?=(\n^\S)))|(?=\S))(?![ \t]*(?:[*+-]|\d+[.])[ \t]+)))/gm,
```

```javascript
// @#@# const HIGHLIGHT_REGEXP = /==(.*?)==/g;
const HIGHLIGHT_REGEXP = /@#@#@#@/g;
```

Check the configuration folder to know what changes to take into account.