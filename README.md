<img width="4204" height="2664" alt="image" src="https://github.com/user-attachments/assets/f7f33d94-e243-4a87-8bd8-b9e332d53947" />

## Plugin ID

```text
santhosh-tekuri/picker.nvim
https://github.com/santhosh-tekuri/picker.nvim
```

you may want to use my [quickfix plugin](https://github.com/santhosh-tekuri/quickfix.nvim) to make quickfix look similar to the list in above screenshot

## Sample Configuration

there is no `setup` function

```lua
local m = require("picker")
vim.ui.select = m.select
vim.keymap.set('n', '<leader>f', m.pick_file)
vim.keymap.set('n', '<leader>b', m.pick_buffer)
vim.keymap.set('n', '<leader>h', m.pick_help)
vim.keymap.set('n', '<leader>/', m.pick_grep)
vim.keymap.set('n', '<leader>u', m.pick_undo)
vim.keymap.set('n', '<leader>q', m.pick_qfitem)
vim.api.nvim_create_autocmd('LspAttach', {
    group = vim.api.nvim_create_augroup('LspPickers', {}),
    callback = function(ev)
        local function opts(desc)
            return { buffer = ev.buf, desc = desc }
        end
        vim.keymap.set('n', 'gd', m.pick_definition, opts("Goto definition"))
        vim.keymap.set('n', 'gD', m.pick_declaration, opts("Goto declaration"))
        vim.keymap.set('n', 'gy', m.pick_type_definition, opts("Goto type definition"))
        vim.keymap.set('n', 'gi', m.pick_implementation, opts("Goto implementation"))
        vim.keymap.set('n', '<leader>r', m.pick_reference, opts("Goto reference"))
        vim.keymap.set('n', '<leader>s', m.pick_document_symbol, opts("Open symbol picker"))
        vim.keymap.set('n', '<leader>S', m.pick_workspace_symbol, opts("Open workspace symbol picker"))
        vim.keymap.set('n', '<leader>d', m.pick_document_diagnostic, opts("Open diagnostic picker"))
        vim.keymap.set('n', '<leader>D', m.pick_workspace_diagnostic, opts("Open workspace diagnostic picker"))
    end,
});
```

by default `<c-q>` open quickfix list. You may want to change this behavior.  
say you want to open trouble quickfix window

```lua
require("picker").after_qflist = function()
    -- do what you want here
end
```

## `Pick` command

- without any arguments it opens picker of pickers
- `Pick <name>` opens named picker. ex: `Pick file`
- `Pick <cmd>`
  - opens picker for cmd argument
  - ex: `Pick colorscheme` `Pick hi` `Pick MasonInstall`

## Keybindings

```text
<esc>           exit picker
<c-c>           cancel background search

<c-h>           toggle filter
<c-g>           toggle live mode
<c-w>           toggle wrap for preview

<cr>            accept selected item
<c-s>           open selected item in horizontal split
<c-v>           open selected item in vertical split
<c-t>           open selected item in new tab
<c-n>           select next item. if no next item, selects first item
<c-p>           select prev item. if no prev item, selects last item
<c-q>           open quickfix list with all items
<a-q>           open location list with all items

<c-d>           scroll list down
<c-u>           scroll list up
<c-f>           scroll preview forward
<c-b>           scroll preview backward

<c-a>           go to start of input
<c-e>           go to end of input
<c-k>           clear input
```

## Search Syntax

accepts multiplse search terms separated with whitespace  
smartcase always enabled

```text
text            containing 'text'
^text           starting with 'text'
text$           ending with 'text'
=text           item is 'text'; shortcut for ^text$
!term           negate match
```

## Search Modifiers (mods)

mods change the target of search. mod starts with `%`

- `%mod:term` search for `term` within the target defined `mod`  
- `%mod:` to just filter the items containing target mod
  - for example `%e:`

there are three types of mods.

### 1. range mods:
- they target a range/portion of text in visible text
- you can see the matched portion highlighted in the item

```plain
%p      filepath
%h      head of filepath
%t      tail of filepath i.e filename
%e      extension of file
%m      main juicy part
        - for symbol picker: it is symbol name
        - for grep filter: it is the fileline
        - for qf like filters: it is item's text
%k      kind
        - for symbol piker: it is symbol kind
%%      entire visible text
```

### 2. string mods:
- they target a text that is not visible
- since the target text is not visible, there is no highlights associated with it

```text
%k      kind
        - for diagnostic picker: it is severity
```

### 3. boolean mods:
- their search target is a boolean value
- the mod starts with capital letter
- they do not accept any search term
- they are standalone mods i.e they don't accept search term
  - for example: just simply type `%E` to filter errors

```text
%E      is error diagnostic
%W      is warning diagnostic
%H      is hint diagnostic
%I      is info diagnostic
```

some tips:
- `%e:=js` filters are files with extension `js`
- '!%mod' filters the items without mod
   - for example: `!%e` to fitler items without file extension

### stickyness:
- without colon after mod, all following terms target the same mod
- non boolean mods become sticky if there is no search term

for example `%h /abc/ def/$` filters all items are are in directory `def` any where inside directory `abc`

you can reset the target to while line using `%%`  
for example `%h term1 term2 %% term3 term4`, here `term1` and `term2` are targeted for `%h` whereas `term3` targets entire line

`!%mod` is never sticky

`%e: term1 term1` here `term1` and `term2` target entire line, and it also filter out items with extension

`%e` just the modifier fitlers out items with extension in this example

## Highlight Groups

You may want to configure highlight groups explicitly depending on your colorscheme.  
you can see the list [here](https://github.com/santhosh-tekuri/picker.nvim/blob/main/lua/picker.lua#L3)

## Undo Picker

- current seq prefixed with `>`
- current seq is preselected
- preview shows `diff prev(selected) selected`
- `<a-p>` toggle the preview to show `diff buffer selected`

## Grep Picker

- needs `ripgrep`
- live mode. can be toggled with `<c-g>`
- smart-case enabled
- accepts `rg` flags in input
  - flag and its value must in in same word ex: `-g=**/dir/**`
  - flags must be before pattern
  - to search for pattern starting with `-` say `-amount` use `-- -amount`
- by default searches in current working directory. you can
  specify it using `--path=some/dir`
- any error messages from `rg` are displayed to user

following are some handy flags that are worth remembering:

```
-l pattern          # show just filenames
-F pattern          # treat pattern as literal string
-g=*.zig            # search in files with given extension
-g=**/xxx/***       # search in directory named xxx anywhere
-g=!**/xxx/***      # don't search in directory named xxx anywhere
-s pattern          # case-sensitive search
```

## File Picker

- needs 'fd' command
- has default filter to exclude hidden files
- toggle filter with `<c-h>`

## LSP Pickers

- has default filter to show only items from current working directory
- toggle filter with `<c-h>`

## QFItem Picker

- to chose an item from quickfix list
- current item is preselected
