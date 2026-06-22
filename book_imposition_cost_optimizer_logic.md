# 书刊印刷核价型拼板逻辑说明

## 1. 目标

> 根据书刊产品信息，自动计算不同拼板方案下的纸张利用率、用纸数量、版数、印刷成本、装订成本和总成本，并推荐最合理的报价方案。

---

## 2. 核心问题

系统需要回答以下问题：

1. 这个书刊产品应该使用什么大纸尺寸？
2. 一张大纸可以拼多少页？
3. 应该做 8P、16P、24P 还是 32P 书帖？
4. 每本书需要多少张大纸？
5. 总生产需要多少纸张？
6. 纸张利用率是多少？
7. 需要多少块 CTP 版？
8. 需要多少印刷印次？
9. 印刷、折页、锁线、胶装等工序成本是多少？
10. 哪个方案成本最低？
11. 哪个方案最省纸？
12. 哪个方案最适合生产？

---

## 3. 定位


```text
Book Imposition Cost Optimizer
书刊印刷拼板核价优化
```

输入：

```text
书刊产品参数 + 纸张参数 + 印刷机参数 + 工序价格规则
```
输出：

```text
多个可行拼板核价方案 + 推荐方案 + 成本明细 + 风险提示
```

---

## 4. 核心设计原则

### 4.1 报价优先

第一阶段只需要计算：

- 纸张利用率
- 书帖方案
- 用纸数量
- 损耗数量
- 印版数量
- 印刷印次
- 工序成本
- 单本成本
- 推荐报价方案

第一阶段不需要：

- 输出真实拼版 PDF
- 生成裁切线
- 生成色标
- 生成 JDF
- 自动陷印
- 自动色彩管理

---

### 4.2 枚举方案，而不是固定方案

系统不应该只按照一个固定拼板逻辑计算。

系统应该自动枚举多个可行方案，例如：

- 不同大纸尺寸
- 不同印刷机
- 不同页面方向
- 不同书帖规格
- 不同印刷方式
- 不同分帖组合

然后逐一计算成本，并排序推荐。

---

### 4.3 成本最低不一定最合理

系统不能只按照总成本排序。

某些方案虽然成本最低，但可能存在生产风险，例如：

- 纸纹方向不合适
- 32P 厚纸折页风险高
- 最后一帖太薄
- 补白页太多
- 折页方式复杂
- 不适合锁线或精装

所以系统需要同时输出：

1. 最低成本方案
2. 最省纸方案
3. 生产最稳妥方案
4. 综合推荐方案

---

## 5. 核心输入参数

## 5.1 产品参数 Product

```json
{
  "product_type": "book",
  "trim_width": 210,
  "trim_height": 285,
  "text_pages": 160,
  "cover_pages": 4,
  "quantity": 1000,
  "binding": "sewn_perfect_binding",
  "bleed": 3
}
```

字段说明：

| 字段 | 说明 |
|---|---|
| product_type | 产品类型，例如 book、catalogue、brochure |
| trim_width | 成品宽度，单位 mm |
| trim_height | 成品高度，单位 mm |
| text_pages | 内文页数 |
| cover_pages | 封面页数，通常为 4P |
| quantity | 生产数量 |
| binding | 装订方式 |
| bleed | 出血，常规 3mm |

---

## 5.2 内文印刷参数 Text Print Spec

```json
{
  "paper_name": "157gsm art paper",
  "color_front": 4,
  "color_back": 4,
  "has_spot_color": false,
  "spot_color_count": 0
}
```

字段说明：

| 字段 | 说明 |
|---|---|
| paper_name | 内文纸张 |
| color_front | 正面颜色数 |
| color_back | 反面颜色数 |
| has_spot_color | 是否有专色 |
| spot_color_count | 专色数量 |

---

## 5.3 封面参数 Cover Spec

```json
{
  "paper_name": "300gsm art card",
  "color_front": 4,
  "color_back": 0,
  "lamination": "matt",
  "spot_uv": false,
  "foil": false,
  "embossing": false,
  "flap_width": 0
}
```

字段说明：

| 字段 | 说明 |
|---|---|
| paper_name | 封面纸张 |
| color_front | 封面正面颜色数 |
| color_back | 封面反面颜色数 |
| lamination | 覆膜方式 |
| spot_uv | 是否局部 UV |
| foil | 是否烫金 |
| embossing | 是否击凸 |
| flap_width | 勒口宽度，无勒口则为 0 |

---

## 5.4 纸张参数 Paper

```json
{
  "paper_name": "157gsm art paper",
  "sheet_width": 889,
  "sheet_height": 1194,
  "gsm": 157,
  "price_per_ton": 8000,
  "grain_direction": "long"
}
```

字段说明：

| 字段 | 说明 |
|---|---|
| paper_name | 纸张名称 |
| sheet_width | 大纸宽度，单位 mm |
| sheet_height | 大纸高度，单位 mm |
| gsm | 纸张克重 |
| price_per_ton | 吨价 |
| grain_direction | 纸纹方向，long 或 short |

---

## 5.5 印刷机参数 Press

```json
{
  "press_name": "KBA 105",
  "max_width": 720,
  "max_height": 1020,
  "min_width": 360,
  "min_height": 520,
  "gripper": 12,
  "side_margin": 10,
  "color_bar_space": 8,
  "color_units": 5
}
```

字段说明：

| 字段 | 说明 |
|---|---|
| press_name | 印刷机名称 |
| max_width | 最大上机宽度 |
| max_height | 最大上机高度 |
| min_width | 最小上机宽度 |
| min_height | 最小上机高度 |
| gripper | 咬口 |
| side_margin | 左右边距 |
| color_bar_space | 色标空间 |
| color_units | 色组数量 |

---

## 6. 核心输出结果

系统每个方案应该输出如下结构：

```json
{
  "plan_id": "A",
  "part": "text",
  "signature_size": 16,
  "signature_count": 10,
  "paper_name": "157gsm art paper",
  "paper_size": "889x1194mm",
  "orientation": "rotated",
  "sheets_per_book": 10,
  "net_sheets": 10000,
  "waste_sheets": 1300,
  "total_sheets": 11300,
  "paper_utilization": 0.865,
  "plate_count": 80,
  "print_impressions": 20000,
  "paper_weight_kg": 1883,
  "paper_cost": 15061,
  "plate_cost": 4800,
  "printing_cost": 4200,
  "folding_cost": 900,
  "binding_cost": 1200,
  "total_cost": 26161,
  "unit_cost": 26.16,
  "score": 87,
  "recommendation": "recommended",
  "risk_notes": [
    "Suitable for sewn binding",
    "No blank pages",
    "Paper grain direction OK"
  ]
}
```

---

## 7. 核心计算流程

整体流程：

```text
输入产品参数
    ↓
读取纸张库、印刷机库、工序价格库
    ↓
计算页面含出血尺寸
    ↓
枚举所有纸张尺寸
    ↓
枚举所有印刷机
    ↓
枚举页面方向：normal / rotated
    ↓
枚举书帖规格：4P / 8P / 16P / 24P / 32P
    ↓
判断方案是否可行
    ↓
计算纸张利用率
    ↓
计算书帖数量和补白页
    ↓
计算用纸数量和损耗
    ↓
计算纸张成本
    ↓
计算版数和版费
    ↓
计算印刷成本
    ↓
计算折页、锁线、胶装成本
    ↓
计算总成本和单本成本
    ↓
计算生产风险评分
    ↓
排序并输出推荐方案
```

---

## 8. 页面尺寸计算

页面实际占位尺寸需要包含出血。

公式：

```text
页面占位宽 = 成品宽 + 2 × 出血
页面占位高 = 成品高 + 2 × 出血
```

示例：

```text
成品尺寸：210 × 285mm
出血：3mm

页面占位宽 = 210 + 2 × 3 = 216mm
页面占位高 = 285 + 2 × 3 = 291mm
```

如果页面之间需要额外间距，则：

```text
页面占位宽 = 成品宽 + 2 × 出血 + 页面间距
页面占位高 = 成品高 + 2 × 出血 + 页面间距
```

---

## 9. 可用印刷面积计算

印刷机或上机纸不能全部用于拼版，需要扣除咬口、边距、色标。

公式：

```text
可用宽度 = 上机纸宽度 - 左边距 - 右边距
可用高度 = 上机纸高度 - 咬口 - 尾部边距 - 色标空间
```

示例：

```text
上机纸尺寸：720 × 1020mm
咬口：12mm
左右边距：10mm
尾部边距：10mm
色标空间：8mm

可用宽度 = 720 - 10 - 10 = 700mm
可用高度 = 1020 - 12 - 10 - 8 = 990mm
```

---

## 10. 一面可拼页数计算

### 10.1 不旋转

```text
横向数量 = floor(可用宽度 / 页面占位宽)
纵向数量 = floor(可用高度 / 页面占位高)
一面页数 = 横向数量 × 纵向数量
```

### 10.2 旋转 90 度

```text
横向数量 = floor(可用宽度 / 页面占位高)
纵向数量 = floor(可用高度 / 页面占位宽)
一面页数 = 横向数量 × 纵向数量
```

### 10.3 双面可容纳页数

```text
双面理论页数 = 一面页数 × 2
```

注意：

> 双面理论页数不等于可用书帖页数。书刊必须匹配实际可生产的书帖规格。

例如理论上双面可拼 20P，不代表一定适合作为 20P 书帖。

---

## 11. 可用书帖规格

系统第一阶段建议支持：

```text
4P
8P
16P
24P
32P
```

不同装订方式的优先级不同。

### 11.1 胶装

优先级：

```text
32P → 16P → 8P → 4P
```

### 11.2 锁线胶装

优先级：

```text
16P → 32P → 8P
```

规则：

- 优先 16P 或 32P
- 尽量避免 4P 小帖
- 最后一帖不应太薄
- 纸纹方向必须正确

### 11.3 骑马钉

优先级：

```text
64P → 48P → 40P → 32P → 24P → 16P → 8P
```

规则：

- 总页数必须是 4 的倍数
- 纸张越厚，可接受总页数越少
- 厚书不适合骑马钉

### 11.4 精装书芯

优先级：

```text
16P → 32P → 8P
```

规则：

- 优先稳定书帖
- 避免零散小帖
- 纸纹方向非常重要

---

## 12. 书帖数量和补白页计算

公式：

```text
书帖数量 = ceil(内文页数 / 每帖页数)
补白页 = 书帖数量 × 每帖页数 - 内文页数
```

示例：

```text
内文页数：160P
每帖页数：16P

书帖数量 = ceil(160 / 16) = 10
补白页 = 10 × 16 - 160 = 0
```

示例：

```text
内文页数：164P
每帖页数：16P

书帖数量 = ceil(164 / 16) = 11
补白页 = 11 × 16 - 164 = 12
```

如果补白页过多，系统应降低该方案评分。

---

## 13. 分帖组合优化

单一书帖规格不一定最优。

例如：

```text
168P 可以拆为：
10 × 16P + 1 × 8P
```

而不是：

```text
11 × 16P = 176P，需要补白 8P
```

因此系统应支持混合分帖。

### 13.1 简单混合分帖逻辑

优先使用主书帖，然后用小书帖补足。

```python
def plan_signatures(page_count, preferred_sizes):
    result = []
    remaining = page_count

    for size in preferred_sizes:
        while remaining >= size:
            result.append(size)
            remaining -= size

    if remaining > 0:
        for size in sorted(preferred_sizes):
            if remaining <= size:
                result.append(size)
                blank_pages = size - remaining
                return result, blank_pages

    return result, 0
```

### 13.2 示例

```text
页数：168P
优先规格：[16, 8, 4]

结果：
10 × 16P + 1 × 8P
补白：0P
```

---

## 14. 每本用纸计算

如果每个书帖使用 1 张大纸，则：

```text
每本用纸张数 = 书帖数量
```

对于混合书帖：

```text
每本用纸张数 = 所有书帖数量之和
```

示例：

```text
160P，16P 一帖
书帖数量 = 10

每本内文用纸 = 10 张大纸
```

---

## 15. 净用纸计算

公式：

```text
净用纸张数 = 每本用纸张数 × 生产数量
```

示例：

```text
每本用纸：10 张
数量：1000 本

净用纸 = 10 × 1000 = 10000 张
```

---

## 16. 损耗计算

损耗建议拆成多个部分，而不是只用固定百分比。

### 16.1 损耗组成

```text
总损耗 =
印刷调机损耗
+ 印刷过程损耗
+ 折页损耗
+ 装订损耗
+ 后道加工损耗
```

### 16.2 印刷调机损耗

公式：

```text
印刷调机损耗 = 印张数量 × 每个印张调机损耗
```

示例：

```text
印张数量：10
每个印张调机损耗：100 张

印刷调机损耗 = 10 × 100 = 1000 张
```

### 16.3 印刷过程损耗

公式：

```text
印刷过程损耗 = 净用纸张数 × 印刷损耗率
```

示例：

```text
净用纸：10000 张
印刷损耗率：2%

印刷过程损耗 = 10000 × 2% = 200 张
```

### 16.4 折页损耗

公式：

```text
折页损耗 = 净用纸张数 × 折页损耗率
```

### 16.5 总用纸

公式：

```text
总用纸张数 = 净用纸张数 + 总损耗张数
```

---

## 17. 纸张重量计算

公式：

```text
单张纸重量 kg = 纸宽 m × 纸高 m × 克重 gsm ÷ 1000
```

示例：

```text
大纸尺寸：889 × 1194mm
克重：157gsm

单张纸重量 =
0.889 × 1.194 × 157 ÷ 1000
≈ 0.1666 kg
```

总重量：

```text
总纸重 kg = 单张纸重量 kg × 总用纸张数
```

---

## 18. 纸张成本计算

如果纸张按吨价计算：

```text
纸张成本 = 总纸重 kg ÷ 1000 × 每吨价格
```

示例：

```text
总纸重：1883 kg
纸价：8000 / 吨

纸张成本 = 1883 ÷ 1000 × 8000
≈ 15064
```

---

## 19. 纸张利用率计算

### 19.1 理论成品利用率

公式：

```text
理论成品利用率 =
单页成品宽 × 单页成品高 × 每张大纸包含页数
÷
大纸宽 × 大纸高
```

示例：

```text
成品尺寸：210 × 285mm
书帖：16P
大纸：889 × 1194mm

有效成品面积 =
210 × 285 × 16
= 957600 mm²

大纸面积 =
889 × 1194
= 1061466 mm²

理论成品利用率 =
957600 ÷ 1061466
≈ 90.2%
```

### 19.2 实际利用率

实际利用率应考虑：

- 出血
- 咬口
- 色标
- 裁切边
- 页面间距
- 纸纹方向限制

公式可简化为：

```text
实际利用率 =
页面占位宽 × 页面占位高 × 每张大纸包含页数
÷
可用印刷面积
```

系统应同时显示：

```text
理论成品利用率
实际拼板利用率
```

---

## 20. 版数计算

公式：

```text
版数 = 印张数量 × (正面颜色数 + 反面颜色数)
```

对于 CMYK 双面：

```text
正面颜色数 = 4
反面颜色数 = 4
每个印张版数 = 8
```

示例：

```text
书帖数量：10
每帖正反 4C + 4C

版数 = 10 × (4 + 4) = 80 块版
```

如果有专色：

```text
版数 = 印张数量 × (正面颜色数 + 反面颜色数 + 专色数量)
```

---

## 21. 版费计算

公式：

```text
版费 = 版数 × 单块版价格
```

示例：

```text
版数：80
单块版价格：60

版费 = 80 × 60 = 4800
```

---

## 22. 印刷印次计算

公式：

```text
印刷印次 = 印张数量 × 生产数量 × 印刷面数
```

双面印刷：

```text
印刷面数 = 2
```

示例：

```text
印张数量：10
生产数量：1000
印刷面数：2

印刷印次 = 10 × 1000 × 2 = 20000
```

---

## 23. 印刷成本计算

建议支持两种模式。

### 23.1 按千印计算

```text
印刷成本 = 印刷印次 ÷ 1000 × 每千印单价
```

### 23.2 按开机费 + 运行费计算

```text
印刷成本 =
印张数量 × 每印张开机费
+
印刷印次 × 单次运行费
```

示例：

```text
印张数量：10
每印张开机费：300
印刷印次：20000
单次运行费：0.06

印刷成本 =
10 × 300 + 20000 × 0.06
= 4200
```

---

## 24. 折页成本计算

公式：

```text
折页数量 = 书帖数量 × 生产数量
```

折页成本：

```text
折页成本 = 折页数量 ÷ 1000 × 每千帖折页单价
```

或者：

```text
折页成本 = 折页开机费 + 折页数量 ÷ 1000 × 每千帖单价
```

---

## 25. 锁线成本计算

适用于锁线胶装、精装书芯。

公式：

```text
锁线成本 = 书帖数量 × 生产数量 × 每帖锁线单价
```

示例：

```text
书帖数量：10
数量：1000
每帖锁线单价：0.03

锁线成本 = 10 × 1000 × 0.03 = 300
```

---

## 26. 胶装成本计算

公式：

```text
胶装成本 = 生产数量 × 每本胶装单价
```

示例：

```text
数量：1000
每本胶装：1.20

胶装成本 = 1000 × 1.20 = 1200
```

---

## 27. 封面核价逻辑

封面应单独核价，不与内文混算。

### 27.1 普通胶装封面展开尺寸

无勒口：

```text
封面展开宽 = 封底宽 + 书脊宽 + 封面宽
封面展开高 = 成品高度
```

有勒口：

```text
封面展开宽 =
后勒口 + 封底宽 + 书脊宽 + 封面宽 + 前勒口
封面展开高 = 成品高度
```

### 27.2 加出血后的封面占位

```text
封面占位宽 = 封面展开宽 + 2 × 出血
封面占位高 = 封面展开高 + 2 × 出血
```

### 27.3 书脊厚度估算

公式：

```text
书脊厚度 = 内文页数 ÷ 2 × 单张纸厚度
```

示例：

```text
内文：160P
即 80 张纸
单张纸厚度：0.1mm

书脊厚度 = 80 × 0.1 = 8mm
```

### 27.4 封面一张大纸可拼数量

```text
横向数量 = floor(大纸宽 / 封面占位宽)
纵向数量 = floor(大纸高 / 封面占位高)

每张大纸可拼封面数量 = 横向数量 × 纵向数量
```

需要同时计算旋转方案，并选择更优结果。

### 27.5 封面净用纸

```text
封面净用纸 = ceil(生产数量 / 每张大纸可拼封面数量)
```

### 27.6 封面成本组成

封面成本包括：

- 封面纸张成本
- 封面版费
- 封面印刷成本
- 覆膜成本
- 压痕成本
- 局部 UV 成本
- 烫金成本
- 击凸成本
- 裁切成本
- 加工损耗

---

## 28. 装订方式规则

## 28.1 骑马钉 Saddle Stitch

规则：

- 总页数必须是 4 的倍数
- 通常适合 8P 到 64P
- 纸张越厚，最大页数越低
- 需要计算爬移，但第一阶段可只提示风险

风险判断：

```text
如果页数 > 64P，提示不建议骑马钉
如果纸张克重较高且页数较多，提示厚度风险
```

---

## 28.2 胶装 Perfect Binding

规则：

- 可接受 8P、16P、32P 混合
- 最后一帖不应太薄
- 可以有少量补白页
- 纸纹方向应正确

风险判断：

```text
如果最后一帖只有 4P，扣分
如果补白页超过 4P，扣分
```

---

## 28.3 锁线胶装 Sewn Perfect Binding

规则：

- 优先 16P 或 32P
- 避免 4P 小帖
- 最后一帖最好是 8P 或以上
- 纸纹方向必须正确

风险判断：

```text
如果出现 4P 书帖，扣分
如果 32P 用于厚纸，扣分
如果纸纹方向错误，大幅扣分
```

---

## 28.4 精装 Hardcover

规则：

- 内文书芯通常优先 16P 或 32P
- 需要考虑环衬、书壳、灰板、堵头布等成本
- 纸纹方向非常重要
- 装订稳定性优先于最低成本

风险判断：

```text
如果方案为了省纸导致装订风险高，不推荐
```

---

## 29. 纸纹方向判断

纸纹方向对书刊很重要。

简化原则：

```text
书脊方向应尽量顺纸纹
```

系统应判断页面在大纸上的方向与纸张 grain_direction 是否匹配。

输出示例：

```text
Paper grain direction OK
```

或：

```text
Warning: Paper grain direction may not be suitable for book binding.
```

评分规则：

```text
纸纹方向正确：不扣分
纸纹方向不确定：扣 10 分
纸纹方向错误：扣 30 分
```

---

## 30. 方案可行性检查

每个方案必须通过以下检查。

### 30.1 尺寸检查

```text
拼版所需尺寸 <= 可用印刷尺寸
上机纸尺寸 <= 印刷机最大尺寸
上机纸尺寸 >= 印刷机最小尺寸
```

### 30.2 书帖检查

```text
书帖规格属于允许列表
书帖规格适合当前装订方式
补白页不能过多
最后一帖不能过薄
```

### 30.3 颜色检查

```text
正面颜色数 <= 印刷机色组数量
反面颜色数 <= 印刷机色组数量
专色数量不能超过机器能力
```

### 30.4 生产检查

```text
纸张克重适合该折页方式
32P 厚纸需要提示风险
骑马钉页数不能太多
```

---

## 31. 方案评分模型

综合评分用于推荐方案。

建议权重：

```text
总成本评分：40%
纸张利用率评分：25%
生产稳定性评分：20%
装订适配评分：10%
交期评分：5%
```

公式：

```python
final_score = (
    cost_score * 0.40
    + paper_utilization_score * 0.25
    + production_stability_score * 0.20
    + binding_score * 0.10
    + delivery_score * 0.05
)
```

---

## 32. 扣分规则示例

```text
纸纹方向错误：-30
纸纹方向不确定：-10
32P 厚纸折页：-15
出现 4P 小帖：-10
最后一帖只有 4P：-10
补白页超过 4P：-8
补白页超过 8P：-15
纸张利用率低于 70%：-15
需要特殊纸张尺寸：-5
需要更换印刷机：-5
骑马钉页数超过 64P：-30
```

---

## 33. 推荐输出格式

系统最终不应只输出一个方案。

建议输出：

### 33.1 最低成本方案

```text
适合普通商业订单，报价最有竞争力。
```

### 33.2 最省纸方案

```text
纸张利用率最高，但可能工序成本或生产风险更高。
```

### 33.3 生产最稳妥方案

```text
适合高端画册、艺术书、精装书、色彩要求高的订单。
```

### 33.4 综合推荐方案

```text
综合考虑成本、纸张利用率和生产风险后的推荐方案。
```

---

## 34. 输出表格示例

| 方案 | 书帖 | 大纸 | 利用率 | 补白 | 版数 | 总成本 | 单本成本 | 建议 |
|---|---:|---|---:|---:|---:|---:|---:|---|
| A | 16P × 10 | 889×1194 | 86.5% | 0P | 80 | 32800 | 32.80 | 推荐 |
| B | 32P × 5 | 889×1194 | 90.1% | 0P | 40 | 30900 | 30.90 | 成本最低 |
| C | 8P × 20 | 787×1092 | 78.4% | 0P | 160 | 36500 | 36.50 | 不推荐 |

说明示例：

```text
方案 B 成本最低，但如果使用 157gsm 铜版纸且客户要求高端画册，32P 折页风险较高。综合建议使用方案 A。
```

---

## 35. 主要函数设计

## 35.1 calculate_page_box

计算含出血页面尺寸。

```python
def calculate_page_box(trim_width, trim_height, bleed):
    return {
        "page_width": trim_width + bleed * 2,
        "page_height": trim_height + bleed * 2
    }
```

---

## 35.2 calculate_usable_area

计算可用印刷面积。

```python
def calculate_usable_area(sheet_width, sheet_height, gripper, side_margin, tail_margin, color_bar_space):
    usable_width = sheet_width - side_margin * 2
    usable_height = sheet_height - gripper - tail_margin - color_bar_space

    return {
        "usable_width": usable_width,
        "usable_height": usable_height
    }
```

---

## 35.3 calculate_nup

计算一面可拼数量。

```python
import math

def calculate_nup(usable_width, usable_height, page_width, page_height):
    normal_cols = math.floor(usable_width / page_width)
    normal_rows = math.floor(usable_height / page_height)
    normal_count = normal_cols * normal_rows

    rotated_cols = math.floor(usable_width / page_height)
    rotated_rows = math.floor(usable_height / page_width)
    rotated_count = rotated_cols * rotated_rows

    return [
        {
            "orientation": "normal",
            "cols": normal_cols,
            "rows": normal_rows,
            "pages_per_side": normal_count,
            "pages_per_sheet": normal_count * 2
        },
        {
            "orientation": "rotated",
            "cols": rotated_cols,
            "rows": rotated_rows,
            "pages_per_side": rotated_count,
            "pages_per_sheet": rotated_count * 2
        }
    ]
```

---

## 35.4 plan_signatures

计算书帖组合。

```python
import math

def plan_signatures(page_count, preferred_sizes):
    result = []
    remaining = page_count

    for size in preferred_sizes:
        while remaining >= size:
            result.append(size)
            remaining -= size

    blank_pages = 0

    if remaining > 0:
        for size in sorted(preferred_sizes):
            if remaining <= size:
                result.append(size)
                blank_pages = size - remaining
                break

    return {
        "signatures": result,
        "signature_count": len(result),
        "blank_pages": blank_pages
    }
```

---

## 35.5 calculate_paper_weight

计算纸张重量。

```python
def calculate_paper_weight(sheet_width_mm, sheet_height_mm, gsm, sheet_count):
    width_m = sheet_width_mm / 1000
    height_m = sheet_height_mm / 1000

    single_sheet_kg = width_m * height_m * gsm / 1000
    total_weight_kg = single_sheet_kg * sheet_count

    return {
        "single_sheet_kg": single_sheet_kg,
        "total_weight_kg": total_weight_kg
    }
```

---

## 35.6 calculate_plate_count

计算版数。

```python
def calculate_plate_count(signature_count, color_front, color_back, spot_color_count=0):
    return signature_count * (color_front + color_back + spot_color_count)
```

---

## 35.7 calculate_print_impressions

计算印刷印次。

```python
def calculate_print_impressions(signature_count, quantity, sides=2):
    return signature_count * quantity * sides
```

---

## 35.8 calculate_total_cost

计算总成本。

```python
def calculate_total_cost(cost_items):
    return sum(cost_items.values())
```

示例输入：

```python
cost_items = {
    "paper_cost": 15061,
    "plate_cost": 4800,
    "printing_cost": 4200,
    "folding_cost": 900,
    "binding_cost": 1200
}
```

---

## 36. 主算法伪代码

```python
def generate_cost_plans(product, papers, presses, cost_rules):
    plans = []

    page_box = calculate_page_box(
        product["trim_width"],
        product["trim_height"],
        product["bleed"]
    )

    for paper in papers:
        for press in presses:
            usable_area = calculate_usable_area(
                sheet_width=min(paper["sheet_width"], press["max_width"]),
                sheet_height=min(paper["sheet_height"], press["max_height"]),
                gripper=press["gripper"],
                side_margin=press["side_margin"],
                tail_margin=cost_rules["tail_margin"],
                color_bar_space=press["color_bar_space"]
            )

            nup_options = calculate_nup(
                usable_area["usable_width"],
                usable_area["usable_height"],
                page_box["page_width"],
                page_box["page_height"]
            )

            for nup in nup_options:
                for signature_size in [4, 8, 16, 24, 32]:
                    if signature_size > nup["pages_per_sheet"]:
                        continue

                    if not is_signature_allowed(signature_size, product["binding"]):
                        continue

                    signature_plan = plan_signatures(
                        product["text_pages"],
                        get_preferred_signature_sizes(product["binding"])
                    )

                    plan = calculate_plan_cost(
                        product=product,
                        paper=paper,
                        press=press,
                        nup=nup,
                        signature_plan=signature_plan,
                        cost_rules=cost_rules
                    )

                    if validate_plan(plan):
                        plans.append(plan)

    return sort_and_recommend(plans)
```

---

## 37. 成本规则数据结构

```json
{
  "plate_price": 60,
  "make_ready_sheets_per_signature": 100,
  "print_waste_rate": 0.02,
  "folding_waste_rate": 0.01,
  "binding_waste_rate": 0.01,
  "print_make_ready_cost_per_signature": 300,
  "print_running_cost_per_impression": 0.06,
  "folding_cost_per_1000_signature": 90,
  "sewing_cost_per_signature": 0.03,
  "perfect_binding_cost_per_book": 1.2,
  "tail_margin": 10,
  "profit_rate": 0.25,
  "management_rate": 0.08
}
```

---

## 38. 报价计算

生产成本：

```text
生产成本 =
纸张成本
+ 版费
+ 印刷成本
+ 折页成本
+ 锁线成本
+ 胶装成本
+ 封面成本
+ 包装成本
```

含管理费：

```text
含管理费成本 = 生产成本 × (1 + 管理费率)
```

报价：

```text
报价 = 含管理费成本 × (1 + 利润率)
```

单本报价：

```text
单本报价 = 报价 ÷ 数量
```

澳洲报价可扩展：

```text
澳币报价 =
人民币报价 ÷ 汇率
+ 国际运输
+ 本地仓储
+ 本地派送
```

GST：

```text
GST = 澳洲报价 × 10%
```

---

## 39. MVP 第一版范围

第一版建议实现以下功能。

### 39.1 支持产品

- 胶装书
- 锁线胶装书
- 骑马钉小册子
- 精装书内文
- 普通封面

### 39.2 支持书帖

- 4P
- 8P
- 16P
- 24P
- 32P

### 39.3 支持计算

- 页面含出血尺寸
- 一面可拼数量
- 双面理论页数
- 书帖数量
- 补白页
- 净用纸
- 损耗用纸
- 总用纸
- 纸张重量
- 纸张成本
- 理论纸张利用率
- 实际纸张利用率
- 版数
- 版费
- 印刷印次
- 印刷成本
- 折页成本
- 锁线成本
- 胶装成本
- 总成本
- 单本成本
- 方案评分
- 推荐方案

### 39.4 暂不支持

- 真实 PDF 拼版
- JDF 输出
- 自动陷印
- 色彩管理
- 专色版拆分
- 复杂包装盒拼版
- 自动排产

---

## 40. 数据库表建议

## 40.1 papers

| 字段 | 类型 | 说明 |
|---|---|---|
| id | string | 纸张 ID |
| name | string | 纸张名称 |
| width | number | 大纸宽度 |
| height | number | 大纸高度 |
| gsm | number | 克重 |
| price_per_ton | number | 吨价 |
| grain_direction | string | 纸纹方向 |
| supplier | string | 供应商 |

---

## 40.2 presses

| 字段 | 类型 | 说明 |
|---|---|---|
| id | string | 印刷机 ID |
| name | string | 印刷机名称 |
| max_width | number | 最大宽度 |
| max_height | number | 最大高度 |
| min_width | number | 最小宽度 |
| min_height | number | 最小高度 |
| gripper | number | 咬口 |
| side_margin | number | 左右边距 |
| color_bar_space | number | 色标空间 |
| color_units | number | 色组数量 |

---

## 40.3 cost_rules

| 字段 | 类型 | 说明 |
|---|---|---|
| plate_price | number | 单块版价格 |
| make_ready_sheets_per_signature | number | 每印张调机损耗 |
| print_waste_rate | number | 印刷损耗率 |
| folding_waste_rate | number | 折页损耗率 |
| binding_waste_rate | number | 装订损耗率 |
| print_make_ready_cost_per_signature | number | 每印张开机费 |
| print_running_cost_per_impression | number | 单印次运行费 |
| folding_cost_per_1000_signature | number | 每千帖折页费 |
| sewing_cost_per_signature | number | 每帖锁线费 |
| perfect_binding_cost_per_book | number | 每本胶装费 |
| profit_rate | number | 利润率 |
| management_rate | number | 管理费率 |

---

## 41. 测试案例

## 41.1 测试案例 A：160P 锁线胶装画册

输入：

```json
{
  "trim_width": 210,
  "trim_height": 285,
  "text_pages": 160,
  "quantity": 1000,
  "binding": "sewn_perfect_binding",
  "bleed": 3,
  "text_paper": "157gsm art paper",
  "color_front": 4,
  "color_back": 4
}
```

预期：

```text
推荐 16P 或 32P 方案
160P 可整除
补白 0P
16P 方案为 10 个书帖
32P 方案为 5 个书帖
需要比较纸张利用率和折页风险
```

---

## 41.2 测试案例 B：168P 胶装书

输入：

```json
{
  "trim_width": 148,
  "trim_height": 210,
  "text_pages": 168,
  "quantity": 2000,
  "binding": "perfect_binding",
  "bleed": 3,
  "text_paper": "128gsm art paper",
  "color_front": 4,
  "color_back": 4
}
```

预期：

```text
推荐混合分帖
例如 10 × 16P + 1 × 8P
补白 0P
```

---

## 41.3 测试案例 C：36P 骑马钉

输入：

```json
{
  "trim_width": 210,
  "trim_height": 297,
  "text_pages": 36,
  "quantity": 5000,
  "binding": "saddle_stitch",
  "bleed": 3,
  "text_paper": "150gsm art paper",
  "color_front": 4,
  "color_back": 4
}
```

预期：

```text
36P 是 4 的倍数
可以骑马钉
需要计算是否适合 150gsm
页数和厚度需要提示风险但不一定禁止
```

---

## 42. 后续第二阶段功能

第二阶段可以增加真正的印前功能：

- 读取 PDF 页数和尺寸
- PDF 预检
- 自动补白页
- 生成拼版预览
- 生成真实拼版 PDF
- 添加裁切线
- 添加折线
- 添加咬口标记
- 添加色标
- 添加印张信息
- 输出 CTP 文件
- 输出 JDF / XML
- 与 ERP 报价系统连接

---

## 43. 重要结论

本系统的核心不是“把页面排到纸上”，而是：

> 通过枚举不同纸张、不同书帖、不同拼板方向和不同工艺路径，找到最适合报价和生产的方案。

第一版最重要的能力：

```text
1. 输入书刊参数
2. 枚举纸张和书帖方案
3. 计算纸张利用率
4. 计算用纸数量
5. 计算版数
6. 计算印刷和装订成本
7. 输出最低成本、最省纸、最稳妥和综合推荐方案
```

真正有价值的数据资产：

```text
纸张尺寸库
印刷机参数库
书帖规则库
装订规则库
工序价格库
损耗规则库
报价利润规则
```

只要这些规则库建立起来，系统就可以成为印刷厂内部核价和报价的重要工具。
