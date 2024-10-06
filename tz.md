
```
package main

import (
 "fmt"
 "math"
 "time"
)

// Параметры стратегии
const (
 accountBalance = 10000.0 // Начальный баланс
 riskPerTrade   = 0.02    // Риск на сделку (2%)
 emaShortPeriod = 50      // Период быстрой EMA
 emaLongPeriod  = 200     // Период медленной EMA
 rsiPeriod      = 14      // Период RSI
 atrMultiplier  = 1.5     // Множитель ATR для стоп-лосса
)

// Позиция
type Position struct {
 Direction  string // "long" или "short"
 EntryPrice float64
 StopLoss   float64
 TakeProfit float64
 Size       float64
}

// EMA - экспоненциальная скользящая средняя
func calculateEMA(prices []float64, period int) float64 {
 multiplier := 2.0 / float64(period+1)
 ema := prices[0]
 for i := 1; i < len(prices); i++ {
  ema = ((prices[i] - ema) * multiplier) + ema
 }
 return ema
}

// RSI - индекс относительной силы
func calculateRSI(prices []float64, period int) float64 {
 gainSum, lossSum := 0.0, 0.0
 for i := 1; i <= period; i++ {
  change := prices[i] - prices[i-1]
  if change > 0 {
   gainSum += change
  } else {
   lossSum -= change
  }
 }
 avgGain := gainSum / float64(period)
 avgLoss := lossSum / float64(period)
 rs := avgGain / avgLoss
 rsi := 100 - (100 / (1 + rs))
 return rsi
}

// ATR - средний истинный диапазон (волатильность)
func calculateATR(highs, lows, closes []float64, period int) float64 {
 trSum := 0.0
 for i := 1; i < len(highs); i++ {
  tr := math.Max(highs[i]-lows[i], math.Max(math.Abs(highs[i]-closes[i-1]), math.Abs(lows[i]-closes[i-1])))
  trSum += tr
 }
 return trSum / float64(period)
}

// Вход в позицию
func enterPosition(prices, highs, lows []float64) *Position {
 emaShort := calculateEMA(prices, emaShortPeriod)
 emaLong := calculateEMA(prices, emaLongPeriod)
 rsi := calculateRSI(prices, rsiPeriod)
 atr := calculateATR(highs, lows, prices, rsiPeriod)

 // Проверка на вход в long-позицию
 if emaShort > emaLong && rsi < 30 {
  entryPrice := prices[len(prices)-1]
  stopLoss := entryPrice - atr*atrMultiplier
  takeProfit := entryPrice + atr*atrMultiplier*2
  size := (accountBalance * riskPerTrade) / math.Abs(entryPrice-stopLoss)

  return &Position{
   Direction:  "long",
   EntryPrice: entryPrice,
   StopLoss:   stopLoss,
   TakeProfit: takeProfit,
   Size:       size,
  }
 }
 return nil
}

func main() {
 // Пример данных цен
 prices := []float64{1.0, 1.05, 1.10, 1.20, 1.25, 1.15, 1.17, 1.18, 1.21, 1.25, 1.30, 1.35}
 highs := []float64{1.02, 1.06, 1.12, 1.22, 1.27, 1.18, 1.19, 1.22, 1.24, 1.27, 1.32, 1.37}
 lows := []float64{0.98, 1.03, 1.08, 1.18, 1.23, 1.12, 1.15, 1.16, 1.20, 1.23, 1.28, 1.33}

 // Проверка возможности входа в позицию
 position := enterPosition(prices, highs, lows)
 if position != nil {
  fmt.Printf("Открыта позиция: %v, Размер: %.2f, Вход: %.2f, Стоп-лосс: %.2f, Тейк-профит: %.2f\n",
   position.Direction, position.Size, position.EntryPrice, position.StopLoss, position.TakeProfit)
 } else {
  fmt.Println("Нет сигнала для входа в позицию.")
 }

 // Подождите обновления данных
 time.Sleep(1 * time.Second)
}

```

Что делает этот код:

* EMA (Exponential Moving Average): Рассчитываются две скользящие средние с разными периодами для определения тренда (короткая и длинная EMA).
* RSI (Relative Strength Index): Определяет состояние перекупленности или перепроданности рынка для подтверждения входа.
* ATR (Average True Range): Используется для определения волатильности и расчёта уровней стоп-лосса и тейк-профита.

Вход в позицию: Робот открывает позицию, если EMA и RSI сигнализируют о благоприятном тренде.

Риск-менеджмент: Размер позиции рассчитывается на основе установленного процента риска и разницы между входной ценой и стоп-лоссом.

Этот пример является базовой реализацией, которую можно расширять в зависимости от специфики торговли и целей стратегии.

Лучший подход:

Для начала можно протестировать стратегию на средних таймфреймах (например, 15 минут или 1 час), поскольку они обеспечивают баланс между количеством сигналов и качеством трендов. После оптимизации стратегии можно попробовать её на более коротких или более длинных таймфреймах, чтобы понять, где она работает лучше всего.

```
package main

import (
 "fmt"
 "math"
 "time"
)

// Параметры стратегии
const (
 accountBalance = 10000.0 // Начальный баланс
 riskPerTrade   = 0.02    // Риск на сделку (2%)
 emaShortPeriod = 50      // Период быстрой EMA
 emaLongPeriod  = 200     // Период медленной EMA
 rsiPeriod      = 14      // Период RSI
 atrMultiplier  = 1.5     // Множитель ATR для стоп-лосса
 imbalanceFactor = 2.0    // Фактор для определения зоны дисбаланса
)

// Позиция
type Position struct {
 Direction  string // "long" или "short"
 EntryPrice float64
 StopLoss   float64
 TakeProfit float64
 Size       float64
}

// EMA - экспоненциальная скользящая средняя
func calculateEMA(prices []float64, period int) float64 {
 multiplier := 2.0 / float64(period+1)
 ema := prices[0]
 for i := 1; i < len(prices); i++ {
  ema = ((prices[i] - ema) * multiplier) + ema
 }
 return ema
}

// RSI - индекс относительной силы
func calculateRSI(prices []float64, period int) float64 {
 gainSum, lossSum := 0.0, 0.0
 for i := 1; i <= period; i++ {
  change := prices[i] - prices[i-1]
  if change > 0 {
   gainSum += change
  } else {
   lossSum -= change
  }
 }
 avgGain := gainSum / float64(period)
 avgLoss := lossSum / float64(period)
 rs := avgGain / avgLoss
 rsi := 100 - (100 / (1 + rs))
 return rsi
}

// ATR - средний истинный диапазон (волатильность)
func calculateATR(highs, lows, closes []float64, period int) float64 {
 trSum := 0.0
 for i := 1; i < len(highs); i++ {
  tr := math.Max(highs[i]-lows[i], math.Max(math.Abs(highs[i]-closes[i-1]), math.Abs(lows[i]-closes[i-1])))
  trSum += tr
 }
 return trSum / float64(period)
}

// Определение зоны дисбаланса
func findImbalanceZone(highs, lows []float64) (bool, int) {
 for i := 1; i < len(highs); i++ {
  rangeMove := math.Abs(highs[i] - lows[i])
  prevRangeMove := math.Abs(highs[i-1] - lows[i-1])

  // Проверка на наличие дисбаланса в движении цены
  if rangeMove > prevRangeMove*imbalanceFactor {
   return true, i
  }
 }
 return false, -1
}

// Вход в позицию
func enterPosition(prices, highs, lows []float64) *Position {
 emaShort := calculateEMA(prices, emaShortPeriod)
 emaLong := calculateEMA(prices, emaLongPeriod)
 rsi := calculateRSI(prices, rsiPeriod)
 atr := calculateATR(highs, lows, prices, rsiPeriod)

 // Проверка на зону дисбаланса
 isImbalance, index := findImbalanceZone(highs, lows)
 if !isImbalance {
  return nil
 }

 // Проверка на вход в long-позицию в зоне дисбаланса
 if emaShort > emaLong && rsi < 30 && index >= 0 {
  entryPrice := prices[index]
  stopLoss := entryPrice - atr*atrMultiplier
  takeProfit := entryPrice + atr*atrMultiplier*2
  size := (accountBalance * riskPerTrade) / math.Abs(entryPrice-stopLoss)

  return &Position{
   Direction:  "long",
   EntryPrice: entryPrice,
   StopLoss:   stopLoss,
   TakeProfit: takeProfit,
   Size:       size,
  }
 }
 return nil
}

func main() {
 // Пример данных цен
 prices := []float64{1.0, 1.05, 1.10, 1.20, 1.25, 1.15, 1.17, 1.18, 1.21, 1.25, 1.30, 1.35}
 highs := []float64{1.02, 1.06, 1.12, 1.22, 1.27, 1.18, 1.19, 1.22, 1.24, 1.27, 1.32, 1.37}
 lows := []float64{0.98, 1.03, 1.08, 1.18, 1.23, 1.12, 1.15, 1.16, 1.20, 1.23, 1.28, 1.33}

 // Проверка возможности входа в позицию
 position := enterPosition(prices, highs, lows)
 if position != nil {
  fmt.Printf("Открыта позиция: %v, Размер: %.2f, Вход: %.2f, Стоп-лосс: %.2f, Тейк-профит: %.2f\n",
   position.Direction, position.Size, position.EntryPrice, position.StopLoss, position.TakeProfit)
 } else {
  fmt.Println("Нет сигнала для входа в позицию.")
 }

 // Подождите обновления данных
 time.Sleep(1 * time.Second)
}

```

Что делает этот обновлённый код:

Определяет зоны дисбаланса: Проверяет, была ли значительная разница в движении цены между предыдущими свечами, чтобы выявить зоны дисбаланса.

Использует концепцию смарт-мани: Ищет точки входа, которые соответствуют движению "умных денег" — областям, где крупные игроки могут войти или выйти из рынка.

Проверяет условия тренда: Использует индикаторы EMA и RSI для подтверждения направления тренда и вероятного разворота.

Этот код — более продвинутая версия базовой стратегии и может использоваться для торговли на основе концепции дисбаланса и анализа смарт-мани.

```
package main

import (
 "fmt"
 "math"
 "time"
)

// Таймфреймы и параметры для каждого из них
var timeframes = []struct {
 Name         string
 EmaShort     int     // Период быстрой EMA
 EmaLong      int     // Период медленной EMA
 RsiPeriod    int     // Период RSI
 atrMultiplier float64 // Множитель ATR для стоп-лосса
 GridSize     int     // Размер сетки
 GridStep     float64 // Шаг сетки
}{
 {"15m", 20, 50, 14, 1.5, 5, 0.005},
 {"1h", 50, 200, 14, 1.5, 5, 0.01},
 {"4h", 50, 200, 14, 1.5, 5, 0.015},
 {"1d", 100, 200, 14, 1.5, 5, 0.02},
 {"1w", 100, 200, 14, 1.5, 5, 0.025},
}

// Параметры стратегии
const (
 accountBalance = 10000.0 // Общий баланс
 riskPerTrade   = 0.02    // Риск на сделку (2%)
 imbalanceFactor = 2.0     // Фактор для определения зоны дисбаланса
)

// Позиция
type Position struct {
 Timeframe  string  // Таймфрейм
 Direction  string  // "long" или "short"
 EntryPrice float64
 StopLoss   float64
 TakeProfit float64
 Size       float64
}

// EMA - экспоненциальная скользящая средняя
func calculateEMA(prices []float64, period int) float64 {
 multiplier := 2.0 / float64(period+1)
 ema := prices[0]
 for i := 1; i < len(prices); i++ {
  ema = ((prices[i] - ema) * multiplier) + ema
 }
 return ema
}

// RSI - индекс относительной силы
func calculateRSI(prices []float64, period int) float64 {
 gainSum, lossSum := 0.0, 0.0
 for i := 1; i <= period; i++ {
  change := prices[i] - prices[i-1]
  if change > 0 {
   gainSum += change
  } else {
   lossSum -= change
  }
 }
 avgGain := gainSum / float64(period)
 avgLoss := lossSum / float64(period)
 rs := avgGain / avgLoss
 rsi := 100 - (100 / (1 + rs))
 return rsi
}

// ATR - средний истинный диапазон (волатильность)
func calculateATR(highs, lows, closes []float64, period int) float64 {
 trSum := 0.0
 for i := 1; i < len(highs); i++ {
  tr := math.Max(highs[i]-lows[i], math.Max(math.Abs(highs[i]-closes[i-1]), math.Abs(lows[i]-closes[i-1])))
  trSum += tr
 }
 return trSum / float64(period)
}

// Определение зоны дисбаланса
func findImbalanceZone(highs, lows []float64) (bool, int) {
 for i := 1; i < len(highs); i++ {
  rangeMove := math.Abs(highs[i] - lows[i])
  prevRangeMove := math.Abs(highs[i-1] - lows[i-1])

  // Проверка на наличие дисбаланса в движении цены
  if rangeMove > prevRangeMove*imbalanceFactor {
   return true, i
  }
 }
 return false, -1
}

// Создание сетки ордеров
func createGrid(entryPrice float64, gridSize int, gridStep float64) []float64 {
 gridLevels := make([]float64, gridSize)
 step := entryPrice * gridStep

 for i := 0; i < gridSize; i++ {
  gridLevels[i] = entryPrice - step*float64(i)
 }

 return gridLevels
}

// Вход в позицию и создание сетки
func enterPosition(prices, highs, lows []float64, timeframeConfig struct {
 Name         string
 EmaShort     int
 EmaLong      int
 RsiPeriod    int
 atrMultiplier float64
 GridSize     int
 GridStep     float64
}, allocatedBalance float64) []*Position {
 emaShort := calculateEMA(prices, timeframeConfig.EmaShort)
 emaLong := calculateEMA(prices, timeframeConfig.EmaLong)
 rsi := calculateRSI(prices, timeframeConfig.RsiPeriod)
 atr := calculateATR(highs, lows, prices, timeframeConfig.RsiPeriod)

 // Проверка на зону дисбаланса
 isImbalance, index := findImbalanceZone(highs, lows)
 if !isImbalance {
  return nil
 }

 // Проверка на вход в long-позицию в зоне дисбаланса
 if emaShort > emaLong && rsi < 30 && index >= 0 {
  entryPrice := prices[index]
  gridLevels := createGrid(entryPrice, timeframeConfig.GridSize, timeframeConfig.GridStep)
  positions := []*Position{}

  // Создание позиций на каждом уровне сетки
  for _, level := range gridLevels {
   stopLoss := level - atr*timeframeConfig.atrMultiplier
   takeProfit := level + atr*timeframeConfig.atrMultiplier*2
   size := (allocatedBalance * riskPerTrade) / math.Abs(level-stopLoss)

   position := &Position{
    Timeframe:  timeframeConfig.Name,
    Direction:  "long",
    EntryPrice: level,
    StopLoss:   stopLoss,
    TakeProfit: takeProfit,
    Size:       size,
   }

   positions = append(positions, position)
  }

return positions
 }
 return nil
}

func main() {
 // Пример данных цен
 prices := []float64{1.0, 1.05, 1.10, 1.20, 1.25, 1.15, 1.17, 1.18, 1.21, 1.25, 1.30, 1.35}
 highs := []float64{1.02, 1.06, 1.12, 1.22, 1.27, 1.18, 1.19, 1.22, 1.24, 1.27, 1.32, 1.37}
 lows := []float64{0.98, 1.03, 1.08, 1.18, 1.23, 1.12, 1.15, 1.16, 1.20, 1.23, 1.28, 1.33}

 // Распределение баланса на 5 таймфреймов
 allocatedBalance := accountBalance / float64(len(timeframes))

 // Проверка возможности входа в позицию для каждого таймфрейма
 for _, tf := range timeframes {
  positions := enterPosition(prices, highs, lows, tf, allocatedBalance)
  if positions != nil {
   for _, position := range positions {
    fmt.Printf("Таймфрейм: %s, Открыта позиция: %v, Размер: %.2f, Вход: %.2f, Стоп-лосс: %.2f, Тейк-профит: %.2f\n",
     position.Timeframe, position.Direction, position.Size, position.EntryPrice, position.StopLoss, position.TakeProfit)
   }
  } else {
   fmt.Printf("Таймфрейм: %s, Нет сигнала для входа в позицию.\n", tf.Name)
  }
 }

 // Подождите обновления данных
 time.Sleep(1 * time.Second)
}
```

Что делает этот код:

5 разных таймфреймов: Определены параметры для каждого таймфрейма — 15 минут, 1 час, 4 часа, 1 день и 1 неделя. У каждого таймфрейма свои настройки для EMA, RSI и сетки.

Распределение баланса: Общий баланс разделён на 5 частей, по одной на каждый таймфрейм.

Сетка ордеров: На каждом таймфрейме создаётся своя сетка ордеров, чтобы извлекать прибыль из колебаний цены.

Отдельные сигналы и позиции: Для каждого таймфрейма отдельно анализируются цены и открываются позиции на основе индивидуальных настроек.

Этот подход позволяет диверсифицировать стратегии на разных таймфреймах

```
package main

import (
 "fmt"
 "math"
 "time"
)

// Таймфреймы и параметры для каждого из них
var timeframes = []struct {
 Name         string
 EmaShort     int     // Период быстрой EMA
 EmaLong      int     // Период медленной EMA
 RsiPeriod    int     // Период RSI
 atrMultiplier float64 // Множитель ATR для стоп-лосса
 GridSize     int     // Размер сетки
 GridStep     float64 // Шаг сетки
}{
 {"15m", 20, 50, 14, 1.5, 3, 0.01},
 {"1h", 50, 200, 14, 1.5, 3, 0.02},
 {"4h", 50, 200, 14, 1.5, 3, 0.03},
 {"1d", 100, 200, 14, 1.5, 3, 0.04},
 {"1w", 100, 200, 14, 1.5, 3, 0.05},
}

// Параметры стратегии
const (
 accountBalance = 10000.0 // Общий баланс
 riskPerTrade   = 0.05    // Риск на сделку (5%)
 imbalanceFactor = 2.0     // Фактор для определения зоны дисбаланса
)

// Позиция
type Position struct {
 Timeframe  string  // Таймфрейм
 Direction  string  // "long"
 EntryPrice float64
 StopLoss   float64
 TakeProfit float64
 Size       float64
}

// EMA - экспоненциальная скользящая средняя
func calculateEMA(prices []float64, period int) float64 {
 multiplier := 2.0 / float64(period+1)
 ema := prices[0]
 for i := 1; i < len(prices); i++ {
  ema = ((prices[i] - ema) * multiplier) + ema
 }
 return ema
}

// RSI - индекс относительной силы
func calculateRSI(prices []float64, period int) float64 {
 gainSum, lossSum := 0.0, 0.0
 for i := 1; i <= period; i++ {
  change := prices[i] - prices[i-1]
  if change > 0 {
   gainSum += change
  } else {
   lossSum -= change
  }
 }
 avgGain := gainSum / float64(period)
 avgLoss := lossSum / float64(period)
 rs := avgGain / avgLoss
 rsi := 100 - (100 / (1 + rs))
 return rsi
}

// ATR - средний истинный диапазон (волатильность)
func calculateATR(highs, lows, closes []float64, period int) float64 {
 trSum := 0.0
 for i := 1; i < len(highs); i++ {
  tr := math.Max(highs[i]-lows[i], math.Max(math.Abs(highs[i]-closes[i-1]), math.Abs(lows[i]-closes[i-1])))
  trSum += tr
 }
 return trSum / float64(period)
}

// Определение зоны дисбаланса
func findImbalanceZone(highs, lows []float64) (bool, int) {
 for i := 1; i < len(highs); i++ {
  rangeMove := math.Abs(highs[i] - lows[i])
  prevRangeMove := math.Abs(highs[i-1] - lows[i-1])

  // Проверка на наличие дисбаланса в движении цены
  if rangeMove > prevRangeMove*imbalanceFactor {
   return true, i
  }
 }
 return false, -1
}

// Создание сетки ордеров
func createGrid(entryPrice float64, gridSize int, gridStep float64) []float64 {
 gridLevels := make([]float64, gridSize)
 step := entryPrice * gridStep

 for i := 0; i < gridSize; i++ {
  gridLevels[i] = entryPrice - step*float64(i)
 }

 return gridLevels
}

// Вход в позицию и создание сетки
func enterPosition(prices, highs, lows []float64, timeframeConfig struct {
 Name         string
 EmaShort     int
 EmaLong      int
 RsiPeriod    int
 atrMultiplier float64
 GridSize     int
 GridStep     float64
}, allocatedBalance float64) []*Position {
 emaShort := calculateEMA(prices, timeframeConfig.EmaShort)
 emaLong := calculateEMA(prices, timeframeConfig.EmaLong)
 rsi := calculateRSI(prices, timeframeConfig.RsiPeriod)
 atr := calculateATR(highs, lows, prices, timeframeConfig.RsiPeriod)

 // Проверка на зону дисбаланса
 isImbalance, index := findImbalanceZone(highs, lows)
 if !isImbalance {
  return nil
 }

 // Проверка на вход в long-позицию в зоне дисбаланса (только long для спота)
 if emaShort > emaLong && rsi < 30 && index >= 0 {
  entryPrice := prices[index]
  gridLevels := createGrid(entryPrice, timeframeConfig.GridSize, timeframeConfig.GridStep)
  positions := []*Position{}

  // Создание позиций на каждом уровне сетки
  for _, level := range gridLevels {
   stopLoss := level - atr*timeframeConfig.atrMultiplier
   takeProfit := level + atr*timeframeConfig.atrMultiplier*2
   size := (allocatedBalance * riskPerTrade) / math.Abs(level-stopLoss)

   position := &Position{
    Timeframe:  timeframeConfig.Name,
    Direction:  "long",
    EntryPrice: level,
    StopLoss:   stopLoss,
    TakeProfit: takeProfit,
    Size:       size,
   }

   positions = append(positions, position)
  }

return positions
 }
 return nil
}

func main() {
 // Пример данных цен
 prices := []float64{1.0, 1.05, 1.10, 1.20, 1.25, 1.15, 1.17, 1.18, 1.21, 1.25, 1.30, 1.35}
 highs := []float64{1.02, 1.06, 1.12, 1.22, 1.27, 1.18, 1.19, 1.22, 1.24, 1.27, 1.32, 1.37}
 lows := []float64{0.98, 1.03, 1.08, 1.18, 1.23, 1.12, 1.15, 1.16, 1.20, 1.23, 1.28, 1.33}

 // Распределение баланса на 5 таймфреймов
 allocatedBalance := accountBalance / float64(len(timeframes))

 // Проверка возможности входа в позицию для каждого таймфрейма
 for _, tf := range timeframes {
  positions := enterPosition(prices, highs, lows, tf, allocatedBalance)
  if positions != nil {
   for _, position := range positions {
    fmt.Printf("Таймфрейм: %s, Открыта позиция: %v, Размер: %.2f, Вход: %.2f, Стоп-лосс: %.2f, Тейк-профит: %.2f\n",
     position.Timeframe, position.Direction, position.Size, position.EntryPrice, position.StopLoss, position.TakeProfit)
   }
  } else {
   fmt.Printf("Таймфрейм: %s, Нет сигнала
```

Этот код реализует стратегию спотовой торговли, которая включает:

1. Мульти-Таймфрейм Анализ:
Работает с пятью разными таймфреймами: 15 минут, 1 час, 4 часа, 1 день и 1 неделя.
Для каждого таймфрейма задаются индивидуальные параметры: короткая и длинная скользящие средние (EMA), индекс относительной силы (RSI), множитель для расчета ATR (средний истинный диапазон) и настройки сетки ордеров.
Баланс разделён на пять частей, чтобы каждый таймфрейм мог управлять своей долей капитала.
2. Стратегия сетки (Grid Trading):
Создаются позиции на покупку (long) в виде сетки, распределённой на нескольких уровнях цены. Это помогает зарабатывать на колебаниях рынка в рамках зоны дисбаланса.
Шаг сетки и количество уровней регулируются индивидуально для каждого таймфрейма.
3. Использование индикаторов:
EMA (Exponential Moving Average): Две скользящие средние (короткая и длинная) определяют направление тренда. Если короткая EMA выше длинной, это сигнал к покупке.
RSI (Relative Strength Index): Определяет перекупленность или перепроданность рынка. Если RSI меньше 30, рынок считается перепроданным, и это сигнал к возможной покупке.
ATR (Average True Range): Используется для определения волатильности и установки уровней стоп-лосса.
4. Определение зон дисбаланса (Imbalance Zones):
Проверяет резкие движения цены на предыдущих свечах для выявления зоны дисбаланса, что может быть сигналом для потенциального входа в позицию.
5. Вход в позицию:
Если одновременно выполняются условия для тренда (EMA), RSI, и цена находится в зоне дисбаланса, открываются позиции на покупку.
Создаётся сетка ордеров на основе уровней цены с рассчитанными стоп-лоссами и тейк-профитами.
6. Вывод данных:
После проверки условий входа по каждому таймфрейму, выводится информация об открытых позициях (размер позиции, входная цена, стоп-лосс, тейк-профит).
В чём особенности для спотовой торговли?
Только позиции на покупку (long), так как в спотовой торговле обычно отсутствует возможность продажи в короткую (short).
Размер позиций и риск на сделку адаптированы под спотовую торговлю, чтобы учитывать отсутствие кредитного плеча и работать на реальном капитале.

Пример:

При запуске кода проверяется каждый из пяти таймфреймов на возможность входа в позицию. Если условия выполняются, открываются "сетки" позиций на покупку, и результат отображается в консоли.

```
package main

import (
 "fmt"
 "math"
 "time"
)

// Таймфреймы и параметры для скальпинга
var timeframes = []struct {
 Name         string
 SmaShort     int     // Период быстрой SMA
 SmaLong      int     // Период медленной SMA
 RsiPeriod    int     // Период RSI
 atrMultiplier float64 // Множитель ATR для стоп-лосса
 GridSize     int     // Размер сетки
 GridStep     float64 // Шаг сетки
}{
 {"1m", 5, 20, 14, 0.5, 2, 0.002},
 {"5m", 10, 30, 14, 0.5, 2, 0.004},
 {"15m", 20, 50, 14, 1.0, 2, 0.006},
}

// Параметры стратегии
const (
 accountBalance = 10000.0 // Общий баланс
 riskPerTrade   = 0.02    // Риск на сделку (2%)
 imbalanceFactor = 2.0     // Фактор для определения зоны дисбаланса
)

// Позиция
type Position struct {
 Timeframe  string  // Таймфрейм
 Direction  string  // "long"
 EntryPrice float64
 StopLoss   float64
 TakeProfit float64
 Size       float64
}

// SMA - простая скользящая средняя
func calculateSMA(prices []float64, period int) float64 {
 sum := 0.0
 for i := 0; i < period; i++ {
  sum += prices[len(prices)-1-i]
 }
 return sum / float64(period)
}

// RSI - индекс относительной силы
func calculateRSI(prices []float64, period int) float64 {
 gainSum, lossSum := 0.0, 0.0
 for i := 1; i <= period; i++ {
  change := prices[i] - prices[i-1]
  if change > 0 {
   gainSum += change
  } else {
   lossSum -= change
  }
 }
 avgGain := gainSum / float64(period)
 avgLoss := lossSum / float64(period)
 rs := avgGain / avgLoss
 rsi := 100 - (100 / (1 + rs))
 return rsi
}

// ATR - средний истинный диапазон (волатильность)
func calculateATR(highs, lows, closes []float64, period int) float64 {
 trSum := 0.0
 for i := 1; i < len(highs); i++ {
  tr := math.Max(highs[i]-lows[i], math.Max(math.Abs(highs[i]-closes[i-1]), math.Abs(lows[i]-closes[i-1])))
  trSum += tr
 }
 return trSum / float64(period)
}

// Определение зоны дисбаланса
func findImbalanceZone(highs, lows []float64) (bool, int) {
 for i := 1; i < len(highs); i++ {
  rangeMove := math.Abs(highs[i] - lows[i])
  prevRangeMove := math.Abs(highs[i-1] - lows[i-1])

  // Проверка на наличие дисбаланса в движении цены
  if rangeMove > prevRangeMove*imbalanceFactor {
   return true, i
  }
 }
 return false, -1
}

// Создание сетки ордеров
func createGrid(entryPrice float64, gridSize int, gridStep float64) []float64 {
 gridLevels := make([]float64, gridSize)
 step := entryPrice * gridStep

 for i := 0; i < gridSize; i++ {
  gridLevels[i] = entryPrice - step*float64(i)
 }

 return gridLevels
}

// Вход в позицию и создание сетки
func enterPosition(prices, highs, lows []float64, timeframeConfig struct {
 Name         string
 SmaShort     int
 SmaLong      int
 RsiPeriod    int
 atrMultiplier float64
 GridSize     int
 GridStep     float64
}, allocatedBalance float64) []*Position {
 smaShort := calculateSMA(prices, timeframeConfig.SmaShort)
 smaLong := calculateSMA(prices, timeframeConfig.SmaLong)
 rsi := calculateRSI(prices, timeframeConfig.RsiPeriod)
 atr := calculateATR(highs, lows, prices, timeframeConfig.RsiPeriod)

 // Проверка на зону дисбаланса
 isImbalance, index := findImbalanceZone(highs, lows)
 if !isImbalance {
  return nil
 }

 // Проверка на вход в long-позицию в зоне дисбаланса (только long для спота)
 if smaShort > smaLong && rsi < 30 && index >= 0 {
  entryPrice := prices[index]
  gridLevels := createGrid(entryPrice, timeframeConfig.GridSize, timeframeConfig.GridStep)
  positions := []*Position{}

  // Создание позиций на каждом уровне сетки
  for _, level := range gridLevels {
   stopLoss := level - atr*timeframeConfig.atrMultiplier
   takeProfit := level + atr*timeframeConfig.atrMultiplier*2
   size := (allocatedBalance * riskPerTrade) / math.Abs(level-stopLoss)

   position := &Position{
    Timeframe:  timeframeConfig.Name,
    Direction:  "long",
    EntryPrice: level,
    StopLoss:   stopLoss,
    TakeProfit: takeProfit,
    Size:       size,
   }

   positions = append(positions, position)
  }

  return positions
 }
 return nil
}

func main() {
 // Пример данных цен
 prices := []float64{1.0, 1.05, 1.10, 1.20, 1.25, 1.15, 1.17, 1.18, 1.21, 1.25, 1.30, 1.35}
 highs := []float64{1.02, 1.06, 1.12, 1.22, 1.27, 1.18, 1.19, 1.22, 1.24, 1.27, 1.32, 1.37}
 lows := []float64{0.98, 1.03, 1.08, 1.18, 1.23, 1.12, 1.15, 1.16, 1.20, 1.23, 1.28, 1.33}

 // Распределение баланса на 3 таймфрейма (скальпинг)
 allocatedBalance := accountBalance / float64(len(timeframes))

 // Проверка возможности входа в позицию для каждого таймфрейма
 for _, tf := range timeframes {
  positions := enterPosition(prices, highs, lows, tf, allocatedBalance)
  if positions != nil {
   for _, position := range positions {
    fmt.Printf("Таймфрейм: %s, Открыта позиция: %v, Размер: %.2f, Вход: %.2f, Стоп-лосс: %.2f, Тейк-профит: %.2f\n",
     position.Timeframe, position.Direction, position.Size, position.EntryPrice, position.StopLoss, position.TakeProfit)
   }
  } else {
   fmt.Printf("Таймфрейм: %s, Нет сигнала для входа в позицию.\n", tf.Name)
  }
 }

 // Подождите обновления данных
 time.Sleep(1 * time.Second)
}
```