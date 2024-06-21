---
title: '[SQL] T-SQL 中的 CTE 介紹與應用'
date: 2024-06-21 09:45:22
updated: 2024-06-21 09:45:22
categories:
- Database
- SqlServer
tags:
- Database
- SqlServer
- TSQL
thumbnail:
---

**CTE的簡介** Common Table Expressions（CTEs）是一種在SQL查詢中定義臨時結果集的方法。CTE在T-SQL中的應用可以提高查詢的可讀性和維護性，使得複雜的查詢邏輯更加清晰。
**為何使用CTE** 使用CTE的主要原因在於它可以讓我們將查詢邏輯分解成可讀性更強的小塊。CTE在遞迴查詢和處理分層數據時特別有用。

> 本篇由 ChatGPT 產生

<!-- more -->

## CTE的基礎概念

**CTE的語法** CTE的語法如下：

```sql
WITH CTE名稱 (欄位1, 欄位2, ...)
AS
(
    -- 查詢定義
    SELECT ...
    FROM ...
    WHERE ...
)
-- 使用CTE
SELECT ...
FROM CTE名稱
WHERE ...
```

**基本CTE範例** 假設我們有一個名為`Employees`的表，包含員工的姓名和部門ID，我們可以使用CTE來查詢部門中的所有員工：

```sql
WITH DepartmentCTE AS
(
    SELECT DepartmentID, EmployeeName
    FROM Employees
    WHERE DepartmentID = 1
)
SELECT *
FROM DepartmentCTE;
```

## CTE的應用場景

**簡化複雜查詢** CTE可以用來分解複雜的查詢，使其更易於理解和維護。例如：

```sql
WITH SalesCTE AS
(
    SELECT SalesPerson, SUM(SalesAmount) AS TotalSales
    FROM Sales
    GROUP BY SalesPerson
)
SELECT *
FROM SalesCTE
WHERE TotalSales > 10000;
```

**逐層遞迴** CTE在處理遞迴查詢時非常有用，例如處理組織結構或分層數據：

```
sql
複製程式碼
WITH HierarchyCTE AS
(
    SELECT EmployeeID, ManagerID, EmployeeName
    FROM Employees
    WHERE ManagerID IS NULL
    UNION ALL
    SELECT e.EmployeeID, e.ManagerID, e.EmployeeName
    FROM Employees e
    INNER JOIN HierarchyCTE h ON e.ManagerID = h.EmployeeID
)
SELECT * FROM HierarchyCTE;
```

## 遞迴CTE

**遞迴CTE的語法** 遞迴CTE由兩部分組成：一個初始查詢（基準成員），和一個遞迴成員。遞迴成員引用CTE本身：

```sql
WITH RecursiveCTE AS
(
    -- 基準成員
    SELECT ...
    FROM ...
    WHERE ...
    UNION ALL
    -- 遞迴成員
    SELECT ...
    FROM ...
    INNER JOIN RecursiveCTE ...
)
SELECT * FROM RecursiveCTE;
```

**遞迴CTE範例** 以下是一個處理員工層次結構的範例：

```sql
WITH HierarchyCTE AS
(
    SELECT EmployeeID, ManagerID, EmployeeName
    FROM Employees
    WHERE ManagerID IS NULL
    UNION ALL
    SELECT e.EmployeeID, e.ManagerID, e.EmployeeName
    FROM Employees e
    INNER JOIN HierarchyCTE h ON e.ManagerID = h.EmployeeID
)
SELECT * FROM HierarchyCTE;
```

## CTE的性能考量

**CTE vs 臨時表** CTE是為了可讀性而設計的，但在某些情況下，使用臨時表可能會有更好的性能。需要根據具體情況進行性能測試。

**使用CTE的最佳實踐**

- 避免過度使用CTE，特別是在大數據集上。
- 確保CTE內部的查詢是高效的。
- 在需要重複使用的情況下考慮使用臨時表或物化視圖。

## 結論

- **CTE的優勢** CTE可以顯著提高查詢的可讀性和維護性，特別是在處理遞迴查詢和分層數據時。

- **使用CTE的注意事項** 在使用CTE時，要注意性能問題，並根據具體的需求選擇合適的解決方案。