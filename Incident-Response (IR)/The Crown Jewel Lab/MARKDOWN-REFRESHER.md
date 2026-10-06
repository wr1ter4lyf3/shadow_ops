# Markdown Refresher for This Write-Up

Markdown is plain text with small formatting markers. GitHub reads those markers and renders them as headings, links, tables, code blocks, and other formatted elements.

## Headings

Use one `#` for the page title and add more `#` characters for subsections:

```markdown
# Page title
## Major section
### Subsection
#### Smaller subsection
```

In the write-up:

```markdown
## Investigation
### 1. Triaging the reverse-shell alert in Splunk
```

Do not choose heading levels just for visual size. Think of them as an outline: an `###` section belongs inside the `##` section above it.

## Emphasis

```markdown
**bold text**
*italic text*
~~strikethrough text~~
```

Rendered:

**bold text**  
*italic text*  
~~strikethrough text~~

## Inline Code

Wrap a short command, field, filename, or indicator in one backtick:

```markdown
I filtered the file with `grep` and searched for port `8080`.
```

This is useful for technical names inside a sentence. It is not the same as quotation marks.

## Fenced Code Blocks

Use three backticks above and below commands that need their own block. Add a language name after the opening backticks for syntax highlighting:

````markdown
```bash
grep 'host="jira"' network_logs | grep 'log_type="http"'
```
````

Other useful labels include `text`, `json`, `html`, and `wireshark`. GitHub may not color every specialized language, but the label still explains what the block contains.

## Bulleted Lists

```markdown
- Splunk
- Wireshark
- Bash
```

Use bullets when order does not matter.

## Numbered Lists

```markdown
1. Find the alert.
2. Extract the indicators.
3. Pivot into the PCAP.
```

Use numbers when the sequence matters. You can technically type `1.` for every item and GitHub will number the list automatically, but explicit numbering is often easier to edit and review.

## Links

```markdown
[Visible link text](https://example.com)
```

For another file in the same repository:

```markdown
[Markdown refresher](MARKDOWN-REFRESHER.md)
```

## Images

Images use link syntax with an exclamation mark at the beginning:

```markdown
![Description of the image](assets/arp-poisoning.png)
```

The text inside `[]` is alternative text. Describe what the image contributes instead of writing `screenshot` or leaving it empty.

The path `assets/arp-poisoning.png` is **relative**. It tells GitHub to start in the folder containing `README.md`, enter the `assets` folder, and load that image.

## Blockquotes

```markdown
> This is a quoted or visually separated note.
```

GitHub also supports callouts:

```markdown
> [!NOTE]
> This is useful background information.

> [!IMPORTANT]
> This affects how the evidence should be interpreted.

> [!WARNING]
> This action could create risk or data loss.
```

## Tables

```markdown
| Tool | Purpose |
| --- | --- |
| Splunk | Alert triage |
| Wireshark | Packet analysis |
```

The second row is required. Its hyphens tell Markdown that the first row contains headers.

For alignment:

```markdown
| Left | Center | Right |
| :--- | :---: | ---: |
| A | B | C |
```

Perfect visual spacing in the raw Markdown is optional; the pipe characters are what define the columns.

## Collapsible Sections

GitHub supports HTML `details` tags inside Markdown. The write-up uses one to hide spoilers:

```html
<details>
<summary><strong>Show spoilers</strong></summary>

Hidden content goes here.

</details>
```

Keep blank lines around the hidden content so GitHub renders Markdown inside the section correctly.

## Horizontal Rules

Three hyphens create a dividing line:

```markdown
---
```

Use these sparingly. Headings usually organize a README more clearly.

## Escaping Markdown Characters

Place a backslash before a formatting character when you need GitHub to display the character literally:

```markdown
\*This displays asterisks instead of italics.\*
```

Inside fenced code blocks, Markdown formatting is not interpreted, so commands usually do not need escaping.

## Line Breaks and Paragraphs

Press Enter twice to start a new paragraph:

```markdown
First paragraph.

Second paragraph.
```

A single newline often remains part of the same paragraph. Two spaces at the end of a line force a line break, but separate paragraphs are normally easier to maintain.

## Recommended GitHub Folder Structure

```text
crown-jewel-writeup/
├── README.md
├── MARKDOWN-REFRESHER.md
└── assets/
    ├── arp-poisoning.png
    ├── dns-exfiltration.png
    ├── jira-user-agent.png
    ├── plaintext-http-post.png
    ├── reverse-shell-tcp-conversation.png
    ├── splunk-destination-port.png
    └── splunk-source-ip.png
```

GitHub automatically displays `README.md` on the repository's main page. Keep the filenames and folder structure intact so the relative image paths continue working.

## Quick Editing Checklist

Before committing a README:

- Preview the rendered Markdown on GitHub.
- Confirm that every image loads.
- Confirm that heading levels follow a logical outline.
- Use inline code for commands, fields, paths, IP addresses, and ports.
- Use fenced code blocks for multi-line commands or filters.
- Add descriptive alt text to images.
- Remove personal information, real credentials, tokens, or organization-specific data.
- State clearly when the environment is an authorized lab.
- Distinguish direct evidence from inference.

