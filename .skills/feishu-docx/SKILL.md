---
name: feishu-docx
description: Export Feishu/Lark cloud documents to Markdown. Supports docx, sheets, bitable, wiki, and batch wiki space export. Use this skill when you need to read, analyze, write, or reference content from Feishu knowledge base.
---

# Feishu Docx Exporter

Export Feishu/Lark cloud documents to Markdown format for AI analysis.

## Setup (One-time)

```bash
pip install feishu-docx
feishu-docx config set --app-id YOUR_APP_ID --app-secret YOUR_APP_SECRET
```

> Token auto-refreshes. No user interaction required.

## Export Documents

```bash
feishu-docx export "<FEISHU_URL>" -o ./output
```

The exported Markdown file will be saved with the document's title as filename.

### Supported Document Types

- **docx**: Feishu cloud documents → Markdown with images
- **sheet**: Spreadsheets → Markdown tables
- **bitable**: Multidimensional tables → Markdown tables
- **wiki**: Knowledge base nodes → Auto-resolved and exported

## Command Reference

| Command | Description |
|---------|-------------|
| `feishu-docx export <URL>` | Export document to Markdown |
| `feishu-docx create <TITLE>` | Create new document |
| `feishu-docx write <URL>` | Append content to document |
| `feishu-docx update <URL>` | Update specific block |
| `feishu-docx export-wiki-space <URL>` | Batch export entire wiki space |
| `feishu-docx export-workspace-schema <ID>` | Export bitable database schema |
| `feishu-docx bitable-create-record <APP_TOKEN> <TABLE_ID> -j <JSON>` | Create bitable record |
| `feishu-docx bitable-update-record <APP_TOKEN> <TABLE_ID> <RECORD_ID> -j <JSON>` | Update bitable record |
| `feishu-docx auth` | OAuth authorization |
| `feishu-docx config set` | Set credentials |
| `feishu-docx config show` | Show current config |
| `feishu-docx config clear` | Clear token cache |
| `feishu-docx tui` | Interactive TUI interface |

## Examples

### Export a wiki page

```bash
feishu-docx export "https://xxx.feishu.cn/wiki/ABC123" -o ./docs
```

### Export a document with custom filename

```bash
feishu-docx export "https://xxx.feishu.cn/docx/XYZ789" -o ./docs -n meeting_notes
```

### Read content directly (recommended for AI Agent)

```bash
# Output content to stdout instead of saving to file
feishu-docx export "https://xxx.feishu.cn/wiki/ABC123" --stdout
# or use short flag
feishu-docx export "https://xxx.feishu.cn/wiki/ABC123" -c
```

### Export with Block IDs (for later updates)

```bash
# Include block IDs as HTML comments in the Markdown output
feishu-docx export "https://xxx.feishu.cn/wiki/ABC123" --with-block-ids
# or use short flag
feishu-docx export "https://xxx.feishu.cn/wiki/ABC123" -b
```

### Batch Export Entire Wiki Space

```bash
# Export all documents in a wiki space (auto-extract space_id from URL)
feishu-docx export-wiki-space "https://xxx.feishu.cn/wiki/ABC123" -o ./wiki_backup

# Specify depth limit
feishu-docx export-wiki-space "https://xxx.feishu.cn/wiki/ABC123" -o ./docs --max-depth 3

# Export with Block IDs for later updates
feishu-docx export-wiki-space "https://xxx.feishu.cn/wiki/ABC123" -o ./docs -b
```

### Export Database Schema

```bash
# Export bitable/workspace database schema as Markdown
feishu-docx export-workspace-schema <workspace_id>

# Specify output file
feishu-docx export-workspace-schema <workspace_id> -o ./schema.md
```

### Edit Bitable Records (CLI)

```bash
# Create one record
feishu-docx bitable-create-record appXXXXXXXX tblYYYYYYYY -j '{"笔记标题":"测试记录","报错信息":"created by cli"}'

# Update one record
feishu-docx bitable-update-record appXXXXXXXX tblYYYYYYYY recZZZZZZZZ -j '{"报错信息":"updated by cli"}'
```

## Write Documents (CLI)

### Create Document

```bash
# Create empty document
feishu-docx create "我的笔记"

# Create with Markdown content
feishu-docx create "会议记录" -c "# 会议纪要\n\n- 议题一\n- 议题二"

# Create from Markdown file
feishu-docx create "周报" -f ./weekly_report.md

# Create in specific folder
feishu-docx create "笔记" --folder fldcnXXXXXX
```

**如何获取 folder token**:
1. 在浏览器中打开目标文件夹
2. 从 URL 中提取 token：`https://xxx.feishu.cn/drive/folder/fldcnXXXXXX`
3. `fldcnXXXXXX` 就是 folder token

### Append Content to Existing Document

```bash
# Append Markdown content
feishu-docx write "https://xxx.feishu.cn/docx/xxx" -c "## 新章节\n\n内容"

# Append from file
feishu-docx write "https://xxx.feishu.cn/docx/xxx" -f ./content.md
```

> Note: In some environments, `feishu-docx write` may report success but add `0` blocks.
> If this happens, use the **direct Docx OpenAPI append flow** below (recommended default fallback).

### Update Specific Block

```bash
# Step 1: Export with Block IDs
feishu-docx export "https://xxx.feishu.cn/docx/xxx" -b -o ./

# Step 2: Find block ID from HTML comments
# <!-- block:blk123abc -->
# # Heading
# <!-- /block -->

# Step 3: Update the specific block
feishu-docx update "https://xxx.feishu.cn/docx/xxx" -b blk123abc -c "新内容"
```

> **Tip for AI Agents**: When you need to update a specific section:
> 1. Export with `-b` to get block IDs
> 2. Find the target block ID from HTML comments
> 3. Use `feishu-docx update` with that block ID

## Preferred Append Path (Verified)

This path was verified successful on **2026-02-25** for appending a new line to an existing Feishu doc.

### 1) Get a usable token (tenant or oauth)

```bash
# App-level token (tenant mode)
feishu-docx auth --auth-mode tenant

# Optional: user-level token (oauth mode)
feishu-docx auth --auth-mode oauth
```

### 2) Append by calling Docx OpenAPI directly

```bash
TOKEN=$(jq -r '.access_token' /Users/goudan/.feishu-docx/token.json)
DOC_ID="RTWyd8geMoCfTQxyZmCcpC7ynTf"   # target document id
ROOT_BLOCK_ID="$DOC_ID"                # append to document root

curl -sS -X POST "https://open.feishu.cn/open-apis/docx/v1/documents/${DOC_ID}/blocks/${ROOT_BLOCK_ID}/children" \
  -H "Authorization: Bearer ${TOKEN}" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "index": -1,
    "children": [
      {
        "block_type": 2,
        "text": {
          "elements": [
            {
              "text_run": {
                "content": "追加一行测试（个人权限API直写）"
              }
            }
          ]
        }
      }
    ]
  }'
```

### 3) Verify

```bash
feishu-docx export "https://xxx.feishu.cn/docx/${DOC_ID}" --stdout | tail -n 40
```

Expected: response `code: 0`, and exported content includes the appended line.

## Tips

- Images auto-download to `{doc_title}/` folder
- Use `--stdout` or `-c` for direct content output (recommended for agents)
- Use `-b` to export with block IDs for later updates
- Token auto-refreshes, no re-auth needed
- For Lark (overseas): add `--lark` flag
- For append operations, if CLI `write` returns `添加了 0 个 Block`, switch to the direct OpenAPI append flow above.
- The direct OpenAPI flow works with either tenant token or oauth token, as long as token scope and document permissions are sufficient.

## Field Notes (2026-02-26)

- `feishu-docx write -c` may write literal `\n` text if shell escaping is incorrect. For multiline content, prefer `-f <markdown_file>` or shell `$'...'` string.
- Do not trust only `✅ 写入成功` output. Always verify by exporting the document immediately.
- Prefer verification with block IDs:
  - `feishu-docx export "<DOC_URL>" --stdout -b`
  - this can reveal hidden formatting issues such as escaped newline literals.
- `feishu-docx update` may fail with `code: 1770001` (`invalid param`) for some block types/content payloads.
- If update fails and task is urgent, fallback to append a plain text record in one line, then revisit formatting later.

### Safe Append Recipe

```bash
cat > /tmp/entry.md <<'MD'
## 新增账号（YYYY-MM-DD）
- 网站： http://example.com
- 用户名： alice
- 密码： secret
MD

feishu-docx write "<DOC_URL>" -f /tmp/entry.md
feishu-docx export "<DOC_URL>" --stdout -b | tail -n 80
```
