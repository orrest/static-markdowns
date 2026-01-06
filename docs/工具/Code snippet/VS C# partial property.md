---
tags:
    - Tools
icon: fontawesome/solid/file-code
---

用于 `CommunityToolkit.Mvvm` 的 partial property Code Snippet。

## Code Snippet

文件路径：`%USERPROFILE%\Documents\Visual Studio 2022\Code Snippets\Visual C#\My Code Snippets`

```XML
<?xml version="1.0" encoding="utf-8"?>
<CodeSnippets xmlns="http://schemas.microsoft.com/VisualStudio/2005/CodeSnippet">
	<CodeSnippet Format="1.0.0">
		<Header>
			<Title>propp</Title>
			<Shortcut>propp</Shortcut>
			<Description>Partial public property</Description>
			<Author>Orrest</Author>
		</Header>
		<Snippet>
			<Declarations>
				<Literal>
					<ID>type</ID>
					<ToolTip>type</ToolTip>
					<Default>Type</Default>
				</Literal>
				<Literal>
					<ID>prop</ID>
					<ToolTip>Property name</ToolTip>
					<Default>MyProperty</Default>
				</Literal>
			</Declarations>
			<Code Language="csharp">
				<![CDATA[
[ObservableProperty]
public partial $type$ $prop$ { get; set; }
        ]]>
			</Code>
		</Snippet>
	</CodeSnippet>
</CodeSnippets>

```

## Ref

[Walkthrough: Create a code snippet in Visual Studio](https://learn.microsoft.com/en-us/visualstudio/ide/walkthrough-creating-a-code-snippet?view=visualstudio)

