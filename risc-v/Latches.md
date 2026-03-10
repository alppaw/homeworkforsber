
# 🔐 Защелки (Latches) в Цифровой Схемотехнике

## 📑 Оглавление
1.[Что такое защелка?](#1-что-такое-защелка)
2.[SR-защелка (Асинхронная)](#2-sr-защелка-асинхронная)
3.[Стробируемая SR-защелка](#3-стробируемая-sr-защелка-gated-sr-latch)
4. [D-защелка (Прозрачная)](#4-d-защелка-прозрачная-transparent-latch)
5. [Лучшие практики в SystemVerilog](#5-лучшие-практики-в-systemverilog)

---

## 1. Что такое защелка?

**Защелка (Latch)** — это простейший бистабильный элемент памяти, чувствительный к **уровню сигнала** (level-sensitive). 

**Главное отличие от триггера (Flip-Flop):**
*   **Защелка** пропускает данные на выход все время, пока разрешающий сигнал (Enable) находится в активном уровне (например, в `1`). Это свойство называется *прозрачностью*.
*   **Триггер** захватывает данные только в одно мгновение — по фронту (edge-triggered) тактового сигнала (Clock).

---

## 2. SR-защелка (Асинхронная)

Самый базовый тип памяти. Состоит из двух перекрестно-связанных логических элементов ИЛИ-НЕ (NOR) или И-НЕ (NAND). У нее есть два входа: `S` (Set — установка в 1) и `R` (Reset — сброс в 0).

В чистом виде асинхронные SR-защелки описываются через непрерывное присваивание с обратной связью.

```systemverilog
module sr_latch (
    input  logic S, // Вход установки
    input  logic R, // Вход сброса
    output logic Q, // Прямой выход
    output logic Qn // Инверсный выход
);
    // Реализация на вентилях NOR (ИЛИ-НЕ)
    assign Q  = ~(R | Qn);
    assign Qn = ~(S | Q);  
endmodule
```
<p align="center">
  <img src="https://github.com/alppaw/homeworkforsber/blob/main/im/rs.png" alt="SR-защелка"><br>
  <em>Схема SR-защелки</em>
</p>
---
## 3. Стробируемая SR-защелка (Gated SR Latch)
Чтобы управлять моментом времени, когда SR-защелка реагирует на входы, добавляется разрешающий сигнал E (Enable). Сигналы S и R влияют на состояние только тогда, когда E = 1.

```systemverilog
module gated_sr_latch (
    input  logic E, // Разрешающий сигнал (Enable)
    input  logic S,
    input  logic R,
    output logic Q,
    output logic Qn
);
    logic s_gated, r_gated;

    // Сигналы доходят до защелки только при E == 1
    assign s_gated = S & E;
    assign r_gated = R & E;

    assign Q  = ~(r_gated | Qn);
    assign Qn = ~(s_gated | Q);
endmodule
```

<p align="center">
  <img src="https://github.com/alppaw/homeworkforsber/blob/main/im/rse.png" alt="SRE-защелка"><br>
  <em>Схема SRE-защелки</em>
</p>

---
## 4. D-защелка (Прозрачная / Transparent Latch)
Это самый популярный тип защелки в цифровом дизайне. Она решает главную проблему SR-защелки — запрещенное состояние (S=1, R=1).
Вход данных D (Data) передается на S, а его инверсия — на R. Таким образом, S и R никогда не бывают равны 1 одновременно.
Когда E = 1, защелка "прозрачна": выход Q повторяет вход D.
Когда E = 0, защелка защелкивает (сохраняет) последнее значение.
Для описания D-защелок в SystemVerilog существует специальный блок always_latch. Он сигнализирует синтезатору о том, что вы намеренно создаете защелку.
code

```systemverilog
module d_latch (
    input  logic E, // Enable
    input  logic D, // Data
    output logic Q  // Output
);

    // always_latch подсказывает EDA инструментам ваши намерения
    always_latch begin
        if (E) begin
            Q <= D; 
        end
        // Неявный else: если E == 0, Q сохраняет текущее значение
    end
endmodule
```

<p align="center">
  <img src="https://github.com/alppaw/homeworkforsber/blob/main/im/d.png" alt="D-защелка"><br>
  <em>Схема D-защелки</em>
</p>
---
## 5. Лучшие практики в SystemVerilog
Используйте always_latch: Никогда не используйте always_comb или always @* для создания защелок. always_latch заставит компилятор/синтезатор выдать предупреждение, если ваша логика не образует защелку.
Избегайте непреднамеренных защелок: В комбинаторных блоках (always_comb) всегда прописывайте ветку else для if и ветку default для case.


```systemverilog
// Создаст непреднамеренную защелку
always_comb begin
    if (sel) out = A;
    // Что будет если sel == 0? Синтезатор создаст защелку для out!
end

// Чистая комбинаторная логика
always_comb begin
    if (sel) out = A;
    else     out = 1'b0;
end
```