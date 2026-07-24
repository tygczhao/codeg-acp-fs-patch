# codeg-acp-fs-patch

在 **自有 GitHub 仓库** 里按需编译带补丁的 [xintaofei/codeg](https://github.com/xintaofei/codeg)：

- **补丁**：ACP `initialize` **不声明** client FS（`fs/read_text_file`）
- **效果**：Grok 本地 `read_file`，可读 PNG 等图片（`ImageContent`）
- **产物**：仅 `codeg-server` + `codeg-mcp`（linux x64）

> 无需对上游 `xintaofei/codeg` 有写权限；workflow 只读下载上游 release 源码后打补丁编译。

## 触发构建

### Web UI

1. 打开本仓库 **Actions** → **build-codeg-no-fs**
2. **Run workflow**
3. `version` 可填上游 tag（如 `v0.21.6`），留空则用 latest
4. 跑完下载 Artifact：
   - `codeg-server-linux-x64-no-fs-<tag>`（裸二进制）
   - `codeg-server-linux-x64-no-fs-<tag>-zip`（zip 包）

### CLI

```bash
gh workflow run build-codeg-no-fs.yml -R tygczhao/codeg-acp-fs-patch

# 指定上游版本
gh workflow run build-codeg-no-fs.yml -R tygczhao/codeg-acp-fs-patch -f version=v0.21.6

# 查看最近一次运行
gh run list -R tygczhao/codeg-acp-fs-patch --workflow=build-codeg-no-fs.yml -L 3

# 下载产物（把 RUN_ID 换成实际 id）
gh run download -R tygczhao/codeg-acp-fs-patch <RUN_ID>
```

## 安装到本机

```bash
# 若下载的是 zip，先解压；目录里应有 codeg-server / codeg-mcp
sudo cp codeg-server codeg-mcp /usr/local/bin/
sudo chmod 755 /usr/local/bin/codeg-server /usr/local/bin/codeg-mcp
systemctl --user restart codeg-server.service

# 必须【新开】Grok 会话再测 read_file 图片
```

## 补丁维护

- 文件：`patches/0001-disable-acp-fs-client.patch`
- 上游改了 `connection.rs` 导致 apply 失败时，在本地基于新版本重新 `git diff` 更新补丁并 push，再跑 workflow。

## 相关

本机一键（不经过 GitHub）：`officeSkill/scripts/install-codeg-patched.sh`
