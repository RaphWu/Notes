---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# 變更摘要

1. 實體類別已更新，採用 IEquatable<T> 實現
所有實體類別均已更新，具體如下：
• 使用主鍵 (Id) 進行相等性比較的 IEquatable<T> 實現
• 重寫了 Equals() 和 GetHashCode() 方法
• 重載了 == 和 != 運算符
• 編寫了清晰一致的 XML 註釋
• 使用 HashSet<T> 正確初始化集合

## 核心實體已更新

• EmployeeEntity - 新增了 EmployeeGroups 和 EmployeePermissions 導航屬性

• DepartmentEntity - 新增了 Employees 和 DepartmentPermissions 導航屬性

• JobTitleEntity - 新增了 Employees 導航屬性

• EmployeeStatusEntity - 新增了 Employees 導航屬性

• MachineEntity - 將 Workstations 變更為 MachineWorkstations

• WorkstationEntity - 新增了 MachineWorkstations 導航屬性

• ModelEntity - 修正了拼字錯誤 Workstations → Workstations

• MachineNameEntity、MachineTypeEntity、MachineCategoryEntity、MachineCondition、MachineBrandEntity、MachineLocationEntity MachineIssueCategoryEntity、ModelStatusEntity

• GroupEntity - 新增了 EmployeeGroups 和 GroupPermissions 導覽屬性

• PermissionEntity - 新增了 EmployeePermissions、DepartmentPermissions 和 GroupPermissions 導覽屬性

MaintiFlow 實體已更新：

• WorkOrderEntity

• MaintenanceUnitEntity

• MaintiFlowIssueCategoryEntity

• WorkOrderEngineer - 修正了外鍵命名（使用 WorkOrderId 而非 TaskOrderId）

2. 建立新的中間表（多對多關係）

• EmployeeCarbonCopy - 員工 ↔ 員工自引用

• MachineWorkstation - 機器 ↔ 工作站

• EmployeeGroup - 員工 ↔ 群組

• EmployeePermission - 員工 ↔ 權限

• DepartmentPermission - 部門 ↔ 權限

• GroupPermission - 群組 ↔ 權限

3. Calin.TaskPulse.Core.DB 命名空間中的 Fluent API 配置

DbContext 類別：

• CoreContext - 包含所有核心實體

• MaintiFlowContext - 包含所有 MaintiFlow 實體

實體配置類別（21 個檔案）：

• 每個實體都有自己的 EntityTypeConfiguration<T> 類

• 所有關係均已明確配置

• 所有外鍵均使用 WillCascadeOnDelete(false)（SQLite 要求）

• 表名已明確指定

• 屬性約束（MaxLength、Required）已定義
