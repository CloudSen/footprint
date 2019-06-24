# VIM 精选插件

## 编辑

- [Rainbow Parentheses Improved](): 五颜六色括号配对；

  ```
  Plug 'luochen1990/rainbow'
  let g:rainbow_active = 1
  ```

- [vim-surround](<https://github.com/tpope/vim-surround>): 成对符号编辑，使用圆括号、花括号、引号包裹内容;

  ```
  Plug 'tpope/vim-surround'
  ```

- [vim-signature](<https://github.com/kshenoy/vim-signature>): 快速标记跳转;

  ```
  Plug 'kshenoy/vim-signature'
  =====================================================
  m<Space>     Delete all marks from the current buffer
  m-           Delete all marks from the current line
  mx           Toggle mark 'x' and display it in the leftmost column
  dmx          Remove mark 'x' where x is a-zA-Z
  m,           Place the next available mark
  m/           Open location list and display marks from current buffer
  ]`           Jump to next mark
  [`           Jump to prev mark
  ]'           Jump to start of next line containing a mark
  ['           Jump to start of prev line containing a mark
  `` 回到到上次修改的位置
  ```

  