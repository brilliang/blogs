# settings.json

```json
{
    "files.autoSave": "onFocusChange",
    "workbench.colorTheme": "Monokai",
    "workbench.colorCustomizations": {
        "tab.activeBackground": "#3b5998",
        "editor.selectionBackground": "#975952", //Current SELECTED text
        "editor.selectionHighlightBackground": "#746202", //same content as the selection
        "editor.findMatchBackground": "#026322a8", //Current SEARCH MATCH
        "editor.findMatchHighlightBackground": "#b85901a1" //Other SEARCH MATCHES
    },
    "interactive-smartlog.pull": "hg pull",
    "interactive-smartlog.fetch": "jf get",
    "editor.fontSize": 14,
    "interactive-smartlog.right-drawer-width": 662,
    "interactive-smartlog.bottom-drawer-height": 500
}

```

# Preferences -> user snippers -> python.json
{
	"head": {
		"prefix": "shebang",
		"body": [
			"#! /usr/bin/python3\n",
			""
		],
		"description": "file Shebang"
	},
	"main": {
		"prefix": "main",
		"body": [
			"def main():\n\n",
			"if __name__ == '__main__':\n\tmain()\n"
		],
		"description": "add main file"
	}
}

