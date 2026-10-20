# dotfiles

## Zed（macOS / Omarchy）

`zed/settings.json` 和 `zed/keymap.json` 保存共用配置，使用 Zed 支持的 JSONC 格式。
来源是 Mac 上的现有配置，保留主题、字体、面板、AI 和语言设置。
当前没有自定义任务或片段，因此没有创建空的 `tasks.json` 或 `snippets/`。
只管理这两个文件，不同步账号凭据、聊天记录、缓存或整个 Zed 配置目录。

### 接入配置

在每台机器上，先将本仓库放到固定位置，关闭 Zed，再从仓库根目录执行以下命令。
命令会备份已有配置并创建文件软链接；重复执行时会跳过已经正确链接的文件。
macOS 使用 `~/.config/zed`，Linux 也支持 `XDG_CONFIG_HOME`。

```sh
(
  set -eu
  dotfiles_dir="$(pwd -P)"
  test -f "$dotfiles_dir/zed/settings.json"
  test -f "$dotfiles_dir/zed/keymap.json"
  zed_config_dir="$HOME/.config/zed"
  if [ "$(uname -s)" = "Linux" ]; then
    zed_config_dir="${XDG_CONFIG_HOME:-$HOME/.config}/zed"
  fi
  mkdir -p "$zed_config_dir"
  zed_backup_dir=""

  for zed_file in settings.json keymap.json; do
    zed_source="$dotfiles_dir/zed/$zed_file"
    zed_target="$zed_config_dir/$zed_file"
    if [ -L "$zed_target" ] && [ "$(readlink "$zed_target")" = "$zed_source" ]; then
      continue
    fi
    if [ -e "$zed_target" ] || [ -L "$zed_target" ]; then
      if [ -z "$zed_backup_dir" ]; then
        zed_backup_dir="$(mktemp -d "$zed_config_dir/dotfiles-backup.XXXXXX")"
        printf '原配置备份到：%s\n' "$zed_backup_dir"
      fi
      mv "$zed_target" "$zed_backup_dir/$zed_file"
    fi
    ln -s "$zed_source" "$zed_target"
  done
)
```

重启 Zed 后使用仓库里的配置。备份位于本机配置目录中的 `dotfiles-backup.*`，
不会进入仓库。软链接依赖仓库位置，移动仓库后需重新执行上述命令。

### 跨平台约定

- 快捷键使用 `secondary`：macOS 对应 `Cmd`，Linux 对应 `Ctrl`。
  `secondary-,` / `secondary-.` 切换标签页，`secondary-shift-e` 聚焦文件面板，
  `secondary-shift-g` 聚焦 Git 面板。原先禁用 `Ctrl-Shift-G` 的规则仅在 macOS 生效，
  避免覆盖 Linux 的 Git 面板快捷键。
- 两端自行安装 `Ioskeley Mono` 字体，Git 不会同步系统字体。
- `auto_install_extensions` 声明自动安装当前使用的 `catppuccin` 主题和 `ruby` 扩展。
- Ruby LSP 的启动命令使用 `mise`，不再绑定 Mac 的绝对路径。
  如需启用语言服务器，两端的 Zed 项目环境必须能在 `PATH` 中找到 `mise`，
  且项目的 Bundler 环境需包含 `ruby-lsp`。当前沿用 `enable_language_server: false`。
- 沿用 `disable_ai: false` 和现有 Copilot / Agent 设置；新设备需要自行登录相关服务。
  保留的 `FIG_NEW_SESSION` 环境变量供本机终端集成使用。

### 日常同步

接入软链接后，在 Zed 中修改设置会写入本仓库。检查 `git diff`，
确认没有密钥或令牌后，自行提交并推送；另一台机器先处理本地未提交修改，再运行
`git pull --ff-only`。这仍是手动 Git 同步，不会自动双向推送或解决冲突。
字体、账号登录和其他未声明的扩展需要在各设备单独配置。

参考：[Zed 快捷键](https://zed.dev/docs/key-bindings)、
[扩展自动安装](https://zed.dev/docs/reference/all-settings#auto-install-extensions)、
[Zed 项目环境](https://zed.dev/docs/environment)。
