# llm-nvim-plugin
Neovim LLM Plugin

## Config Steps for LLM
- Install Neovim (v0.8.0+)
- Add below config that will be out of date.
- Add API key env varibles in bash/whatever shell is being used with the respective name in the function call.
- Profit

```lua
vim.g.base46_cache = vim.fn.stdpath "data" .. "/nvchad/base46/"
vim.g.mapleader = " "

-- bootstrap lazy and all plugins
local lazypath = vim.fn.stdpath "data" .. "/lazy/lazy.nvim"

if not vim.uv.fs_stat(lazypath) then
  local repo = "https://github.com/folke/lazy.nvim.git"
  vim.fn.system { "git", "clone", "--filter=blob:none", repo, "--branch=stable", lazypath }
end

vim.opt.rtp:prepend(lazypath)

local lazy_config = require "configs.lazy"

-- load plugins
require("lazy").setup({
  {
    "NvChad/NvChad",
    lazy = false,
    branch = "v2.5",
    import = "nvchad.plugins",
  },

  { import = "plugins" },
}, lazy_config)

-- load theme
dofile(vim.g.base46_cache .. "defaults")
dofile(vim.g.base46_cache .. "statusline")

require "options"
require "nvchad.autocmds"

vim.schedule(function()
  require "mappings"
end)

    {
      'DarkNoon5891/llm-nvim-plugin',
      dependencies = { 'nvim-lua/plenary.nvim' },
      lazy = false,  -- Explicitly load this plugin on startup
      config = function()
        local system_prompt =
        'You should replace the code that you are sent, only following the comments. Do not talk at all. Only output valid code. Do not provide any backticks that surround the code. Never ever output backticks like this ```. Any comment that is asking you for something should be removed after you satisfy them. Other comments should left alone. Do not output backticks'
        local helpful_prompt =
        'You are a helpful assistant. What I have sent are my notes so far. You are very curt, yet helpful.'
        local llm = require 'llm'


        local function handle_open_router_spec_data(data_stream)
          local success, json = pcall(vim.json.decode, data_stream)
          if success then
            if json.choices and json.choices[1] and json.choices[1].text then
              local content = json.choices[1].text
              if content then
                llm.write_string_at_cursor(content)
              end
            end
          else
            print("non json " .. data_stream)
          end
        end

        local function custom_make_openai_spec_curl_args(opts, prompt)
          local url = opts.url
          local api_key = opts.api_key_name and os.getenv(opts.api_key_name)
          local data = {
            prompt = prompt,
            model = opts.model,
            temperature = 0.7,
            stream = true,
          }
          local args = { '-N', '-X', 'POST', '-H', 'Content-Type: application/json', '-d', vim.json.encode(data) }
          if api_key then
            table.insert(args, '-H')
            table.insert(args, 'Authorization: Bearer ' .. api_key)
          end
          table.insert(args, url)
          return args
        end


        local function llama_405b_base()
          llm.invoke_llm_and_stream_into_editor({
            url = 'https://openrouter.ai/api/v1/chat/completions',
            model = 'meta-llama/llama-3.1-405b',
            api_key_name = 'OPEN_ROUTER_API_KEY',
            max_tokens = '128',
            replace = false,
          }, custom_make_openai_spec_curl_args, handle_open_router_spec_data)
        end

        local function groq_replace()
          llm.invoke_llm_and_stream_into_editor({
            url = 'https://api.groq.com/openai/v1/chat/completions',
            model = 'llama-3.1-70b-versatile',
            api_key_name = 'GROQ_API_KEY',
            system_prompt = system_prompt,
            replace = true,
          }, llm.make_openai_spec_curl_args, llm.handle_openai_spec_data)
        end

        local function groq_help()
          llm.invoke_llm_and_stream_into_editor({
            url = 'https://api.groq.com/openai/v1/chat/completions',
            model = 'llama-3.1-70b-versatile',
            api_key_name = 'GROQ_API_KEY',
            system_prompt = helpful_prompt,
            replace = false,
          }, llm.make_openai_spec_curl_args, llm.handle_openai_spec_data)
        end


        local function ollama_replace()
          llm.invoke_llm_and_stream_into_editor({
            url = 'http://localhost:11434/v1/chat/completions',
            model = 'llama3.1',      -- or any other model you have in Ollama
            api_key_name = 'ollama', -- Ollama doesn't require an API key
            system_prompt = system_prompt,
            replace = true,
          }, llm.make_openai_spec_curl_args, llm.handle_openai_spec_data)
        end

        local function ollama_help()
          llm.invoke_llm_and_stream_into_editor({
            url = 'http://localhost:11434/v1/chat/completions',
            model = 'llama3.1',      -- or any other model you have in Ollama
            api_key_name = 'ollama', -- Ollama doesn't require an API key
            system_prompt = helpful_prompt,
            replace = false,
          }, llm.make_openai_spec_curl_args, llm.handle_openai_spec_data)
        end

        local function openai_replace()
          llm.invoke_llm_and_stream_into_editor({
            url = 'https://api.openai.com/v1/chat/completions',
            model = 'gpt-4o',
            api_key_name = 'OPENAI_API_KEY',
            system_prompt = system_prompt,
            replace = true,
          }, llm.make_openai_spec_curl_args, llm.handle_openai_spec_data)
        end

        local function openai_help()
          llm.invoke_llm_and_stream_into_editor({
            url = 'https://api.openai.com/v1/chat/completions',
            model = 'gpt-4o',
            api_key_name = 'OPENAI_API_KEY',
            system_prompt = helpful_prompt,
            replace = false,
          }, llm.make_openai_spec_curl_args, llm.handle_openai_spec_data)
        end

        local function anthropic_help()
          llm.invoke_llm_and_stream_into_editor({
            url = 'https://api.anthropic.com/v1/messages',
            model = 'claude-3-5-sonnet-20240620',
            api_key_name = 'ANTHROPIC_API_KEY',
            system_prompt = helpful_prompt,
            replace = false,
          }, llm.make_anthropic_spec_curl_args, llm.handle_anthropic_spec_data)
        end

        local function anthropic_replace()
          llm.invoke_llm_and_stream_into_editor({
            url = 'https://api.anthropic.com/v1/messages',
            model = 'claude-3-5-sonnet-20240620',
            api_key_name = 'ANTHROPIC_API_KEY',
            system_prompt = system_prompt,
            replace = true,
          }, llm.make_anthropic_spec_curl_args, llm.handle_anthropic_spec_data)
        end

        vim.keymap.set({ 'n', 'v' }, '<leader>k', groq_replace, { desc = 'llm groq' })
        vim.keymap.set({ 'n', 'v' }, '<leader>K', groq_help, { desc = 'llm groq_help' })
        vim.keymap.set({ 'n', 'v' }, '<leader>L', openai_help, { desc = 'llm openai_help' })
        vim.keymap.set({ 'n', 'v' }, '<leader>l', openai_replace, { desc = 'llm openai' })
        vim.keymap.set({ 'n', 'v' }, '<leader>I', anthropic_help, { desc = 'llm anthropic_help' })
        vim.keymap.set({ 'n', 'v' }, '<leader>i', anthropic_replace, { desc = 'llm anthropic' })
        vim.keymap.set({ 'n', 'v' }, '<leader>o', ollama_replace, { desc = 'llm ollama' })
        vim.keymap.set({ 'n', 'v' }, '<leader>o', llama_405b_base, { desc = 'llama base' }) -- this second o mapping will overwire the first one
        vim.keymap.set({ 'n', 'v' }, '<leader>O', ollama_help, { desc = 'llm ollama_help' })
      end,
    }


```
