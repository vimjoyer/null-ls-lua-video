## code actions

```lua
local null_ls = require("null-ls")

local function get_fun_predicate(node, params)
    if node == nil or node:type() ~= "identifier" then return false end

    local parent = node:parent()
    if parent == nil or parent:type() ~= "function_definition" then return false end

    local node_text = vim.treesitter.get_node_text(node, params["bufnr"])
    if node_text:match("^get") == nil then return false end

    return true
end

null_ls.register({
    method = null_ls.methods.CODE_ACTION,
    filetypes = { "python" },
    generator = {
        fn = function(params)
            local out = {}

            local node = vim.treesitter.get_node()
            if get_fun_predicate(node, params) then
                table.insert(out,
                    {
                        title = 'test',
                        action = function()
                            vim.fn.append(params["row"]-1, "# OMG")
                        end
                    })
            end
            return out
        end,
    },
})
```

## diagnistics

```lua
local null_ls = require("null-ls")

require 'null-ls'.register({
    method = null_ls.methods.DIAGNOSTICS,
    filetypes = { "python" },
    generator = {
        fn = function(params)
            local diagnostics = {}

            for i, line in ipairs(params.content) do
                local match = line:match([[# URGENTFIX(.*)]])
                if match then
                    table.insert(diagnostics, {
                        row = i,
                        message = [[please ]] .. match,
                        severity = vim.diagnostic.severity.WARN,
                    })
                end
            end

            for i, line in ipairs(params.content) do
                local col, end_col = line:find([[waitUntil]])
                if col and end_col then
                    local linee = vim.fn.getline(i)
                    local match = linee:match([[waitUntil%("(.*)"%)]])
                    if tableContains(waitUntils, match) then
                        table.insert(diagnostics, {
                            row = i,
                            col = col,
                            end_col = end_col + 1,
                            source = "video-utils",
                            message = "Duplicate waitUntil",
                            severity = vim.diagnostic.severity.ERROR,
                        })
                    end
                    table.insert(waitUntils, match)
                end
            end
            
            table.insert(diagnostics, {
                row = 9,
                -- col = 1,
                -- end_col = 4,
                -- source = "video-utils",
                message = "This line is wrong!",
                severity = vim.diagnostic.severity.ERROR,
            })

            return diagnostics
        end,
    },
})
```
