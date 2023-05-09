layout: page
title: "WPF DataGrid一个编辑单元格容易崩溃bug的改进办法"
permalink: /wpf_datagrid_bug.md

# WPF DataGrid一个编辑单元格容易崩溃bug的改进办法

WPF DataGrid在自动生成列情况下，编辑单元格时如果进行了ItemSource切换就会导致程序崩溃，这是WPF DataGrid的一个bug，改进办法就是在每次切换ItemSouce前执行：

datagrid.CommitEdit(DataGridEditingUnit.Row, true);

目前这个bug还没修复，只能采取此临时解决办法。
