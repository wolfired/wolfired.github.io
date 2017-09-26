---
layout: post
---

[TOC]

## Keyboard Shortcuts

```json
[
    //参数提示
    {
        "key": "alt+/",
        "command": "editor.action.triggerParameterHints",
        "when": "!editorReadonly && editorTextFocus && editorHasSignatureHelpProvider"
    },
    //成员提示
    {
        "key": "alt+.",
        "command": "editor.action.triggerSuggest",
        "when": "!editorReadonly && editorTextFocus && editorHasCompletionItemProvider"
    },
    //删除行
    {
        "key": "ctrl+d",
        "command": "editor.action.deleteLines",
        "when": "!editorReadonly && editorTextFocus && !editorHasSelection"
    },
    //选中匹配项
    {
        "key": "ctrl+d",
        "command": "editor.action.addSelectionToNextFindMatch",
        "when": "!editorReadonly && editorTextFocus && editorHasSelection"
    },
    //大写选中项
    {
        "key": "ctrl+shift+x",
        "command": "editor.action.transformToUppercase",
        "when": "!editorReadonly && editorTextFocus && editorHasSelection"
    },
    //小写选中项
    {
        "key": "ctrl+shift+y",
        "command": "editor.action.transformToLowercase",
        "when": "!editorReadonly && editorTextFocus && editorHasSelection"
    },
    //选中
    {
        "key": "ctrl+shift+l",
        "command": "editor.action.selectHighlights",
        "when": "!editorReadonly && editorTextFocus && !editorHasSelection"
    },
    //分割选中行
    {
        "key": "ctrl+shift+l",
        "command": "editor.action.insertCursorAtEndOfEachLineSelected",
        "when": "!editorReadonly && editorTextFocus && editorHasSelection"
    }
]
```