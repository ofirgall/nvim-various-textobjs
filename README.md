# nvim-various-textobjs 🟪🔷🟡
Bundle of more than two dozen new text objects for Neovim.

<!-- vale Google.Units = NO -->
> __Note__  
> If you installed the plugin before March 31st and have set your own keymaps,
> you should change your keymappings to call the text objects via Ex-commands `"<cmd>lua require('various-textobjs').textobj(bool)<CR>"`. This makes the text objects dot-repeatable. See the example in the [Configuration Section](#configuration).
<!-- vale Google.Units = YES -->

<!--toc:start-->
- [List of Text Objects](#list-of-text-objects)
- [Installation](#installation)
- [Configuration](#configuration)
- [Advanced Usage](#advanced-usage)
	- [Smart Alternative to `gx`](#smart-alternative-to-gx)
	- [Delete Surrounding Indentation](#delete-surrounding-indentation)
	- [Other Ideas?](#other-ideas)
- [Limitations](#limitations)
- [Other Text Object Plugins](#other-text-object-plugins)
- [Credits](#credits)
<!--toc:end-->

## List of Text Objects
<!-- vale off -->
<!-- LTeX: enabled=false -->

| textobj              | description                                                                               | inner / outer                                                                             | forward-seeking |     default keymaps      | filetypes (for default keymaps) |
|:---------------------|:------------------------------------------------------------------------------------------|:------------------------------------------------------------------------------------------|:----------------|:------------------------:|:--------------------------------|
| indentation          | surrounding lines with same or higher indentation                                         | [see overview from vim-indent-object](https://github.com/michaeljsmith/vim-indent-object) | no              | `ii`, `ai`, `aI`, (`iI`) | all                             |
| restOfIndentation    | lines down with same or higher indentation                                                | \-                                                                                        | no              |           `R`            | all                             |
| subword              | like `iw`, but treating `-`, `_`, and `.` as word delimiters *and* only part of camelCase | outer includes trailing `_`,`-`, or space                                                 | no              |        `iS`, `aS`        | all                             |
| toNextClosingBracket | from cursor to next closing `]`, `)`, or `}`                                              | \-                                                                                        | no              |           `%`            | all                             |
| restOfParagraph      | like `}`, but linewise                                                                    | \-                                                                                        | no              |           `r`            | all                             |
| entireBuffer         | entire buffer as one text object                                                          | \-                                                                                        | \-              |           `gG`           | all                             |
| nearEoL              | from cursor position to end of line, minus one character                                  | \-                                                                                        | no              |           `n`            | all                             |
| lineCharacterwise    | current line, but characterwise                                                           | outer includes indentation and trailing spaces                                            | no              |        `i_`, `a_`        | all                             |
| column               | column down until indent or shorter line. Accepts `{count}` for multiple columns.         | \-                                                                                        | no              |           `\|`           | all                             |
| value                | value of key-value pair, or right side of a variable assignment (inside one line)         | outer includes trailing commas or semicolons                                              | yes             |        `iv`, `av`        | all                             |
| key                  | key of key-value pair, or left side of a variable assignment                              | outer includes the `=` or `:`                                                             | yes             |        `ik`, `ak`        | all                             |
| url                  | link beginning with "http"                                                                | \-                                                                                        | yes             |           `L`            | all                             |
| number\*             | numbers, similar to `<C-a>`                                                               | inner: only pure digits, outer: number including minus sign and decimal point             | yes             |        `in`, `an`        | all                             |
| diagnostic           | LSP diagnostic (requires built-in LSP)                                                    | \-                                                                                        | yes             |           `!`            | all                             |
| closedFold           | closed fold                                                                               | outer includes one line after the last folded line                                        | yes             |        `iz`, `az`        | all                             |
| chainMember          | field with the full call, like `.encode(param)`                                           | outer includes the leading `.` (or `:`)                                                   | yes             |        `im`, `am`        | all                             |
| visibleInWindow      | all lines visible in the current window                                                   |                                                                                           | no              |           `gw`           | all                             |
| restOfWindow         | from the cursorline to the last line in the window                                        |                                                                                           | no              |           `gW`           | all                             |
| mdlink               | markdown link like `[title](url)`                                                         | inner is only the link title (between the `[]`)                                           | yes             |        `il`, `al`        | markdown, toml                  |
| mdFencedCodeBlock    | markdown fenced code (enclosed by three backticks)                                        | outer includes the enclosing backticks                                                    | yes             |        `iC`, `aC`        | markdown                        |
| cssSelector          | class in CSS like `.my-class`                                                             | outer includes trailing comma and space                                                   | yes             |        `ic`, `ac`        | css, scss                       |
| htmlAttribute        | attribute in html/xml like `href="foobar.com"`                                            | inner is only the value inside the quotes trailing comma and space                        | yes             |        `ix`, `ax`        | html, xml, css, scss, vue       |
| jsRegex\*            | JavaScript regex pattern                                                                  | outer includes the slashes and any flags                                                  | yes             |        `i/`, `a/`        | javascript, typescript          |
| doubleSquareBrackets | text enclosed by `[[]]`                                                                   | outer includes the four square brackets                                                   | yes             |        `iD`, `aD`        | lua, shell, neorg, markdown     |
| shellPipe            | command stdout is piped to                                                                | outer includes the front pipe character                                                   | yes             |        `iP`,`aP`         | bash, zsh, fish, sh             |

<!-- vale on -->
<!-- LTeX: enabled=true -->

> __Warning__  
> \* Textobject deprecated due to
> [treesitter-textobject](https://github.com/nvim-treesitter/nvim-treesitter-textobjects)
> introducing a similar textobject that is more capable.

## Installation

```lua
-- packer
use {
	"chrisgrieser/nvim-various-textobjs",
	config = function () 
		require("various-textobjs").setup({ useDefaultKeymaps = true })
	end,
}

-- lazy.nvim
{
	"chrisgrieser/nvim-various-textobjs",
	opts = { useDefaultKeymaps = true },
},
```

## Configuration
The `.setup()` call is optional if you are fine with the defaults below. (Note that the default is to __not__ set any keymaps.)

```lua
-- default config
require("various-textobjs").setup {
	-- lines to seek forwards for "small" textobjs (mostly characterwise textobjs)
	-- set to 0 to only look in the current line
	lookForwardSmall = 5, 

	-- lines to seek forwards for "big" textobjs (linewise textobjs & url textobj)
	lookForwardBig = 15,

	-- use suggested keymaps (see README)
	useDefaultKeymaps = false, 
}
```

---

If you want to set your own keybindings, you can do so by calling the respective functions:
- The function names correspond to the textobject names from the [overview table](#list-of-text-objects).
- The text objects that differentiate between outer and inner require a Boolean parameter, `true` always meaning "inner," and `false` meaning "outer."
- The keymaps *need* to be called as Ex-command, otherwise they will not be
  dot-repeatable. `function () require("various-textobjs").diagnostic() end` as third argument for the keymap works in general, but the text objects<!-- vale Google.Will = NO --> will not be dot-repeatable then.
<!-- vale Google.Will = YES -->

```lua
-- example: `?` for diagnostic textobj
vim.keymap.set({ "o", "x" }, "?", '<cmd>lua require("various-textobjs").diagnostic()<CR>')

-- example: `an` for outer subword, `in` for inner subword
vim.keymap.set({ "o", "x" }, "aS", '<cmd>lua require("various-textobjs").subword(false)<CR>')
vim.keymap.set({ "o", "x" }, "iS", '<cmd>lua require("various-textobjs").subword(true)<CR>')

-- exception: indentation textobj requires two parameters, the first for
-- exclusion of the starting border, the second for the exclusion of ending
-- border
vim.keymap.set({ "o", "x" }, "ii", '<cmd>lua require("various-textobjs").indentation(true, true)<CR>')
vim.keymap.set({ "o", "x" }, "ai", '<cmd>lua require("various-textobjs").indentation(false, true)<CR>')
```

For your convenience, here the code to create mappings for all text objects. You can copypaste this list and enter your own bindings.
<details>
<summary>Mappings for all text objects</summary>

```lua
local keymap = vim.keymap.set

keymap({ "o", "x" }, "ii", "<cmd>lua require('various-textobjs').indentation(true, true)<CR>")
keymap({ "o", "x" }, "ai", "<cmd>lua require('various-textobjs').indentation(false, true)<CR>")
keymap({ "o", "x" }, "iI", "<cmd>lua require('various-textobjs').indentation(true, true)<CR>")
keymap({ "o", "x" }, "aI", "<cmd>lua require('various-textobjs').indentation(false, false)<CR>")

keymap({ "o", "x" }, "YOUR_MAPPING", "<cmd>lua require('various-textobjs').subword(true)<CR>")
keymap({ "o", "x" }, "YOUR_MAPPING", "<cmd>lua require('various-textobjs').subword(false)<CR>")

keymap({ "o", "x" }, "YOUR_MAPPING", "<cmd>lua require('various-textobjs').toNextClosingBracket()<CR>")

keymap({ "o", "x" }, "YOUR_MAPPING", "<cmd>lua require('various-textobjs').restOfParagraph()<CR>")

keymap({ "o", "x" }, "YOUR_MAPPING", "<cmd>lua require('various-textobjs').entireBuffer()<CR>")

keymap({ "o", "x" }, "YOUR_MAPPING", "<cmd>lua require('various-textobjs').nearEoL()<CR>")

keymap({ "o", "x" }, "YOUR_MAPPING", "<cmd>lua require('various-textobjs').lineCharacterwise(true)<CR>")
keymap({ "o", "x" }, "YOUR_MAPPING", "<cmd>lua require('various-textobjs').lineCharacterwise(false)<CR>")

keymap({ "o", "x" }, "YOUR_MAPPING", "<cmd>lua require('various-textobjs').column()<CR>")

keymap({ "o", "x" }, "YOUR_MAPPING", "<cmd>lua require('various-textobjs').value(true)<CR>")
keymap({ "o", "x" }, "YOUR_MAPPING", "<cmd>lua require('various-textobjs').value(false)<CR>")

keymap({ "o", "x" }, "YOUR_MAPPING", "<cmd>lua require('various-textobjs').key(true)<CR>")
keymap({ "o", "x" }, "YOUR_MAPPING", "<cmd>lua require('various-textobjs').key(false)<CR>")

keymap({ "o", "x" }, "YOUR_MAPPING", "<cmd>lua require('various-textobjs').url()<CR>")

keymap({ "o", "x" }, "YOUR_MAPPING", "<cmd>lua require('various-textobjs').diagnostic()<CR>")

keymap({ "o", "x" }, "YOUR_MAPPING", "<cmd>lua require('various-textobjs').closedFold(true)<CR>")
keymap({ "o", "x" }, "YOUR_MAPPING", "<cmd>lua require('various-textobjs').closedFold(false)<CR>")

keymap({ "o", "x" }, "YOUR_MAPPING", "<cmd>lua require('various-textobjs').chainMember(true)<CR>")
keymap({ "o", "x" }, "YOUR_MAPPING", "<cmd>lua require('various-textobjs').chainMember(false)<CR>")

keymap({ "o", "x" }, "YOUR_MAPPING", "<cmd>lua require('various-textobjs').visibleInWindow()<CR>")
keymap({ "o", "x" }, "YOUR_MAPPING", "<cmd>lua require('various-textobjs').restOfWindow()<CR>")

--------------------------------------------------------------------------------------
-- put these into the ftplugins or autocms for the filetypes you want to use them with

keymap(
	{ "o", "x" },
	"YOUR_MAPPING",
	"<cmd>lua require('various-textobjs').mdlink(true)<CR>",
	{ buffer = true }
)
keymap(
	{ "o", "x" },
	"YOUR_MAPPING",
	"<cmd>lua require('various-textobjs').mdlink(false)<CR>",
	{ buffer = true }
)

keymap(
	{ "o", "x" },
	"YOUR_MAPPING",
	"<cmd>lua require('various-textobjs').mdFencedCodeBlock(true)<CR>",
	{ buffer = true }
)
keymap(
	{ "o", "x" },
	"YOUR_MAPPING",
	"<cmd>lua require('various-textobjs').mdFencedCodeBlock(false)<CR>",
	{ buffer = true }
)

keymap(
	{ "o", "x" },
	"YOUR_MAPPING",
	"<cmd>lua require('various-textobjs').cssSelector(true)<CR>",
	{ buffer = true }
)
keymap(
	{ "o", "x" },
	"YOUR_MAPPING",
	"<cmd>lua require('various-textobjs').cssSelector(false)<CR>",
	{ buffer = true }
)

keymap(
	{ "o", "x" },
	"YOUR_MAPPING",
	"<cmd>lua require('various-textobjs').htmlAttribute(true)<CR>",
	{ buffer = true }
)
keymap(
	{ "o", "x" },
	"YOUR_MAPPING",
	"<cmd>lua require('various-textobjs').htmlAttribute(false)<CR>",
	{ buffer = true }
)

keymap(
	{ "o", "x" },
	"YOUR_MAPPING",
	"<cmd>lua require('various-textobjs').doubleSquareBrackets(true)<CR>",
	{ buffer = true }
)
keymap(
	{ "o", "x" },
	"YOUR_MAPPING",
	"<cmd>lua require('various-textobjs').doubleSquareBrackets(false)<CR>",
	{ buffer = true }
)

keymap(
	{ "o", "x" },
	"YOUR_MAPPING",
	"<cmd>lua require('various-textobjs').shellPipe(true)<CR>",
	{ buffer = true }
)
keymap(
	{ "o", "x" },
	"YOUR_MAPPING",
	"<cmd>lua require('various-textobjs').shellPipe(false)<CR>",
	{ buffer = true }
)
```

</details>

## Advanced Usage

### Smart Alternative to `gx`
Using the URL textobject, you can also write a small snippet to replace netrw's `gx`. The code below retrieves the next URL (within the amount of lines configured in the `setup` call), and opens it in your browser. While this is already an improvement to vim's built-in `gx`, which requires the cursor to be standing on a URL to work, you can even go one step further. If no URL has been found within the next few lines, the `:UrlView` command from [urlview.nvim](https://github.com/axieax/urlview.nvim) is triggered. This searches the entire buffer for URLs to choose from.

```lua
vim.keymap.set("n", "gx", function()
	 -- select URL
	require("various-textobjs").url()

	-- plugin only switches to visual mode when textobj found
	local foundURL = vim.fn.mode():find("v")

	-- if not found, search whole buffer via urlview.nvim instead
	if not foundURL then
		vim.cmd.UrlView("buffer")
		return
	end

	-- retrieve URL with the z-register as intermediary
	vim.cmd.normal { '"zy', bang = true }
	local url = vim.fn.getreg("z")

	-- open with the OS-specific shell command
	local opener
	if vim.fn.has("macunix") == 1 then
		opener = "open"
	elseif vim.fn.has("linux") == 1 then
		opener = "xdg-open"
	elseif vim.fn.has("win64") == 1 or vim.fn.has("win32") == 1 then
		opener = "start"
	end
	local openCommand = string.format("%s '%s' >/dev/null 2>&1", opener, url)
	os.execute(openCommand)
end, { desc = "Smart URL Opener" })
```

### Delete Surrounding Indentation
Using the indentation textobject, you can also create custom indentation-related utilities. A common operation is to remove the line before and after an indentation, like in this case where remove the `foo` condition for running the two print commands:

```lua
-- before (cursor on `print("bar")`)
if foo then
	print("bar")
	print("baz")
end

-- after
print("bar")
print("baz")
```

The code below achieves this by dedenting the inner indentation textobject (essentially running `<ii`), and deleting the two lines surrounding it. As for the mapping, `dsi` should make sense since this command is somewhat similar to the `ds` operator from [vim-surround](https://github.com/tpope/vim-surround) but performed on an indentation textobject. (It is also an intuitive mnemonic: `d`elete `s`urrounding `i`ndentation.)

```lua
vim.keymap.set("n", "dsi", function()
	-- select inner indentation
	require("various-textobjs").indentation(true, true)

	-- plugin only switches to visual mode when textobj found
	local notOnIndentedLine = vim.fn.mode():find("V") == nil
	if notOnIndentedLine then return end

	-- dedent indentation
	vim.cmd.normal { ">" , bang = true }

	-- delete surrounding lines
	local endBorderLn = vim.api.nvim_buf_get_mark(0, ">")[1] + 1
	local startBorderLn = vim.api.nvim_buf_get_mark(0, "<")[1] - 1
	vim.cmd(tostring(endBorderLn) .. " delete") -- delete end first so line index is not shifted
	vim.cmd(tostring(startBorderLn) .. " delete")
end, { desc = "Delete surrounding indentation" })
```

### Other Ideas?
If you have some other useful ideas, feel free to [share them in this repo's discussion page](https://github.com/chrisgrieser/nvim-various-textobjs/discussions).

## Limitations
- This plugin uses pattern matching, so it can be inaccurate in some edge cases. 
- The value-textobj does not work with multi-line values. 

## Other Text Object Plugins
- [treesitter-textobjects](https://github.com/nvim-treesitter/nvim-treesitter-textobjects)
- [treesitter-textsubjects](https://github.com/RRethy/nvim-treesitter-textsubjects)
- [ts-hint-textobject](https://github.com/mfussenegger/nvim-ts-hint-textobject)
- [mini.ai](https://github.com/echasnovski/mini.nvim/blob/main/readmes/mini-ai.md)
- [targets.vim](https://github.com/wellle/targets.vim)

## Credits
__Thanks__  
- To the Valuable Dev for [their blog post on how to get started with creating custom text objects](https://thevaluable.dev/vim-create-text-objects/).
- [To `@vypxl` and `@ii14` for figuring out dot-repeatability.](https://github.com/chrisgrieser/nvim-spider/pull/4)

<!-- vale Google.FirstPerson = NO -->
__About Me__  
In my day job, I am a sociologist studying the social mechanisms underlying the digital economy. For my PhD project, I investigate the governance of the app economy and how software ecosystems manage the tension between innovation and compatibility. If you are interested in this subject, feel free to get in touch.

__Blog__  
I also occasionally blog about vim: [Nano Tips for Vim](https://nanotipsforvim.prose.sh)

__Profiles__  
- [Discord](https://discordapp.com/users/462774483044794368/)
- [Academic Website](https://chris-grieser.de/)
- [GitHub](https://github.com/chrisgrieser/)
- [Twitter](https://twitter.com/pseudo_meta)
- [ResearchGate](https://www.researchgate.net/profile/Christopher-Grieser)
- [LinkedIn](https://www.linkedin.com/in/christopher-grieser-ba693b17a/)

__Buy Me a Coffee__  
<br>
<a href='https://ko-fi.com/Y8Y86SQ91' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://cdn.ko-fi.com/cdn/kofi1.png?v=3' border='0' alt='Buy Me a Coffee at ko-fi.com' /></a>
