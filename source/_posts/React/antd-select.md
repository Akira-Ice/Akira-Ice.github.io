---
title: Antd-Select-custom
date: 2023/10/8 17:07:00
updated: 2023/10/8 17:07:00
categories:
  - [React]
tags:
---

# 需求

选择下拉框 -> 补全选项后的条件 -> 生成 tag。

手动控制 open，选择后关闭下拉。

# 实现

```tsx
import { message, Select } from "antd";
import React, { useState } from "react";

const OPTIONS = ["Apples", "Nails", "Bananas", "Helicopters"];

const ToolMgt: React.FC = () => {
  /** 已选项 */
  const [selectedItems, setSelectedItems] = useState<string[]>([]);
  /** 输入框 value */
  const [searchValue, setSearchValue] = useState("");
  const [open, setOpen] = useState(false);

  const selectKeys = [...new Set(selectedItems.map((item) => item.split(":")[0]))];
  /** 下拉框 list - 剔除已选 */
  const filteredOptions = OPTIONS.filter((o) => !selectKeys.includes(o)).map((item) => ({
    value: item,
    label: item,
  }));

  function handleSearch(value: string) {
    setSearchValue(value);
  }

  function handleSelect(value: string | number) {
    setSearchValue(value + ":");
    setOpen(false);
  }

  function handleEnter(e: React.KeyboardEvent<HTMLDivElement>) {
    if (e.key !== "Enter") return;
    /** 不包含':' 属于未选择或选择后删除':' 属于意外终止 enter */
    if (!searchValue.includes(":")) return;
    /** 校验 */
    if (!verifySearch(searchValue, OPTIONS)) {
      message.error("规则不符合");
      return;
    }

    setSelectedItems([...selectedItems, searchValue]);
    setSearchValue("");
  }

  function handleDeselect(value: string | number) {
    setSelectedItems(selectedItems.filter((item) => item != value));
  }

  /** 输入校验 */
  function verifySearch(searchValue: string, initOptions: string[]) {
    /** 分割连接符 */
    const searchValueArr = searchValue.split(":");
    const isInclude = initOptions.includes(searchValueArr[0]);

    if (searchValueArr.length != 2) return false;
    if (searchValueArr[1] == "") return false;
    if (!isInclude) return false;

    return true;
  }

  return (
    <>
      {searchValue}
      <br />
      <Select
        open={open && !searchValue.includes(":")}
        mode="multiple"
        placeholder="Inserted are removed"
        value={selectedItems}
        searchValue={searchValue}
        autoClearSearchValue={false}
        style={{ width: "50%" }}
        options={filteredOptions}
        onDeselect={handleDeselect}
        onSearch={handleSearch}
        onSelect={handleSelect}
        onKeyDown={handleEnter}
        onDropdownVisibleChange={(visible) => setOpen(visible)}
      />
    </>
  );
};

export default ToolMgt;
```

# 效果

![636769ef-6dc9-4de0-ae08-eed996543325.gif](https://s2.loli.net/2023/10/09/yJIkWMdV5pALquE.gif)
