[TOC]

## Keyboard Shortcuts
```json
[
//参数提示
{ "key": "alt+/",               "command": "editor.action.triggerParameterHints",
                                "when": "editorHasSignatureHelpProvider && editorTextFocus && !editorReadonly" },
//成员提示
{ "key": "alt+.",               "command": "editor.action.triggerSuggest",
                                "when": "editorHasCompletionItemProvider && editorTextFocus && !editorReadonly" },
//删除行
{ "key": "ctrl+d",              "command": "editor.action.deleteLines",
                                "when": "editorTextFocus && !editorHasSelection && !editorReadonly" },
//选中匹配项
{ "key": "ctrl+d",              "command": "editor.action.addSelectionToNextFindMatch",
                                "when": "editorTextFocus && editorHasSelection && !editorReadonly" }
]
```