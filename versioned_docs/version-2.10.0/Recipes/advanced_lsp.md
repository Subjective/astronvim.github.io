---
id: advanced_lsp
title: Advanced LSP Setup
---

## Customizing auto-format on save

AstroNvim has made formatting on save part of the default functionality out of the box. If you don't want your code to get auto formatted on save, you can disable it in your `lsp.formatting` table. Here is a minimal `user/init.lua` file that disables the formatting on save:

```lua
return {
  lsp = {
    formatting = {
      format_on_save = false, -- enable or disable automatic formatting on save
    },
  },
}
```

We have also added an extension to just `true` or `false` for this option to give more the user the ability to disable the auto formatting for specific filetypes. For example:

```lua
return {
  lsp = {
    formatting = {
      format_on_save = {
        enabled = true, -- enable format on save
        ignore_filetypes = { -- disable format on save for specified filetypes
          "markdown",
          "python",
        }
      },
    },
  },
}
```

If you would rather use a whitelist of filetypes for formatting on save rather than a blacklist type model, you can do that as well with the `allow_filetypes` table. If you have `allow_filetypes` it will take presidence over `ignore_filetypes`. So please only use one of these options at a time. Here is an example:

```lua
return {
  lsp = {
    formatting = {
      format_on_save = {
        enabled = true, -- enable format on save
        allow_filetypes = { -- only allow formatting on save for these filetypes
          "lua",
          "python",
        }
      },
    },
  },
}
```

With the formatting on save enabled, we have also provided the mapping `<leader>uf` to toggle the auto formatting temporarily along with the variable `vim.g.autoformat_enabled` which allows you to control whether the auto formatter runs on startup initially (_Note:_ Format on save must be enabled in the `lsp.formatting` table for this option and keybinding to do anything).

## Controlling Formatting

Since Neovim v0.8 there have been improvements to how language servers are used for formatting files. Previously Neovim could only use a single language server to format files at a time and would prompt on each format if multiple were available. This led to users disabling formatting capabilities for different language servers and losing that functionality all together for convenience. Now you are able to format with many formatters at the same time and filter them with a function. To empower this, AstroNvim has a configuration option for controlling what formatters are used. This can be done either with a filter function or a list of disabled clients. Here are a couple example `user/init.lua` contents that demonstrate each possibility.

### Disabling formatting for a filter function

using the `filter` option you can supply filter function to be run on each client that has formatting capabilities and if it returns `true` then it will be used for formatting and if it returns `false` then it will not be used. This applies to whenever you format your code either on save, with `<leader>lf`, or with `:Format`.

```lua
return {
  lsp = {
    formatting = {
      filter = function(client)
        -- disable formatting for sumneko_lua
        if client.name == "sumneko_lua" then
          return false
        end

        -- only enable null-ls for javascript files
        if vim.bo.filetype == "javascript" then
          return client.name == "null-ls"
        end

        -- enable all other clients
        return true
      end
    },
  },
}
```

### Disabling formatting for a list of language servers

using the `disabled` option you can supply an array like list of language server client names and those clients will be disabled with you format your code either on save, with `<leader>lf`, or with `:Format`.

```lua
return {
  lsp = {
    formatting = {
      disabled = {
        "sumneko_lua",
        "rust_analyzer",
      },
    },
  },
}
```

### Using both filter function and disabled list

When using the options together, a client listed in the `disabled` list will always be disabled and then all other clients will then be passed into the `filter` function. For example, we can simplify our previous `filter` function by just disabling the `sumneko_lua` client in the `disabled` table:

```lua
return {
  lsp = {
    formatting = {
      disabled = { "sumneko_lua" },
      filter = function(client)
        -- only enable null-ls for javascript files
        if vim.bo.filetype == "javascript" then
          return client.name == "null-ls"
        end

        -- enable all other clients
        return true
      end,
    },
  },
}
```

### Configure other formatting options

We have also provided an option in the `lsp.formatting` table for easily enabling and disabling automatic formatting on save with `lsp.formatting.format_on_save`.

The `lsp.formatting` option also allows you to specify other parameters for the `vim.lsp.buf.format()` call. Any of the other formatting options are allowed to be used here to be used as the default options. This means being able to easily adjust the default `timeout_ms` for formatting in AstroNvim or making asynchronous formatting the default. For example you can do the following to increase the formatting timeout along with adjust the filtering:

```lua
return {
  lsp = {
    formatting = {
      format_on_save = true, -- enable or disable automatic formatting on save
      timeout_ms = 3200, -- adjust the timeout_ms variable for formatting
      disabled = { "sumneko_lua" },
      filter = function(client)
        -- only enable null-ls for javascript files
        if vim.bo.filetype == "javascript" then
          return client.name == "null-ls"
        end

        -- enable all other clients
        return true
      end,
    },
  },
}
```

## LSP Setup Without Installer

AstroNvim comes with [mason-lspconfig](https://github.com/williamboman/mason-lspconfig.nvim) as an easy interface for setting up and installing language servers, but this might not be adequate for all users. The LSP installer doesn't support all of the language servers that Neovim's LSP config supports and some users may already have the language servers installed on their machine and don't want to reinstall it separately. In these cases we have added an easy interface for setting up these servers. The following is a minimal `user/init.lua` file that simply sets up `pyright` language server for a user with `pyright` already available on their system:

```lua
return {
  lsp = {
    servers = {
      "pyright",
    },
  },
}
```

If the user wants to configure server specific settings and configurations then they can do this with the `lsp.server-settings` table as well. For example if the user wants to use `pyright` that is already available on their system and disable the single file support then can do the following:

```lua
return {
  lsp = {
    servers = {
      "pyright",
    },
    ["server-settings"] = {
      pyright = {
        single_filesupport = false,
      },
    },
  },
}
```

## Custom LSP Definition

`nvim-lspconfig` is great, but it doesn't support all language servers that exist. You may want to set up a custom server where you manually define the `cmd` and the `root_dir`. This can be done completely through `lsp.servers` and `lsp.server-settings` just like setting up servers that are supported by `lspconfig`! For these custom servers, the minimum requirement is setting up a `cmd` in `server-settings`, but to get automatic starting of language servers you also need to set `filetypes` and `root_dir`. Here is a simple example setting up a Prolog LSP with `swipl`:

```lua
return {
  lsp = {
    servers = {
      "prolog_lsp",
    },
    ["server-settings"] = {
      prolog_lsp = {
        cmd = {"swipl",
               "-g", "use_module(library(lsp_server)).",
               "-g", "lsp_server:main",
               "-t", "halt",
               "--", "stdio"};
        filetypes = {"prolog"};
        root_dir = require("lspconfig.util").root_pattern("pack.pl");
      }
    },
  },
}
```

## LSP Specific Plugins

There are some plugins available for doing advanced setup of language servers that require the user to not use the `lspconfig` setup call and instead use their own plugin setup for handling this. AstroNvim provides a nice way to do this while still using `mason-lspconfig` for installing the language servers. You can use the `lsp.skip_setup` table for specifying which language servers to guarantee AstroNvim will not automatically call the `lspconfig` setup for. We also provide a helper function for getting the AstroNvim default server configuration like our built in `capabilities`, `on_attach`, as well as the user defined options in `lsp.server-settings`. Here is a couple examples for some common LSP plugins:

### Typescript ([typescript.nvim](https://github.com/jose-elias-alvarez/typescript.nvim))

```lua
return {
  lsp = {
    skip_setup = { "tsserver" },
  },
  plugins = {
    init = {
      {
        "jose-elias-alvarez/typescript.nvim",
        after = "mason-lspconfig.nvim",
        config = function()
          require("typescript").setup {
            server = astronvim.lsp.server_settings "tsserver",
          }
        end,
      },
    },
    ["mason-lspconfig"] = {
      ensure_installed = { "tsserver" },
    },
  },
}
```

### Deno ([deno-nvim](https://github.com/sigmaSd/deno-nvim))

```lua
return {
  lsp = {
    skip_setup = { "denols" },
  },
  plugins = {
    init = {
      {
        "sigmasd/deno-nvim",
        after = "mason-lspconfig.nvim",
        config = function()
          require("deno-nvim").setup {
            server = astronvim.lsp.server_settings "denols",
          }
        end
      },
    },
    ["mason-lspconfig"] = {
      ensure_installed = { "denols" },
    },
  },
}
```

**tsserver + denols**

Since both `tsserver` and `denols` (and others such as `eslint` and `prettier`) attach to TypeScript/JavaScript files, some extra configuration may be required if both are installed.

To conditionally enable `tsserver`/`denols` based on the presence of `package.json`/`deno.json`, add the following to `lsp.server-settings`:

```lua
return {
  lsp = {
    ["server-settings"] = {
      denols = {
        root_dir = require("lspconfig.util").root_pattern("deno.json", "deno.jsonc"),
      },
      tsserver = {
        root_dir = require("lspconfig.util").root_pattern("package.json"),
      },
      -- For eslint:
      -- eslint = {
      --   root_dir = require("lspconfig.util").root_pattern("package.json", ".eslintrc.json", ".eslintrc.js"),
      -- },
    },
  },
}
```

For `null-ls` packages (such as `prettier`, `prettierd`, or `eslint_d`), set the following to `mason-null-ls.setup_handlers`:

```lua
return {
  ["mason-null-ls"] = {
    setup_handlers = {
      prettier = function()
        require("null-ls").register(require("null-ls").builtins.formatting.prettier.with({
          condition = function(utils)
            return utils.root_has_file("package.json") or utils.root_has_file(".prettierrc") or utils.root_has_file(".prettierrc.json") or utils.root_has_file(".prettierrc.js")
          end
        }))
      end,
      -- For prettierd:
      -- prettierd = function()
      --   require("null-ls").register(require("null-ls").builtins.formatting.prettierd.with({
      --     condition = function(utils)
      --       return utils.root_has_file("package.json") or utils.root_has_file(".prettierrc") or utils.root_has_file(".prettierrc.json") or utils.root_has_file(".prettierrc.js")
      --     end
      --   }))
      -- end,
      -- For eslint_d:
      -- eslint_d = function()
      --   require("null-ls").register(require("null-ls").builtins.diagnostics.eslint_d.with({
      --     condition = function(utils)
      --       return utils.root_has_file("package.json") or utils.root_has_file(".eslintrc.json") or utils.root_has_file(".eslintrc.js")
      --     end
      --   }))
      -- end,
    }
  },
}
```

### C/C++ ([clangd_extensions.nvim](https://github.com/p00f/clangd_extensions.nvim))

```lua
return {
  lsp = {
    skip_setup = { "clangd" },
    ["server-settings"] = {
      clangd = {
        capabilities = {
          offsetEncoding = "utf-8",
        },
      },
    },
  },
  plugins = {
    init = {
      {
        "p00f/clangd_extensions.nvim",
        after = "mason-lspconfig.nvim", -- make sure to load after mason-lspconfig
        config = function()
          require("clangd_extensions").setup {
            server = astronvim.lsp.server_settings "clangd",
          }
        end,
      },
    },
    ["mason-lspconfig"] = {
      ensure_installed = { "clangd" },
    },
  },
}
```

### Dart Flutter ([flutter-tools.nvim](https://github.com/akinsho/flutter-tools.nvim))

```lua
return {
  lsp = {
    skip_setup = { "dartls" }, -- skip lsp setup because flutter-tools will do it itself
    ["server-settings"] = {
      dartls = {
        -- any changes you want to make to the LSP setup, for example
        color = {
          enabled = true,
        },
        settings = {
          showTodos = true,
          completeFunctionCalls = true,
        },
      },
    },
  },
  plugins = {
    init = {
      {
        "akinsho/flutter-tools.nvim",
        requires = "nvim-lua/plenary.nvim",
        after = "mason-lspconfig.nvim", -- make sure to load after mason-lspconfig
        config = function()
          require("flutter-tools").setup {
            lsp = astronvim.lsp.server_settings "dartls", -- get the server settings and built in capabilities/on_attach
          }
        end,
      },
    },
    ["mason-lspconfig"] = {
      ensure_installed = { "dartls" }, -- install dartls
    },
  },
}
```

### Rust ([rust-tools.nvim](https://github.com/simrat39/rust-tools.nvim))

```lua
return {
  lsp = {
    skip_setup = { "rust_analyzer" }, -- skip lsp setup because rust-tools will do it itself
  },
  plugins = {
    init = {
      {
        "simrat39/rust-tools.nvim",
        after = "mason-lspconfig.nvim", -- make sure to load after mason-lspconfig
        config = function()
          require("rust-tools").setup {
            server = astronvim.lsp.server_settings "rust_analyzer", -- get the server settings and built in capabilities/on_attach
          }
        end,
      },
    },
    ["mason-lspconfig"] = {
      ensure_installed = { "rust_analyzer" }, -- install rust_analyzer
    },
  },
}
```

### Java ([nvim-jdtls](https://github.com/mfussenegger/nvim-jdtls))

```lua
return {
  lsp = {
    skip_setup = { "jdtls" },
    ["server-settings"] = {
      -- set jdtls server settings
      jdtls = function()
        -- use this function notation to build some variables
        local root_markers = { ".git", "mvnw", "gradlew", "pom.xml", "build.gradle" }
        local root_dir = require("jdtls.setup").find_root(root_markers)

        -- calculate workspace dir
        local project_name = vim.fn.fnamemodify(vim.fn.getcwd(), ":p:h:t")
        local workspace_dir = vim.fn.stdpath "data" .. "/site/java/workspace-root/" .. project_name
        os.execute("mkdir " .. workspace_dir)

        -- get the mason install path
        local install_path = require("mason-registry").get_package("jdtls"):get_install_path()

        -- get the current OS
        local os
        if vim.fn.has "macunix" then
          os = "mac"
        elseif vim.fn.has "win32" then
          os = "win"
        else
          os = "linux"
        end

        -- return the server config
        return {
          cmd = {
            "java",
            "-Declipse.application=org.eclipse.jdt.ls.core.id1",
            "-Dosgi.bundles.defaultStartLevel=4",
            "-Declipse.product=org.eclipse.jdt.ls.core.product",
            "-Dlog.protocol=true",
            "-Dlog.level=ALL",
            "-javaagent:" .. install_path .. "/lombok.jar",
            "-Xms1g",
            "--add-modules=ALL-SYSTEM",
            "--add-opens",
            "java.base/java.util=ALL-UNNAMED",
            "--add-opens",
            "java.base/java.lang=ALL-UNNAMED",
            "-jar",
            vim.fn.glob(install_path .. "/plugins/org.eclipse.equinox.launcher_*.jar"),
            "-configuration",
            install_path .. "/config_" .. os,
            "-data",
            workspace_dir,
          },
          root_dir = root_dir,
        }
      end,
    },
  },
  plugins = {
    init = {
      ["mfussenegger/nvim-jdtls"] = { module = "jdtls" }, -- load jdtls on module
    },
    ["mason-lspconfig"] = { ensure_installed = { "jdtls" } }, -- install jdtls
  },
  polish = function()
    vim.api.nvim_create_autocmd("Filetype", {
      pattern = "java", -- autocmd to start jdtls
      callback = function()
        local config = astronvim.lsp.server_settings "jdtls"
        if config.root_dir and config.root_dir ~= "" then require("jdtls").start_or_attach(config) end
      end,
    })
  end,
}
```
