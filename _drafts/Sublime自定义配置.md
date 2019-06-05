---
layout: post
---

[TOC]

## Keyboard Shortcuts

```json
[
	//大写选中项
	{ "keys": ["ctrl+shift+x"], "command": "upper_case" },
	//小写选中项
	{ "keys": ["ctrl+shift+y"], "command": "lower_case" },
	//删除行
	{ "keys": ["ctrl+d"], "command": "run_macro_file", "args": {"file": "res://Packages/Default/Delete Line.sublime-macro"}, "context":
		[
			{"key": "selection_empty", "operator": "equal", "operand": true}
		]
	}
]

```
