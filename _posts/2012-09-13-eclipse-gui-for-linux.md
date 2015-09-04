---
layout: post
title : Linux下Eclipse的界面调整
category : linux
tags : [linux,eclipse]
---
{% include JB/setup %}

Eclipse在Linux的图标以及图标间距都比较大,导致界面整体布局不够windows下来的美观,而且在原本屏幕就不大的笔记本上,显得空间利用率下降了n次方.

那么我们只能忍受这种丑陋的界面布局吗?其实一开始我觉得用久了会习惯的,但是每次看到别人的Eclipse界面布局那么紧凑美观,还是心里会有略微的不爽.

Google了一下,尝试了2种方案,第一种在debian6下以失败告终,第二种方案成功.遂记录如下,以备后用.

步骤一

创建自定义界面文件.gtkrc4eclipse,文件内容如下

	style "gtkcompact" {
	 font_name="Sans 8"
	 GtkButton::default_border={0,0,0,0}
	 GtkButton::default_outside_border={0,0,0,0}
	 GtkButtonBox::child_min_width=0
	 GtkButtonBox::child_min_heigth=0
	 GtkButtonBox::child_internal_pad_x=0
	 GtkButtonBox::child_internal_pad_y=0
	 GtkMenu::vertical-padding=1
	 GtkMenuBar::internal_padding=0
	 GtkMenuItem::horizontal_padding=4
	 GtkToolbar::internal-padding=0
	 GtkToolbar::space-size=0
	 GtkOptionMenu::indicator_size=0
	 GtkOptionMenu::indicator_spacing=0
	 GtkPaned::handle_size=4
	 GtkRange::trough_border=0
	 GtkRange::stepper_spacing=0
	 GtkScale::value_spacing=0
	 GtkScrolledWindow::scrollbar_spacing=0
	 GtkExpander::expander_size=10
	 GtkExpander::expander_spacing=0
	 GtkTreeView::vertical-separator=0
	 GtkTreeView::horizontal-separator=0
	 GtkTreeView::expander-size=10
	 GtkTreeView::fixed-height-mode=TRUE
	 GtkWidget::focus_padding=0
	}
	class "GtkWidget" style "gtkcompact"
	style "gtkcompactextra" {
	 xthickness=0
	 ythickness=0
	}
	class "GtkButton" style "gtkcompactextra"
	class "GtkToolbar" style "gtkcompactextra"
	class "GtkPaned" style "gtkcompactextra"

步骤二

创建启动脚本eclipse.sh,内容如下

```bash
#!/bin/sh
export MY_ECLIPSE_GTKRC=/path/to/gtkrc4eclipse
export GTK_RC_FILES=$GTK_RC_FILES:$MY_ECLIPSE_GTKRC
export GTK2_RC_FILES=$GTK2_RC_FILES:$MY_ECLIPSE_GTKRC
/path/to/eclipse &
```

