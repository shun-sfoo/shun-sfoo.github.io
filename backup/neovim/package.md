# package

编写一个插件管理器，将会用到

1. 在 neovim 中使用异步库，调用外部程序，并将错误信息输出到日志中
2. 在 neovim 中使用协程，将下载的过程中的信息实时输出

参考资料

1 [use libuv in neovim](https://teukka.tech/posts/2020-01-07-vimloop/)

2 [paq.nvim](https://github.com/savq/paq-nvim)

3 [luv](https://github.com/luvit/luv/blob/master/docs.md#uvspawnfile-options-onexit)

## let start

1. 定义 clone 操作,使用到了 libuv 异步库

```lua
local uv = vim.loop

local function clone(pkg, cb)
  local log = uv.fs_open("/tmp/err.log", "a+", 0x1A4)
  local stderr = uv.new_pipe(false) -- 类似 pipe 写入错误信息
  local process = "git"
  local args = {"clone", "https://github.com" .. pkg .. ".git", "/tmp/" .. pkg}
  local handle, _ =
    uv.spawn(
    process,
    {args = args, stdio = {nil, nil, stderr}},
    vim.schedule_wrap(
      function(code)
        uv.fs_close(log) -- 关闭资源操作
        stderr:close()
        handle:close()
        cb(code == 0) -- 完成后的回调操作
      end
    )
  )
end

```

2. 使用协程实时输出信息

首先定义 report

```lua
local function report(op, name, res, n, total)
  local count = n and " [%d/%d]":format(n, total) or ""
  vim.notify("%s %s %s":format(count, messages[op][res], name), res == "err" and vim.log.levels.ERROR)
end
```

然后定义一个 counter

```lua
local function new_counter()
  return coroutine.wrap(
    function(op, total)
      local c = {ok = 0, err = 0, nop = 0}
      while c.ok + c.err + c.nop < total do
        local name, result, over_op = coroutine.yield(true)
        c[result] = c[result] + 1
        if result ~= "nop" then
          report(over_op or op, name, res, c.ok + c.nop, total)
        end
      end

      local summary = " %s complete. %d ok; %d errors;" .. c.nop > 0 and " %d no-ops" or ""
      vim.notify(string:format(summary, op, c.ok, c.err, c.nop))
    end
  )
end
```

1. 执行某项行为时，首先初始化一个 counter 将操作名称和插件数量传入(op, total)
2. 由于 yield 的返回值是 resume 的参数(见:lua 程序设计 p273), 后续调用 counter(name, result, over_op)
   会捕获到这三个参数，然后生成报道.
