# Description of ComfyUI-zveroboy-photo Project

## Overall Purpose

**ComfyUI-zveroboy-photo** is a custom node suite for ComfyUI designed for **post-processing images to give them characteristics of real photographs**.

This node suite is a **research tool** for exploring the boundaries of AI image detection, not for deception. Its use should be ethical and responsible.

![comfy-photo](https://github.com/thezveroboy/ComfyUI-zveroboy-photo/blob/main/img1.jpg)

![comfy-photo](https://github.com/thezveroboy/ComfyUI-zveroboy-photo/blob/main/img2.jpg)

### Main Objectives:

1. **Researching AI image detector vulnerabilities** - demonstrating that simple signatures (noise, EXIF) are insufficient for reliable detection
2. **Imitating real cameras** - adding realistic sensor characteristics, noise, and metadata
3. **Protecting authorship** - complicating automatic image classification
4. **Enhancing photorealism** - adding aesthetic grain and analog artifacts

### Ethical Position:
> The goal is NOT to deceive detectors, but to demonstrate their imperfection and stimulate the development of more advanced algorithms based on semantic analysis rather than superficial features.

---

## Node Descriptions

### 1. PhotoAddNoise (Basic)
**Purpose:** Adding Gaussian noise for AI detector protection

**Parameters:**
- `noise_level` (0.0-1.0) - noise intensity

**Features:**
- Reduces noise in blue channel (x0.8) to imitate real sensors
- Uniform white noise across all channels

**When to use:**
- Quick basic protection
- Minimal quality impact
- Values 0.01-0.02 for subtlety

---

### 2. PhotoAddGrain (Aesthetic)
**Purpose:** Adding film grain for visual aesthetics

**Parameters:**
- `grain_strength` - grain intensity
- `grain_size` - grain size (1-10)

**Features:**
- Monochrome grain (Luminance)
- Overlay blending algorithm
- Grain scaling to imitate different films

**When to use:**
- Artistic processing
- Analog photography imitation
- Not for protection (too noticeable)

---

### 3. PhotoAddExif
**Purpose:** Embedding camera metadata (EXIF) to imitate real shots

**Parameters:**
- `preset` - camera selection (Canon/Sony/Nikon/Fujifilm/Leica/iPhone)
- `artist`, `software`, `copyright` - author data
- `body_serial`, `lens_serial` - serial numbers (auto-generated)
- `focal_length_mm`, `fnumber`, `exposure_1_over_s`, `iso` - shooting parameters
- `datetime_original` - shooting date

**Features:**
- Returns EXIF as Base64 string for inter-node transfer
- Uses presets from `presets.py`
- Automatic random serial generation

**When to use:**
- Always before saving JPG
- For imitating specific cameras
- Combined with PhotoSaveJpg

---

### 4. PhotoLoadRaw
**Purpose:** Loading RAW and HEIC images with EXIF extraction

**Parameters:**
- `raw_file` - full file path

**Supported formats:**
- **HEIC** (iPhone via pillow-heif)
- **DNG** (Apple ProRAW, cameras)
- **NEF** (Nikon)
- **ARW** (Sony)
- **CR2/CR3** (Canon)

**Features:**
- Two-stage loading: Pillow first, then rawpy
- Extraction of existing EXIF data
- Conversion to ComfyUI RGB tensor

**When to use:**
- Working with camera/iPhone originals
- Preserving maximum quality
- Extracting metadata for later modification

---

### 5. PhotoSaveJpg
**Purpose:** Saving to JPEG with embedded EXIF data

**Parameters:**
- `exif_data` - Base64 EXIF string (from PhotoAddExif)
- `filename_prefix` - filename prefix
- `quality` (1-100) - JPEG quality

**Features:**
- Saves to ComfyUI output_directory
- Timestamp in filename
- Huffman coding optimization

**When to use:**
- Final saving
- For publication/sharing
- When EXIF metadata is needed

---

### 6. PhotoSaveRaw
**Purpose:** Saving to lossless formats (TIFF/DNG) with color matrices

**Parameters:**
- `format` - TIFF or DNG
- `preset` - camera selection for matrices
- `exif_data` - metadata

**Features:**
- **DNG**: writes ColorMatrix1/2, AsShotNeutral, CalibrationIlluminant
- **TIFF**: standard lossless with EXIF
- Uses tifffile for correct DNG structure
- Preserves all data without loss

**When to use:**
- Maximum quality archiving
- Transfer to professional editors
- Hiding processing traces (TIFF looks like RAW)

---

### 7. PhotoAdvancedNoise (Sensor Simulation)
**Purpose:** Advanced camera sensor noise imitation for bypassing detectors

**Parameters:**
- `noise_strength` (0.0-0.5) - overall intensity
- `color_correlation` - Bayer Pattern imitation (different noise per channel)
- `add_grain_layer` - additional grain layer
- `grain_size` - grain size
- `jpeg_compression` - final compression

**Features:**
- **Shot Noise**: depends on pixel brightness `(arr ** 0.5)`
- **Read Noise**: constant component
- **Color Correlation**: R x1.2, G x0.9, B x1.4 (like real sensors)
- **JPEG artifacts**: masking AI smoothness

**When to use:**
- **Instead of PhotoAddNoise** for better protection
- Values 0.005-0.008 for subtlety
- Disable `add_grain_layer` to avoid quality loss
- `jpeg_compression` 95-98

---

### 8. Realism7NoiseExif (All-in-One)
**Purpose:** Comprehensive "all-in-one" processing for quick results

**Parameters:**
- All EXIF parameters (like PhotoAddExif)
- `noise_level` - noise
- `jpeg_quality_first/final` - double compression
- `random_color_jitter` - random color correction
- `jitter_strength` - jitter intensity (Brightness/Color/Contrast)

**Features:**
- **Sequence**: Jitter -> Noise -> JPEG (low quality) -> JPEG (high quality + EXIF)
- Double JPEG creates compression artifacts
- Color jitter imitates processing variations

**When to use:**
- Quick processing without node chains
- When everything is needed at once
- For mass production of "realistic" images

---

## Typical Workflows

### Workflow 1: Basic AI Image Protection
```
Load Image -> PhotoAddNoise (0.012) -> PhotoAddExif -> PhotoSaveJpg (95)
```

### Workflow 2: Maximum Realism
```
Load Image -> PhotoAdvancedNoise (0.006, grain=False) -> PhotoAddExif -> PhotoSaveJpg (96)
```

### Workflow 3: RAW Processing
```
PhotoLoadRaw -> PhotoAdvancedNoise -> PhotoAddExif -> PhotoSaveRaw (DNG)
```

### Workflow 4: Aesthetic Processing
```
Load Image -> PhotoAddGrain (0.08, size=3) -> PhotoAddExif -> PhotoSaveJpg
```

---

## Parameter Recommendations

| Task | Noise Level | Grain | JPEG Quality | EXIF |
|------|-------------|-------|--------------|------|
| **Subtle protection** | 0.005-0.008 | Off | 95-98 | Required |
| **Medium protection** | 0.01-0.015 | Off | 92-95 | Required |
| **Aggressive protection** | 0.02-0.03 | Size 2-3 | 88-92 | Required |
| **Aesthetics** | 0 | 0.08-0.12, size 3-4 | 95+ | Optional |

---

## Technical Features

1. **Base64 EXIF transfer** - allows metadata transfer between ComfyUI nodes
2. **Color matrices** - correct DNG writing with Calibration Illuminant
3. **HEIC support** - iPhone compatibility via pillow-heif
4. **RAW loading** - universal loader for all formats
5. **Camera presets** - expandable `presets.py` file

---

## Important Notes

1. **Do not use noise >0.03** - will be visible to the eye
2. **Always add EXIF** - protection is meaningless without it
3. **JPEG compression 95+** - lower values degrade quality
4. **PhotoAdvancedNoise is better than PhotoAddNoise** - more realistic sensor noise
5. **Grain and Noise are different** - Grain for aesthetics, Noise for protection

---





# Описание проекта ComfyUI-zveroboy-photo

## Общее назначение проекта

**ComfyUI-zveroboy-photo** — это набор пользовательских нод для ComfyUI, предназначенный для **пост-обработки изображений с целью придания им характеристик реальных фотографий**.

Этот набор нод — **инструмент для исследования** границ детекции AI-изображений, а не для обмана. Его использование должно быть этичным и ответственным.

![comfy-photo](https://github.com/thezveroboy/ComfyUI-zveroboy-photo/blob/main/img1.jpg)

![comfy-photo](https://github.com/thezveroboy/ComfyUI-zveroboy-photo/blob/main/img2.jpg)

### Основные цели:

1. **Исследование уязвимостей детекторов AI-изображений** — демонстрация того, что простые сигнатуры (шум, EXIF) недостаточны для надежной детекции
2. **Имитация реальных камер** — добавление реалистичных характеристик сенсоров, шумов и метаданных
3. **Защита авторства** — усложнение автоматической классификации изображений
4. **Повышение фотореализма** — добавление эстетической зернистости и аналоговых артефактов

### Этическая позиция проекта:
> **Цель НЕ в обмане детекторов**, а в демонстрации их несовершенства и стимулировании разработки более продвинутых алгоритмов, основанных на семантическом анализе, а не на поверхностных признаках.

---

## Описание каждой ноды

### 1. **PhotoAddNoise (Basic)**
**Назначение:** Добавление гауссова шума для защиты от AI-детекторов

**Параметры:**
- `noise_level` (0.0-1.0) — интенсивность шума

**Особенности:**
- Приглушает шум в синем канале (×0.8) для имитации реальных сенсоров
- Равномерный белый шум по всем каналам

**Когда использовать:**
- Быстрая базовая защита
- Минимальное влияние на качество
- Значения 0.01-0.02 для незаметности

---

### 2. **PhotoAddGrain (Aesthetic)**
**Назначение:** Добавление пленочной зернистости для визуальной эстетики

**Параметры:**
- `grain_strength` — сила зерна
- `grain_size` — размер зерна (1-10)

**Особенности:**
- Монохромное зерно (Luminance)
- Наложение через алгоритм Overlay
- Масштабирование зерна для имитации разных пленок

**Когда использовать:**
- Художественная обработка
- Имитация аналоговой фотографии
- Не для защиты (слишком заметно)

---

### 3. **PhotoAddExif**
**Назначение:** Внедрение метаданных камеры (EXIF) для имитации реальных снимков

**Параметры:**
- `preset` — выбор камеры (Canon/Sony/Nikon/Fujifilm/Leica/iPhone)
- `artist`, `software`, `copyright` — авторские данные
- `body_serial`, `lens_serial` — серийные номера (генерируются автоматически)
- `focal_length_mm`, `fnumber`, `exposure_1_over_s`, `iso` — параметры съемки
- `datetime_original` — дата съемки

**Особенности:**
- Возвращает EXIF как Base64 строку для передачи между нодами
- Использует пресеты из `presets.py`
- Автоматическая генерация случайных серийников

**Когда использовать:**
- Всегда перед сохранением JPG
- Для имитации конкретной камеры
- В связке с PhotoSaveJpg

---

### 4. **PhotoLoadRaw**
**Назначение:** Загрузка RAW и HEIC изображений с извлечением EXIF

**Параметры:**
- `raw_file` — полный путь к файлу

**Поддерживаемые форматы:**
- **HEIC** (iPhone через pillow-heif)
- **DNG** (Apple ProRAW, камеры)
- **NEF** (Nikon)
- **ARW** (Sony)
- **CR2/CR3** (Canon)

**Особенности:**
- Двухэтапная загрузка: сначала Pillow, потом rawpy
- Извлечение существующих EXIF данных
- Конвертация в RGB тензор ComfyUI

**Когда использовать:**
- Работа с оригиналами с камеры/iPhone
- Сохранение максимального качества
- Извлечение метаданных для последующей модификации

---

### 5. **PhotoSaveJpg**
**Назначение:** Сохранение в JPEG с внедренными EXIF данными

**Параметры:**
- `exif_data` — Base64 строка с EXIF (от PhotoAddExif)
- `filename_prefix` — префикс имени файла
- `quality` (1-100) — качество JPEG

**Особенности:**
- Сохранение в output_directory ComfyUI
- Timestamp в имени файла
- Оптимизация Huffman coding

**Когда использовать:**
- Финальное сохранение
- Для публикации/передачи
- Когда нужны EXIF метаданные

---

### 6. **PhotoSaveRaw**
**Назначение:** Сохранение в lossless форматы (TIFF/DNG) с цветовыми матрицами

**Параметры:**
- `format` — TIFF или DNG
- `preset` — выбор камеры для матриц
- `exif_data` — метаданные

**Особенности:**
- **DNG**: записывает ColorMatrix1/2, AsShotNeutral, CalibrationIlluminant
- **TIFF**: стандартный lossless с EXIF
- Использует tifffile для правильной структуры DNG
- Сохраняет все данные без потерь

**Когда использовать:**
- Архивация максимального качества
- Передача в профессиональные редакторы
- Сокрытие следов обработки (TIFF выглядит как RAW)

---

### 7. **PhotoAdvancedNoise (Sensor Simulation)**
**Назначение:** Продвинутая имитация шума сенсора камеры для обхода детекторов

**Параметры:**
- `noise_strength` (0.0-0.5) — общая сила
- `color_correlation` — имитация Bayer Pattern (разный шум по каналам)
- `add_grain_layer` — дополнительный слой зерна
- `grain_size` — размер зерна
- `jpeg_compression` — финальное сжатие

**Особенности:**
- **Shot Noise**: зависит от яркости пикселя `(arr ** 0.5)`
- **Read Noise**: постоянная составляющая
- **Color Correlation**: R×1.2, G×0.9, B×1.4 (как у реальных сенсоров)
- **JPEG артефакты**: маскировка AI-гладкости

**Когда использовать:**
- **Вместо PhotoAddNoise** для лучшей защиты
- Значения 0.005-0.008 для незаметности
- Отключать `add_grain_layer` чтобы не портить качество
- `jpeg_compression` 95-98

---

### 8. **Realism7NoiseExif (All-in-One)**
**Назначение:** Комплексная обработка "всё в одном" для быстрого результата

**Параметры:**
- Все параметры EXIF (как в PhotoAddExif)
- `noise_level` — шум
- `jpeg_quality_first/final` — двойное сжатие
- `random_color_jitter` — случайная коррекция цвета
- `jitter_strength` — сила джиттера (Brightness/Color/Contrast)

**Особенности:**
- **Последовательность**: Jitter → Noise → JPEG (низкое качество) → JPEG (высокое качество + EXIF)
- Двойное JPEG создает артефакты сжатия
- Color jitter имитирует вариации обработки

**Когда использовать:**
- Быстрая обработка без цепочки нод
- Когда нужно всё сразу
- Для массового производства "реалистичных" изображений

---

## Типичные рабочие процессы

### Workflow 1: Базовая защита AI-изображения
```
Load Image → PhotoAddNoise (0.012) → PhotoAddExif → PhotoSaveJpg (95)
```

### Workflow 2: Максимальный реализм
```
Load Image → PhotoAdvancedNoise (0.006, grain=False) → PhotoAddExif → PhotoSaveJpg (96)
```

### Workflow 3: Работа с RAW
```
PhotoLoadRaw → PhotoAdvancedNoise → PhotoAddExif → PhotoSaveRaw (DNG)
```

### Workflow 4: Эстетическая обработка
```
Load Image → PhotoAddGrain (0.08, size=3) → PhotoAddExif → PhotoSaveJpg
```

---

## Рекомендации по параметрам

| Задача | Noise Level | Grain | JPEG Quality | EXIF |
|--------|-------------|-------|--------------|------|
| **Незаметная защита** | 0.005-0.008 | Off | 95-98 | Обязательно |
| **Средняя защита** | 0.01-0.015 | Off | 92-95 | Обязательно |
| **Агрессивная защита** | 0.02-0.03 | Size 2-3 | 88-92 | Обязательно |
| **Эстетика** | 0 | 0.08-0.12, size 3-4 | 95+ | По желанию |

---

## Технические особенности проекта

1. **Base64 передача EXIF** — позволяет передавать метаданные между нодами ComfyUI
2. **Цветовые матрицы** — корректная запись DNG с Calibration Illuminant
3. **HEIC поддержка** — работа с iPhone через pillow-heif
4. **RAW загрузка** — универсальный loader для всех форматов
5. **Пресеты камер** — расширяемый файл `presets.py`

---

## Важные замечания

1. **Не используйте шум >0.03** — будет заметно глазу
2. **Всегда добавляйте EXIF** — без этого защита бессмысленна
3. **JPEG compression 95+** — меньшие значения портят качество
4. **PhotoAdvancedNoise лучше PhotoAddNoise** — более реалистичный шум сенсора
5. **Grain и Noise — разные вещи** — Grain для эстетики, Noise для защиты

---


