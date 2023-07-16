---
layout: post
title:  Ubuntu keymap
date:   2023-07-13
Author: CBD
tags: [Ubuntu]
---

## Update

Swap Ctrl with Alt on layout level.

![](/images/layout-options.png)

-----

## 基本需求

从 macOS 迁移到 Ubuntu 之后，尽量保持快捷键键位一致。

在使用相同的键盘时，IDE 的快捷键键位一致。

## 背景分析

需求中讲的都是 “键位一致”，是因为 macOS 跟 Linux、Windows 在 Command 键的使用上有很大不同。

- mac 以 Command 为中心，用左大拇指为中心去操控；
- windows、linux 以 ctrl 为中心，用左小拇指为中心去操控。

Ubuntu 的 Command 键叫 Super 键，布局如下：

![](/images/ubuntu-us.png)

键盘的实体按键如下：

![](/images/keyboard.png)

很自然的，就考虑用 alt 来代替 Command 键。

（ 最开始的想法是先把 Super 和 alt 的键帽互换，然后功能互换，然后用 Super 代替 Command。实际操作下来效果很不好，Super 有很多全局的冲突。所以退而求其次的只要求在 IDE 中的键位保持一致。）

## 实操

这里以 JetBrains 的 IDE 为例。

### 导出快捷键配置文件

IDE 中自带了配置文件备份的功能，但这里导出的是增量的修改，我们需要全量的 mapping。

- 安装插件 `macOS Keymap`
- Settings - Keymap - select `macOS` - Duplicate - Save
- 安装插件 [Keymap XML Exporter)](https://plugins.jetbrains.com/plugin/18927-keymap-xml-exporter)
- Help - Export keymap to XML

### XML 配置文件

位于目录 `~/.config/JetBrains/RubyMine2023.1/keymaps`， 最终处理好的文件要覆盖掉这里的 xml 文件。

修改前的样子，有 parent 元素，action 内容是增量的：

```xml
<keymap version="1" name="GNOME copy" parent="Default for GNOME">
  <action id="Terminal.CloseTab">
    <keyboard-shortcut first-keystroke="ctrl f4" />
    <keyboard-shortcut first-keystroke="alt w" />
  </action>
  <action id="ExpandToLevel5">
    <keyboard-shortcut first-keystroke="alt multiply" second-keystroke="5" />
    <keyboard-shortcut first-keystroke="alt multiply" second-keystroke="numpad5" />
  </action>
</keymap>
```

对于全量的 xml， 如果没有对应的 action，则该功能没有快捷键。

### 替换快捷键

替换规则：

- 把 meta 替换为 alt；
- 如果替换为 alt 后存在冲突，则丢弃原快捷键；
- 如果快捷键本身就是 alt 和 meta 的组合，删除该组合；

XML 的解析比较繁琐，先做一下预处理，把每个 action formate 到单独一行，以行为单位进行处理文本文件。

```ruby
require 'byebug'

# delete conflict alt keys with meta
# Ubuntu 22
# ~/.config/JetBrains/RubyMine2023.1/keymaps

result = ''
full = File.read('macOS.xml')

File.readlines('macOS.xml').each do |line|
  keys = line.scan(/keystroke=\"([^"]*)\"/).flatten

  skip = false
  keys.filter { |key| key.include?('alt') }.each do |alt_key|
    # has same meta keymap
    if alt_key.include?('meta') || full.include?(alt_key.gsub('alt', 'meta'))
      skip = true
      puts line
      break
    end
  end

  next if skip

  # replace meta to alt
  result << line.gsub('meta', 'alt')
end

File.write('new.xml', result)

puts 'end'

```

被去除的快捷键中有个别还是很常用的，就手动加回来，处理完的结果如下，

写回到配置文件中 `~/.config/JetBrains/RubyMine2023.1/keymaps/macOS-ubuntu.xml`:

```xml
<keymap version="1" name="macOS-ubuntu">
<action id="$Copy"><keyboard-shortcut first-keystroke="alt c"/><keyboard-shortcut first-keystroke="alt insert"/></action>
<action id="$Cut"><keyboard-shortcut first-keystroke="alt x"/><keyboard-shortcut first-keystroke="shift delete"/></action>
<action id="$Delete"><keyboard-shortcut first-keystroke="delete"/><keyboard-shortcut first-keystroke="back_space"/><keyboard-shortcut first-keystroke="alt back_space"/></action>
<action id="$Paste"><keyboard-shortcut first-keystroke="alt v"/><keyboard-shortcut first-keystroke="shift insert"/></action>
<action id="$SelectAll"><keyboard-shortcut first-keystroke="alt a"/></action>
<action id="$Undo"><keyboard-shortcut first-keystroke="alt z"/></action>
<action id="ActivateBookmarksToolWindow"><keyboard-shortcut first-keystroke="alt 2"/></action>
<action id="ActivateCommitToolWindow"><keyboard-shortcut first-keystroke="alt 0"/></action>
<action id="ActivateDebugToolWindow"><keyboard-shortcut first-keystroke="alt 5"/></action>
<action id="ActivateFindToolWindow"><keyboard-shortcut first-keystroke="alt 3"/></action>
<action id="ActivateProblemsViewToolWindow"><keyboard-shortcut first-keystroke="alt 6"/></action>
<action id="ActivateProjectToolWindow"><keyboard-shortcut first-keystroke="alt 1"/></action>
<action id="ActivateRunToolWindow"><keyboard-shortcut first-keystroke="alt 4"/></action>
<action id="ActivateServicesToolWindow"><keyboard-shortcut first-keystroke="alt 8"/></action>
<action id="ActivateStructureToolWindow"><keyboard-shortcut first-keystroke="alt 7"/></action>
<action id="ActivateVersionControlToolWindow"><keyboard-shortcut first-keystroke="alt 9"/></action>
<action id="Arrangement.Rule.Edit"><keyboard-shortcut first-keystroke="f2"/></action>
<action id="AutoIndentLines"><keyboard-shortcut first-keystroke="ctrl alt i"/></action>
<action id="BraceOrQuoteOut"><keyboard-shortcut first-keystroke="tab"/></action>
<action id="CWMHostShowPopupAction"><keyboard-shortcut first-keystroke="shift ctrl y"/></action>
<action id="CallHierarchy"><keyboard-shortcut first-keystroke="ctrl alt h"/></action>
<action id="ChangeSignature"><keyboard-shortcut first-keystroke="alt f6"/></action>
<action id="ChangeTypeSignature"><keyboard-shortcut first-keystroke="shift alt f6"/></action>
<action id="ChangesView.GroupBy.Directory"><keyboard-shortcut first-keystroke="ctrl p"/></action>
<action id="ChangesView.GroupBy.Module"><keyboard-shortcut first-keystroke="ctrl m"/></action>
<action id="ChangesView.Move"><keyboard-shortcut first-keystroke="shift alt m"/></action>
<action id="ChangesView.Rename"><keyboard-shortcut first-keystroke="f2"/><keyboard-shortcut first-keystroke="shift f6"/></action>
<action id="ChangesView.SetDefault"><keyboard-shortcut first-keystroke="ctrl space"/></action>
<action id="ChangesView.ShelveSilently"><keyboard-shortcut first-keystroke="shift alt h"/></action>
<action id="CheckinProject"><keyboard-shortcut first-keystroke="alt k"/></action>
<action id="ChooseDebugConfiguration"><keyboard-shortcut first-keystroke="ctrl alt d"/></action>
<action id="CloseActiveTab"><keyboard-shortcut first-keystroke="shift ctrl f4"/></action>
<action id="CloseContent"><keyboard-shortcut first-keystroke="alt w"/></action>
<action id="CloseGotItTooltip"><keyboard-shortcut first-keystroke="escape"/></action>
<action id="CodeCompletion"><keyboard-shortcut first-keystroke="ctrl space"/></action>
<action id="CollapseAll"><keyboard-shortcut first-keystroke="alt subtract"/><keyboard-shortcut first-keystroke="alt minus"/></action>
<action id="CollapseAllRegions"><keyboard-shortcut first-keystroke="shift alt subtract"/><keyboard-shortcut first-keystroke="shift alt minus"/></action>
<action id="CollapseBlock"><keyboard-shortcut first-keystroke="shift alt period"/></action>
<action id="CollapseExpandableComponent"><keyboard-shortcut first-keystroke="shift enter"/><keyboard-shortcut first-keystroke="alt subtract"/><keyboard-shortcut first-keystroke="alt minus"/></action>
<action id="CollapseRegion"><keyboard-shortcut first-keystroke="alt subtract"/><keyboard-shortcut first-keystroke="alt minus"/></action>
<action id="CollapseSelection"><keyboard-shortcut first-keystroke="alt period"/></action>
<action id="CollapseTreeNode"><keyboard-shortcut first-keystroke="subtract"/></action>
<action id="CollapsiblePanel-toggle"><keyboard-shortcut first-keystroke="space"/></action>
<action id="CommentByLineComment"><keyboard-shortcut first-keystroke="alt slash"/><keyboard-shortcut first-keystroke="alt divide"/></action>
<action id="CompareTwoFiles"><keyboard-shortcut first-keystroke="alt d"/></action>
<action id="Compile"><keyboard-shortcut first-keystroke="shift alt f9"/></action>
<action id="CompileDirty"><keyboard-shortcut first-keystroke="alt f9"/></action>
<action id="Console.Execute"><keyboard-shortcut first-keystroke="enter"/></action>
<action id="Console.Execute.Multiline"><keyboard-shortcut first-keystroke="alt enter"/></action>
<action id="Console.Open"><keyboard-shortcut first-keystroke="shift alt f10"/></action>
<action id="Console.TableResult.ColumnVisibility"><keyboard-shortcut first-keystroke="space"/></action>
<action id="Console.TableResult.CompareCells"><keyboard-shortcut first-keystroke="shift alt d"/></action>
<action id="Console.TableResult.SelectRow"><keyboard-shortcut first-keystroke="shift space"/></action>
<action id="ContextHelp"><keyboard-shortcut first-keystroke="shift alt f1"/></action>
<action id="CopyElement"><keyboard-shortcut first-keystroke="f5"/></action>
<action id="CopyPaths"><keyboard-shortcut first-keystroke="shift alt c"/></action>
<action id="CurrentLeadUnfollowAction"><keyboard-shortcut first-keystroke="shift ctrl alt y"/></action>
<action id="DatabaseView.DropAction"><keyboard-shortcut first-keystroke="delete"/></action>
<action id="DatabaseView.PropertiesAction"><keyboard-shortcut first-keystroke="alt i"/></action>
<action id="Debug"><keyboard-shortcut first-keystroke="ctrl d"/></action>
<action id="DebugClass"><keyboard-shortcut first-keystroke="shift ctrl d"/></action>
<action id="Diff.ApplyLeftSide"><keyboard-shortcut first-keystroke="ctrl alt right"/></action>
<action id="Diff.ApplyRightSide"><keyboard-shortcut first-keystroke="ctrl alt left"/></action>
<action id="Diff.FocusOppositePane"><keyboard-shortcut first-keystroke="shift ctrl tab"/></action>
<action id="Diff.NextChange"><keyboard-shortcut first-keystroke="shift ctrl right"/></action>
<action id="Diff.PrevChange"><keyboard-shortcut first-keystroke="shift ctrl left"/></action>
<action id="Diff.ShowDiff"><keyboard-shortcut first-keystroke="alt d"/></action>
<action id="Diff.ShowSettingsPopup"><keyboard-shortcut first-keystroke="shift alt d"/></action>
<action id="DirDiffMenu.SynchronizeDiff"><keyboard-shortcut first-keystroke="enter"/></action>
<action id="DirDiffMenu.SynchronizeDiff.All"><keyboard-shortcut first-keystroke="alt enter"/></action>
<action id="Docker.RemoteServers.StartComposeService"><keyboard-shortcut first-keystroke="alt enter"/></action>
<action id="DomCollectionControl.Add"><keyboard-shortcut first-keystroke="insert"/></action>
<action id="DuplicatesForm.SendToLeft"><keyboard-shortcut first-keystroke="alt 1"/></action>
<action id="DuplicatesForm.SendToRight"><keyboard-shortcut first-keystroke="alt 2"/></action>
<action id="EditBreakpoint"><keyboard-shortcut first-keystroke="shift alt f8"/></action>
<action id="EditSource"><keyboard-shortcut first-keystroke="alt down"/><keyboard-shortcut first-keystroke="f4"/></action>
<action id="EditSourceInNewWindow"><keyboard-shortcut first-keystroke="shift f4"/></action>
<action id="EditorBackSpace"><keyboard-shortcut first-keystroke="back_space"/><keyboard-shortcut first-keystroke="shift back_space"/></action>
<action id="EditorChooseLookupItem"><keyboard-shortcut first-keystroke="enter"/></action>
<action id="EditorChooseLookupItemDot"><keyboard-shortcut first-keystroke="ctrl period"/></action>
<action id="EditorChooseLookupItemReplace"><keyboard-shortcut first-keystroke="tab"/></action>
<action id="EditorCompleteStatement"><keyboard-shortcut first-keystroke="shift alt enter"/></action>
<action id="EditorContextInfo"><keyboard-shortcut first-keystroke="shift ctrl q"/></action>
<action id="EditorCreateRectangularSelection"><mouse-shortcut keystroke="shift alt button2"/></action>
<action id="EditorCutLineEnd"><keyboard-shortcut first-keystroke="ctrl k"/></action>
<action id="EditorDecreaseFontSizeGlobal"><keyboard-shortcut first-keystroke="shift ctrl comma"/></action>
<action id="EditorDelete"><keyboard-shortcut first-keystroke="delete"/></action>
<action id="EditorDeleteLine"><keyboard-shortcut first-keystroke="alt back_space"/></action>
<action id="EditorDown"><keyboard-shortcut first-keystroke="down"/><keyboard-shortcut first-keystroke="ctrl n"/></action>
<action id="EditorDownWithSelection"><keyboard-shortcut first-keystroke="shift down"/></action>
<action id="EditorDuplicate"><keyboard-shortcut first-keystroke="alt d"/></action>
<action id="EditorEnter"><keyboard-shortcut first-keystroke="enter"/></action>
<action id="EditorEscape"><keyboard-shortcut first-keystroke="escape"/></action>
<action id="EditorFocusGutter"><keyboard-shortcut first-keystroke="shift alt 6" second-keystroke="f"/></action>
<action id="EditorIncreaseFontSizeGlobal"><keyboard-shortcut first-keystroke="shift ctrl period"/></action>
<action id="EditorIndentSelection"><keyboard-shortcut first-keystroke="tab"/></action>
<action id="EditorJoinLines"><keyboard-shortcut first-keystroke="shift ctrl j"/></action>
<action id="EditorLeft"><keyboard-shortcut first-keystroke="left"/><keyboard-shortcut first-keystroke="ctrl b"/></action>
<action id="EditorLeftWithSelection"><keyboard-shortcut first-keystroke="shift left"/></action>
<action id="EditorLineEnd"><keyboard-shortcut first-keystroke="end"/><keyboard-shortcut first-keystroke="alt right"/><keyboard-shortcut first-keystroke="ctrl e"/></action>
<action id="EditorLineEndWithSelection"><keyboard-shortcut first-keystroke="shift end"/><keyboard-shortcut first-keystroke="shift alt right"/></action>
<action id="EditorLineStart"><keyboard-shortcut first-keystroke="home"/><keyboard-shortcut first-keystroke="alt left"/><keyboard-shortcut first-keystroke="ctrl a"/></action>
<action id="EditorLineStartWithSelection"><keyboard-shortcut first-keystroke="shift home"/><keyboard-shortcut first-keystroke="shift alt left"/></action>
<action id="EditorMatchBrace"><keyboard-shortcut first-keystroke="ctrl m"/></action>
<action id="EditorMoveToPageBottom"><keyboard-shortcut first-keystroke="alt page_down"/></action>
<action id="EditorMoveToPageBottomWithSelection"><keyboard-shortcut first-keystroke="shift alt page_down"/></action>
<action id="EditorMoveToPageTop"><keyboard-shortcut first-keystroke="alt page_up"/></action>
<action id="EditorMoveToPageTopWithSelection"><keyboard-shortcut first-keystroke="shift alt page_up"/></action>
<action id="EditorPageDown"><keyboard-shortcut first-keystroke="page_down"/></action>
<action id="EditorPageDownWithSelection"><keyboard-shortcut first-keystroke="shift page_down"/></action>
<action id="EditorPageUp"><keyboard-shortcut first-keystroke="page_up"/></action>
<action id="EditorPageUpWithSelection"><keyboard-shortcut first-keystroke="shift page_up"/></action>
<action id="EditorPasteFromX11"><mouse-shortcut keystroke="button2"/></action>
<action id="EditorRight"><keyboard-shortcut first-keystroke="right"/><keyboard-shortcut first-keystroke="ctrl f"/></action>
<action id="EditorRightWithSelection"><keyboard-shortcut first-keystroke="shift right"/></action>
<action id="EditorScrollToCenter"><keyboard-shortcut first-keystroke="ctrl l"/></action>
<action id="EditorShowGutterIconTooltip"><keyboard-shortcut first-keystroke="shift alt 6" second-keystroke="t"/></action>
<action id="EditorSplitLine"><keyboard-shortcut first-keystroke="alt enter"/></action>
<action id="EditorStartNewLine"><keyboard-shortcut first-keystroke="shift enter"/></action>
<action id="EditorTab"><keyboard-shortcut first-keystroke="tab"/></action>
<action id="EditorTextEnd"><keyboard-shortcut first-keystroke="alt end"/></action>
<action id="EditorTextEndWithSelection"><keyboard-shortcut first-keystroke="shift alt end"/></action>
<action id="EditorTextStart"><keyboard-shortcut first-keystroke="alt home"/></action>
<action id="EditorTextStartWithSelection"><keyboard-shortcut first-keystroke="shift alt home"/></action>
<action id="EditorToggleCase"><keyboard-shortcut first-keystroke="shift alt u"/></action>
<action id="EditorToggleColumnMode"><keyboard-shortcut first-keystroke="shift alt 8"/></action>
<action id="EditorToggleInsertState"><keyboard-shortcut first-keystroke="insert"/></action>
<action id="EditorUnindentSelection"><keyboard-shortcut first-keystroke="shift tab"/></action>
<action id="EditorUp"><keyboard-shortcut first-keystroke="up"/><keyboard-shortcut first-keystroke="ctrl p"/></action>
<action id="EditorUpWithSelection"><keyboard-shortcut first-keystroke="shift up"/></action>
<action id="EmojiAndSymbols"><keyboard-shortcut first-keystroke="ctrl alt space"/></action>
<action id="EscUnfollowUserAction"><keyboard-shortcut first-keystroke="escape"/></action>
<action id="Exit"><keyboard-shortcut first-keystroke="alt q"/></action>
<action id="ExpandAll"><keyboard-shortcut first-keystroke="alt add"/><keyboard-shortcut first-keystroke="alt equals"/></action>
<action id="ExpandAllRegions"><keyboard-shortcut first-keystroke="shift alt add"/><keyboard-shortcut first-keystroke="shift alt equals"/></action>
<action id="ExpandExpandableComponent"><keyboard-shortcut first-keystroke="shift enter"/><keyboard-shortcut first-keystroke="alt add"/><keyboard-shortcut first-keystroke="alt equals"/></action>
<action id="ExpandLiveTemplateByTab"><keyboard-shortcut first-keystroke="tab"/></action>
<action id="ExpandRegion"><keyboard-shortcut first-keystroke="alt add"/><keyboard-shortcut first-keystroke="alt equals"/></action>
<action id="ExpandToLevel1"><keyboard-shortcut first-keystroke="alt multiply" second-keystroke="1"/><keyboard-shortcut first-keystroke="alt multiply" second-keystroke="numpad1"/></action>
<action id="ExpandToLevel2"><keyboard-shortcut first-keystroke="alt multiply" second-keystroke="2"/><keyboard-shortcut first-keystroke="alt multiply" second-keystroke="numpad2"/></action>
<action id="ExpandToLevel3"><keyboard-shortcut first-keystroke="alt multiply" second-keystroke="3"/><keyboard-shortcut first-keystroke="alt multiply" second-keystroke="numpad3"/></action>
<action id="ExpandToLevel4"><keyboard-shortcut first-keystroke="alt multiply" second-keystroke="4"/><keyboard-shortcut first-keystroke="alt multiply" second-keystroke="numpad4"/></action>
<action id="ExpandToLevel5"><keyboard-shortcut first-keystroke="alt multiply" second-keystroke="5"/><keyboard-shortcut first-keystroke="alt multiply" second-keystroke="numpad5"/></action>
<action id="ExpandTreeNode"><keyboard-shortcut first-keystroke="add"/></action>
<action id="ExportToTextFile"><keyboard-shortcut first-keystroke="ctrl o"/></action>
<action id="ExpressionTypeInfo"><keyboard-shortcut first-keystroke="shift ctrl p"/></action>
<action id="ExternalJavaDoc"><keyboard-shortcut first-keystroke="shift f1"/></action>
<action id="ExternalSystem.ProjectRefreshAction"><keyboard-shortcut first-keystroke="shift alt i"/></action>
<action id="FileChooser.GotoDesktop"><keyboard-shortcut first-keystroke="alt d"/></action>
<action id="FileChooser.GotoHome"><keyboard-shortcut first-keystroke="alt 1"/></action>
<action id="FileChooser.GotoModule"><keyboard-shortcut first-keystroke="alt 3"/></action>
<action id="FileChooser.GotoProject"><keyboard-shortcut first-keystroke="alt 2"/></action>
<action id="FileChooser.TogglePathBar"><keyboard-shortcut first-keystroke="alt p"/></action>
<action id="FileChooserList.LevelUp"><keyboard-shortcut first-keystroke="alt up"/></action>
<action id="FileChooserList.Root"><keyboard-shortcut first-keystroke="alt slash"/></action>
<action id="FileStructurePopup"><keyboard-shortcut first-keystroke="alt f12"/></action>
<action id="Find"><keyboard-shortcut first-keystroke="alt f"/></action>
<action id="FindInPath"><keyboard-shortcut first-keystroke="shift alt f"/></action>
<action id="FindNext"><keyboard-shortcut first-keystroke="alt g"/></action>
<action id="FindPrevious"><keyboard-shortcut first-keystroke="shift alt g"/></action>
<action id="FindUsagesInFile"><keyboard-shortcut first-keystroke="alt f7"/></action>
<action id="FocusEditor"><keyboard-shortcut first-keystroke="escape"/></action>
<action id="ForceOthersToFollowAction"><keyboard-shortcut first-keystroke="shift ctrl o"/></action>
<action id="FullyExpandTreeNode"><keyboard-shortcut first-keystroke="multiply"/></action>
<action id="Generate"><keyboard-shortcut first-keystroke="alt n"/><keyboard-shortcut first-keystroke="ctrl enter"/></action>
<action id="Git.Log.Branches.Change.Branch.Filter"><mouse-shortcut keystroke="button1 doubleClick"/><keyboard-shortcut first-keystroke="enter"/></action>
<action id="Git.Rename.Local.Branch"><keyboard-shortcut first-keystroke="f2"/><keyboard-shortcut first-keystroke="shift f6"/></action>
<action id="Git.Reword.Commit"><keyboard-shortcut first-keystroke="f2"/><keyboard-shortcut first-keystroke="shift f6"/></action>
<action id="Github.PullRequest.Diff.Comment.Create"><keyboard-shortcut first-keystroke="shift alt x"/></action>
<action id="GotoAction"><keyboard-shortcut first-keystroke="shift alt a"/></action>
<action id="GotoBookmark0"><keyboard-shortcut first-keystroke="ctrl 0"/></action>
<action id="GotoBookmark1"><keyboard-shortcut first-keystroke="ctrl 1"/></action>
<action id="GotoBookmark2"><keyboard-shortcut first-keystroke="ctrl 2"/></action>
<action id="GotoBookmark3"><keyboard-shortcut first-keystroke="ctrl 3"/></action>
<action id="GotoBookmark4"><keyboard-shortcut first-keystroke="ctrl 4"/></action>
<action id="GotoBookmark5"><keyboard-shortcut first-keystroke="ctrl 5"/></action>
<action id="GotoBookmark6"><keyboard-shortcut first-keystroke="ctrl 6"/></action>
<action id="GotoBookmark7"><keyboard-shortcut first-keystroke="ctrl 7"/></action>
<action id="GotoBookmark8"><keyboard-shortcut first-keystroke="ctrl 8"/></action>
<action id="GotoBookmark9"><keyboard-shortcut first-keystroke="ctrl 9"/></action>
<action id="GotoClass"><keyboard-shortcut first-keystroke="alt o"/></action>
<action id="GotoDeclaration"><keyboard-shortcut first-keystroke="alt b"/><mouse-shortcut keystroke="alt button1"/><mouse-shortcut keystroke="Force touch"/><mouse-shortcut keystroke="button2"/></action>
<action id="GotoFile"><keyboard-shortcut first-keystroke="shift alt o"/></action>
<action id="GotoLine"><keyboard-shortcut first-keystroke="alt l"/></action>
<action id="GotoNextElementUnderCaretUsage"><keyboard-shortcut first-keystroke="ctrl alt down"/></action>
<action id="GotoNextError"><keyboard-shortcut first-keystroke="f2"/></action>
<action id="GotoPreviousError"><keyboard-shortcut first-keystroke="shift f2"/></action>
<action id="GotoRelated"><keyboard-shortcut first-keystroke="ctrl alt up"/></action>
<action id="GotoSuperMethod"><keyboard-shortcut first-keystroke="alt u"/></action>
<action id="GotoTest"><keyboard-shortcut first-keystroke="shift alt t"/></action>
<action id="GotoTypeDeclaration"><keyboard-shortcut first-keystroke="shift alt b"/><keyboard-shortcut first-keystroke="shift ctrl b"/><mouse-shortcut keystroke="shift alt button1"/></action>
<action id="GotoUrlAction"><keyboard-shortcut first-keystroke="shift alt back_slash"/></action>
<action id="Graph.ActualSize"><keyboard-shortcut first-keystroke="alt divide"/><keyboard-shortcut first-keystroke="alt slash"/></action>
<action id="Graph.AlignNodes.Bottom"><keyboard-shortcut first-keystroke="shift b"/></action>
<action id="Graph.AlignNodes.Center"><keyboard-shortcut first-keystroke="shift c"/></action>
<action id="Graph.AlignNodes.Left"><keyboard-shortcut first-keystroke="shift l"/></action>
<action id="Graph.AlignNodes.Middle"><keyboard-shortcut first-keystroke="shift m"/></action>
<action id="Graph.AlignNodes.Right"><keyboard-shortcut first-keystroke="shift r"/></action>
<action id="Graph.AlignNodes.Top"><keyboard-shortcut first-keystroke="shift t"/></action>
<action id="Graph.ApplyCurrentLayout"><keyboard-shortcut first-keystroke="shift f5"/></action>
<action id="Graph.DistributeNodes.Horizontally"><keyboard-shortcut first-keystroke="shift h"/></action>
<action id="Graph.DistributeNodes.Vertically"><keyboard-shortcut first-keystroke="shift v"/></action>
<action id="Graph.RouteEdges"><keyboard-shortcut first-keystroke="f5"/></action>
<action id="Graph.ZoomIn"><keyboard-shortcut first-keystroke="add"/><keyboard-shortcut first-keystroke="equals"/></action>
<action id="Graph.ZoomOut"><keyboard-shortcut first-keystroke="subtract"/><keyboard-shortcut first-keystroke="minus"/></action>
<action id="HideActiveWindow"><keyboard-shortcut first-keystroke="shift escape"/></action>
<action id="HideAllWindows"><keyboard-shortcut first-keystroke="shift alt f12"/></action>
<action id="HighlightUsagesInFile"><keyboard-shortcut first-keystroke="shift alt f7"/></action>
<action id="Images.Editor.ActualSize"><keyboard-shortcut first-keystroke="alt divide"/><keyboard-shortcut first-keystroke="alt slash"/></action>
<action id="Images.Editor.ToggleGrid"><keyboard-shortcut first-keystroke="alt quote"/></action>
<action id="Images.Thumbnails.EnterAction"><keyboard-shortcut first-keystroke="enter"/></action>
<action id="Images.Thumbnails.UpFolder"><keyboard-shortcut first-keystroke="back_space"/></action>
<action id="ImplementMethods"><keyboard-shortcut first-keystroke="ctrl i"/></action>
<action id="InsertLiveTemplate"><keyboard-shortcut first-keystroke="alt j"/></action>
<action id="InsertRubyInjection"><keyboard-shortcut first-keystroke="shift alt period"/></action>
<action id="InsertRubyInjectionWithoutOutput"><keyboard-shortcut first-keystroke="shift alt comma"/></action>
<action id="Jdbc.OpenConsole.New.Generate"/>
<action id="JumpToLastChange"><keyboard-shortcut first-keystroke="shift alt back_space"/></action>
<action id="JumpToLastWindow"><keyboard-shortcut first-keystroke="f12"/></action>
<action id="Markdown.Styling.CreateLink"><keyboard-shortcut first-keystroke="shift alt u"/></action>
<action id="MaximizeToolWindow"><keyboard-shortcut first-keystroke="shift alt quote"/></action>
<action id="MethodDown"><keyboard-shortcut first-keystroke="shift ctrl down"/></action>
<action id="MethodHierarchy"><keyboard-shortcut first-keystroke="shift alt h"/></action>
<action id="MethodUp"><keyboard-shortcut first-keystroke="shift ctrl up"/></action>
<action id="MinimizeCurrentWindow"><keyboard-shortcut first-keystroke="alt m"/></action>
<action id="Move"><keyboard-shortcut first-keystroke="f6"/></action>
<action id="MoveStatementDown"><keyboard-shortcut first-keystroke="shift alt down"/></action>
<action id="MoveStatementUp"><keyboard-shortcut first-keystroke="shift alt up"/></action>
<action id="NewElement"><keyboard-shortcut first-keystroke="alt n"/><keyboard-shortcut first-keystroke="ctrl enter"/></action>
<action id="NewElementSamePlace"><keyboard-shortcut first-keystroke="ctrl alt n"/></action>
<action id="NewScratchFile"><keyboard-shortcut first-keystroke="shift alt n"/></action>
<action id="NextDiff"><keyboard-shortcut first-keystroke="f7"/></action>
<action id="NextEditorTab"><keyboard-shortcut first-keystroke="shift ctrl right"/></action>
<action id="NextParameter"><keyboard-shortcut first-keystroke="tab"/></action>
<action id="NextSplitter"><keyboard-shortcut first-keystroke="alt tab"/></action>
<action id="NextTab"><keyboard-shortcut first-keystroke="shift alt close_bracket"/><keyboard-shortcut first-keystroke="ctrl right"/></action>
<action id="NextTemplateVariable"><keyboard-shortcut first-keystroke="tab"/><keyboard-shortcut first-keystroke="enter"/></action>
<action id="NextWindow"><keyboard-shortcut first-keystroke="alt back_quote"/></action>
<action id="OpenInRightSplit"><keyboard-shortcut first-keystroke="shift enter"/><mouse-shortcut keystroke="alt button1 doubleClick"/></action>
<action id="OptimizeImports"><keyboard-shortcut first-keystroke="ctrl alt o"/></action>
<action id="OverrideMethods"><keyboard-shortcut first-keystroke="ctrl o"/></action>
<action id="ParameterInfo"><keyboard-shortcut first-keystroke="alt p"/></action>
<action id="PasteMultiple"><keyboard-shortcut first-keystroke="shift alt v"/><keyboard-shortcut first-keystroke="shift alt insert"/></action>
<action id="PrevParameter"><keyboard-shortcut first-keystroke="shift tab"/></action>
<action id="PrevSplitter"><keyboard-shortcut first-keystroke="shift alt tab"/></action>
<action id="PreviousDiff"><keyboard-shortcut first-keystroke="shift f7"/></action>
<action id="PreviousEditorTab"><keyboard-shortcut first-keystroke="shift ctrl left"/></action>
<action id="PreviousTab"><keyboard-shortcut first-keystroke="shift alt open_bracket"/><keyboard-shortcut first-keystroke="ctrl left"/></action>
<action id="PreviousTemplateVariable"><keyboard-shortcut first-keystroke="shift tab"/></action>
<action id="PreviousWindow"><keyboard-shortcut first-keystroke="shift alt back_quote"/></action>
<action id="QuickChangeScheme"><keyboard-shortcut first-keystroke="ctrl back_quote"/></action>
<action id="QuickJavaDoc"><keyboard-shortcut first-keystroke="f1"/><keyboard-shortcut first-keystroke="ctrl j"/><mouse-shortcut keystroke="control button2"/></action>
<action id="QuickPreview"><keyboard-shortcut first-keystroke="space"/></action>
<action id="RecentFiles"><keyboard-shortcut first-keystroke="alt e"/></action>
<action id="RecentLocations"><keyboard-shortcut first-keystroke="shift alt e"/></action>
<action id="Refactorings.QuickListPopupAction"><keyboard-shortcut first-keystroke="ctrl t"/></action>
<action id="ReformatCode"><keyboard-shortcut first-keystroke="shift alt l"/></action>
<action id="Refresh"><keyboard-shortcut first-keystroke="alt r"/></action>
<action id="RenameElement"><keyboard-shortcut first-keystroke="shift f6"/></action>
<action id="Replace"><keyboard-shortcut first-keystroke="alt r"/></action>
<action id="ReplaceInPath"><keyboard-shortcut first-keystroke="shift alt r"/></action>
<action id="Rerun"><keyboard-shortcut first-keystroke="alt r"/></action>
<action id="ResetIdeScaleAction"><keyboard-shortcut first-keystroke="ctrl alt 0"/></action>
<action id="ResizeToolWindowDown"><keyboard-shortcut first-keystroke="ctrl alt down"/></action>
<action id="RestoreDefaultLayout"><keyboard-shortcut first-keystroke="shift f12"/></action>
<action id="Run"><keyboard-shortcut first-keystroke="ctrl r"/></action>
<action id="RunClass"><keyboard-shortcut first-keystroke="shift ctrl r"/></action>
<action id="RunJsbtTask"><keyboard-shortcut first-keystroke="alt f11"/></action>
<action id="SafeDelete"><keyboard-shortcut first-keystroke="alt delete"/></action>
<action id="SaveAll"><keyboard-shortcut first-keystroke="alt s"/></action>
<action id="SearchEverywhere.CompleteCommand"><keyboard-shortcut first-keystroke="tab"/></action>
<action id="SearchEverywhere.NavigateToNextGroup"><keyboard-shortcut first-keystroke="page_down"/><keyboard-shortcut first-keystroke="alt down"/></action>
<action id="SearchEverywhere.NavigateToPrevGroup"><keyboard-shortcut first-keystroke="page_up"/><keyboard-shortcut first-keystroke="alt up"/></action>
<action id="SearchEverywhere.NextTab"><keyboard-shortcut first-keystroke="tab"/></action>
<action id="SearchEverywhere.PrevTab"><keyboard-shortcut first-keystroke="shift tab"/></action>
<action id="SearchEverywhere.SelectItem"><keyboard-shortcut first-keystroke="enter"/></action>
<action id="SelectAllOccurrences"><keyboard-shortcut first-keystroke="ctrl alt g"/></action>
<action id="SelectNextOccurrence"><keyboard-shortcut first-keystroke="ctrl g"/></action>
<action id="SendEOF"><keyboard-shortcut first-keystroke="alt d"/></action>
<action id="ServiceView.GroupByContributor"><keyboard-shortcut first-keystroke="ctrl t"/></action>
<action id="ServiceView.GroupByServiceGroups"><keyboard-shortcut first-keystroke="ctrl p"/></action>
<action id="ServiceView.ShowServices"><keyboard-shortcut first-keystroke="shift alt t"/></action>
<action id="ShelveChanges.UnshelveWithDialog"><keyboard-shortcut first-keystroke="shift alt u"/></action>
<action id="ShelvedChanges.Rename"><keyboard-shortcut first-keystroke="f2"/><keyboard-shortcut first-keystroke="shift f6"/></action>
<action id="ShowBookmarks"><keyboard-shortcut first-keystroke="alt f3"/></action>
<action id="ShowContent"><keyboard-shortcut first-keystroke="shift ctrl down"/></action>
<action id="ShowErrorDescription"><keyboard-shortcut first-keystroke="alt f1"/></action>
<action id="ShowProjectStructureSettings"><keyboard-shortcut first-keystroke="alt semicolon"/></action>
<action id="ShowSettings"><keyboard-shortcut first-keystroke="alt comma"/></action>
<action id="SingleUserFollowAction"><keyboard-shortcut first-keystroke="shift ctrl alt y"/></action>
<action id="SmartStepInto"><keyboard-shortcut first-keystroke="shift f7"/></action>
<action id="SmartTypeCompletion"><keyboard-shortcut first-keystroke="shift ctrl space"/></action>
<action id="Space.Review.CreateDiffComment"><keyboard-shortcut first-keystroke="shift alt x"/></action>
<action id="SplitChooser.Duplicate"><keyboard-shortcut first-keystroke="alt enter"/></action>
<action id="SplitChooser.NextWindow"><keyboard-shortcut first-keystroke="tab"/></action>
<action id="SplitChooser.PreviousWindow"><keyboard-shortcut first-keystroke="shift tab"/></action>
<action id="SplitChooser.Split"><keyboard-shortcut first-keystroke="enter"/></action>
<action id="SplitChooser.SplitCenter"><keyboard-shortcut first-keystroke="space"/></action>
<action id="StepInto"><keyboard-shortcut first-keystroke="f7"/></action>
<action id="StepOut"><keyboard-shortcut first-keystroke="shift f8"/></action>
<action id="StepOver"><keyboard-shortcut first-keystroke="f8"/></action>
<action id="Stop"><keyboard-shortcut first-keystroke="alt f2"/></action>
<action id="StopBackgroundProcesses"><keyboard-shortcut first-keystroke="shift alt f2"/></action>
<action id="SwitchHeaderSource"><keyboard-shortcut first-keystroke="f10"/></action>
<action id="Switcher"><keyboard-shortcut first-keystroke="ctrl tab"/><keyboard-shortcut first-keystroke="shift ctrl tab"/></action>
<action id="SwitcherIterateItems"><keyboard-shortcut first-keystroke="alt e"/></action>
<action id="SwitcherRecentEditedChangedToggleCheckBox"><keyboard-shortcut first-keystroke="alt e"/></action>
<action id="Table-startEditing"><keyboard-shortcut first-keystroke="f2"/></action>
<action id="Terminal.ClearBuffer"><keyboard-shortcut first-keystroke="alt l"/><keyboard-shortcut first-keystroke="alt k"/></action>
<action id="Terminal.CopySelectedText"><keyboard-shortcut first-keystroke="alt c"/><keyboard-shortcut first-keystroke="alt insert"/></action>
<action id="Terminal.NewTab"><keyboard-shortcut first-keystroke="alt t"/></action>
<action id="Terminal.Paste"><keyboard-shortcut first-keystroke="alt v"/><keyboard-shortcut first-keystroke="shift insert"/></action>
<action id="Terminal.SelectAll"><keyboard-shortcut first-keystroke="alt a"/></action>
<action id="Terminal.SmartCommandExecution.Debug"><keyboard-shortcut first-keystroke="shift alt enter"/></action>
<action id="Terminal.SmartCommandExecution.Run"><keyboard-shortcut first-keystroke="alt enter"/></action>
<action id="Terminal.SwitchFocusToEditor"><keyboard-shortcut first-keystroke="escape"/></action>
</keymap>
```

## 苹果键盘的功能键行为

https://help.ubuntu.com/community/AppleKeyboard#Change_Function_Key_behavior
