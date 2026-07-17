# Criteria-based Algorithm Selection

**Status:** *Unvalidated idea — needs feasibility validation.*

**Date:** 2026-07-17

## Idea

Не выбирать алгоритм, не выбирать категорию алгоритма (`Adaptive`,
`MemoryOptimized`, `Parallel`), а выбирать **критерий оптимизации**.
Компилятор (или рантайм) сам подбирает алгоритм, исходя из:

- критериев оптимизации (скорость, память, предсказуемость)
- платформы (CPU, GPU, WebAssembly, bare metal)
- контекста данных (например, данные уже на GPU → GPU sort)

## Два уровня

### Семантический (часть языка)

```orthon
sort(data)              // гарантирует: sortedness, stability
```

Это уже подсказка реализации. Она может отсутствовать на некоторых
платформах.

### Policy (часть Strategy)

```orthon
// ❌ Still names a strategy class
Algorithm Policy = Adaptive

// ✅ Declares optimization intent
//    "minimize memory on this hot path"
//    "maximize throughput, data is on GPU"
```

## Пример: GPU sort

Если сортируемые данные находятся на GPU, компилятор автоматически
выбирает GPU-оптимизированный алгоритм сортировки. Программист не
пишет `GpuSort` — он указывает критерий (или ничего, используя
стратегию по умолчанию).

## Связь с философией Orthon

Это последовательное применение принципа «программист описывает WHAT,
реализация выбирает HOW» — на уровень глубже, чем текущая модель Policy,
где значения всё ещё именуют категории стратегий, а не чистые критерии.

В других местах языка этот принцип уже проводится. Сортировка становится
ещё одним примером.

## Open questions

- Как сформулировать критерии оптимизации декларативно?
- Как разрешать конфликты между критериями (speed vs memory)?
- Может ли программист всё же «зафиксировать» алгоритм когда нужно?
- Как это влияет на предсказуемость производительности?
- Относится ли эта идея только к Algorithm Policy, или шире?

## See also

- `docs/how/IMPLEMENTATION_POLICIES.md` § Algorithm Policy
- `docs/how/architecture/ARCHITECTURE.md` § Implementation Strategy
- `docs/how/strategies/IMPLEMENTATION_STRATEGIES.md`
