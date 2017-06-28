# JS代码格式化那些事儿
*本文主要讲团队开发中如何保持个人编码习惯，同时保持代码格式的一致性*  

## 存在的问题
团队开发中，每个人都有自己的编码风格以及习惯使用的编辑器，这就导致一个项目中存在各式各样的代码，降低了项目整体的可读性和可维护性  
如果制定一套代码规范，强制每个人都按照规范来编写代码，会降低团队成员的编码体验；毕竟编码风格是很久养成的习惯，而且不同项目组会有不同的代码规范，强制去改变习惯不太好。

## 解决办法： [jsbeautify](http://jsbeautifier.org)
使用一个跨编辑器的代码格式化插件可以解决上述问题  
[jsbeautify](http://jsbeautifier.org)支持主流的Sublime Text和VS code, 并且支持通过.jsbeautifyrc文件自定义格式化配置  
支持的自定义配置请参见[setting.md](https://github.com/HookyQR/VSCodeBeautify/blob/master/Settings.md)  
js配置快速浏览：
```javascript
"js": {
    "allowed_file_extensions": ["js", "json", "jshintrc", "jsbeautifyrc"],
    "break_chained_methods": false, // Break chained method calls across subsequent lines
    "e4x": false, // Pass E4X xml literals through untouched
    "end_with_newline": false, // End output with newline
    "indent_char": " ", // Indentation character
    "indent_level": 0, // Initial indentation level
    "indent_size": 4, // Indentation size
    "indent_with_tabs": false, // Indent with tabs, overrides `indent_size` and `indent_char`
    "jslint_happy": false, // If true, then jslint-stricter mode is enforced
    "keep_array_indentation": false, // Preserve array indentation
    "keep_function_indentation": false, // Preserve function indentation
    "max_preserve_newlines": 0, // Maximum number of line breaks to be preserved in one chunk (0 disables)
    "preserve_newlines": true, // Whether existing line breaks should be preserved
    "space_after_anon_function": false, // Should the space before an anonymous function's parens be added, "function()" vs "function ()"
    "space_before_conditional": true, // Should the space before conditional statement be added, "if(true)" vs "if (true)"
    "space_in_empty_paren": false, // Add padding spaces within empty paren, "f()" vs "f( )"
    "space_in_paren": false, // Add padding spaces within paren, ie. f( a, b )
    "unescape_strings": false, // Should printable characters in strings encoded in \xNN notation be unescaped, "example" vs "\x65\x78\x61\x6d\x70\x6c\x65"
    "wrap_line_length": 0 // Lines should wrap at next opportunity after this number of characters (0 disables)
  }
```