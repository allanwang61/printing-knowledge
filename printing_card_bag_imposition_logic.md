# 印刷纸卡与包装手提袋类拼板逻辑

> 用途：供 Codex / AI 助手学习印刷行业中「纸卡类」与「包装手提袋类」产品的拼板、核价与生产可行性判断逻辑。  
> 目标：根据产品信息自动判断展开尺寸、出血尺寸、拼板方案、纸张利用率、印刷张数、损耗、后道工艺与综合成本。

---

## 目录

1. 核心目标
2. 通用输入参数
3. 纸卡类拼板逻辑
4. 包装手提袋类拼板逻辑
5. 通用拼板算法
6. 损耗逻辑
7. 后道工艺影响
8. 综合成本判断
9. 输出字段
10. 伪代码
11. 示例

---

# 1. 核心目标

印刷拼板的目的不是单纯看产品能否排进纸张，而是为了找到综合成本最低且生产可行的方案。

系统需要同时考虑：

- 成品尺寸
- 展开尺寸
- 出血
- 咬口
- 拉规
- 印刷机最大上机尺寸
- 可采购纸张尺寸
- 纸纹方向
- 拼数
- 纸张利用率
- 印刷张数
- 后道加工方式
- 损耗率
- 是否适合合版
- 是否适合机器生产
- 是否有生产风险

最终推荐方案应该满足：

```text
综合成本最低
+ 纸张利用率合理
+ 印刷机可上机
+ 后道设备可加工
+ 工艺风险可控
```

---

# 2. 通用输入参数

所有纸卡和包装类产品都应该先收集以下参数。

```yaml
product_type: 产品类型
finished_width: 成品宽度
finished_height: 成品高度
finished_depth: 成品厚度或侧宽，手提袋需要
flat_width: 展开宽度，可由系统计算或人工输入
flat_height: 展开高度，可由系统计算或人工输入
quantity: 成品数量

paper_type: 纸张类型
paper_weight: 纸张克重
paper_size_list: 可选纸张尺寸列表
grain_direction_required: 是否要求纸纹方向
grain_direction: 纸纹方向

printing_colors_front: 正面印刷颜色
printing_colors_back: 反面印刷颜色
spot_color: 是否有专色
full_coverage: 是否满版印刷
dark_solid_background: 是否深色大面积实地

bleed: 出血
gap_between_items: 产品之间间距
gripper_margin: 咬口
side_margin: 拉规 / 侧规 / 安全边
tail_margin: 尾边安全边

surface_finish: 表面处理
die_cut: 是否模切
creasing: 是否压痕
folding: 是否折叠
gluing: 是否糊盒 / 糊袋
hole_punching: 是否打孔
round_corner: 是否倒圆角
foil_stamping: 是否烫金
embossing: 是否击凸
spot_uv: 是否局部 UV

machine_print_max_size: 印刷机最大上机尺寸
machine_die_cut_max_size: 模切机最大加工尺寸
machine_lamination_max_size: 覆膜机最大加工尺寸
machine_gluing_max_size: 糊袋 / 糊盒机最大加工尺寸
```

---

# 3. 纸卡类拼板逻辑

## 3.1 纸卡类产品范围

纸卡类产品包括但不限于：

- 明信片
- 贺卡
- 吊牌
- 书签
- 包装背卡
- 插卡
- 说明卡
- 展示卡
- 封套卡
- 折卡
- 票券
- 小型包装纸卡部件
- 异形模切卡

---

## 3.2 纸卡类核心特点

纸卡通常属于平面产品，拼板重点是：

```text
成品尺寸
+ 出血
+ 拼数
+ 纸张利用率
+ 是否折叠
+ 是否模切
+ 是否需要特殊工艺
```

纸卡类可以优先追求高拼数和高纸张利用率，但必须确认后道加工可行。

---

## 3.3 单张纸卡开料尺寸

普通单张纸卡：

```text
开料宽度 = 成品宽度 + 左出血 + 右出血
开料高度 = 成品高度 + 上出血 + 下出血
```

通常：

```yaml
普通纸卡出血: 3 mm
异形模切纸卡出血: 3-5 mm
深色满版纸卡出血: 3-5 mm
```

示例：

```text
成品尺寸：100 × 150 mm
出血：3 mm

开料宽度 = 100 + 3 + 3 = 106 mm
开料高度 = 150 + 3 + 3 = 156 mm
```

---

## 3.4 折卡展开尺寸

折卡不能只看成品尺寸，必须使用展开尺寸。

例如 A6 对折卡：

```text
成品尺寸：105 × 148 mm
展开尺寸：210 × 148 mm
出血后尺寸：216 × 154 mm
```

折卡需要额外判断：

- 折线方向
- 纸纹方向
- 压痕方向
- 正反面对位
- 折后是否容易爆线
- 覆膜后是否容易爆线

---

## 3.5 异形卡 / 吊牌占位尺寸

异形产品应该以刀线最大外框作为拼板占位尺寸。

```text
拼板占位宽度 = 刀线最大宽度 + 左右出血 + 刀间距
拼板占位高度 = 刀线最大高度 + 上下出血 + 刀间距
```

还要判断：

- 打孔位置
- 圆角位置
- 异形边缘
- 清废空间
- 模切刀线安全距离
- 是否适合自动模切

异形卡不能只按矩形面积判断，因为废料通常更多。

---

## 3.6 纸卡拼板计算

### Step 1：计算有效印刷区域

```text
有效印刷宽度 = 上机纸宽度 - 左安全边 - 右安全边
有效印刷高度 = 上机纸高度 - 咬口 - 尾边安全边
```

### Step 2：计算横向和纵向可排数量

```text
横向可排数量 = floor((有效印刷宽度 + 间距) / (单个开料宽度 + 间距))

纵向可排数量 = floor((有效印刷高度 + 间距) / (单个开料高度 + 间距))

每张拼数 = 横向可排数量 × 纵向可排数量
```

### Step 3：测试旋转方向

纸卡通常需要比较两个方向：

```text
方案 A：正常方向，宽 × 高
方案 B：旋转 90°，高 × 宽
```

选择时比较：

- 拼数
- 纸张利用率
- 纸纹方向
- 后道稳定性
- 是否适合模切
- 是否适合折叠
- 综合成本

---

## 3.7 纸卡纸张利用率

```text
纸张利用率 = 单个开料面积 × 每张拼数 ÷ 上机纸面积
```

一般情况下：

```text
利用率越高，纸张成本越低
但利用率不是唯一标准
```

如果高利用率方案导致以下问题，则不应直接采用：

- 模切不稳定
- 间距过小
- 清废困难
- 压痕容易爆线
- 纸纹方向错误
- 后道机器无法加工
- 印刷叼口不足
- 色标 / 套准线无空间

---

## 3.8 纸卡合版逻辑

纸卡很适合合版，但必须满足以下条件：

```yaml
paper_type: 相同
paper_weight: 相同
printing_color: 相同或接近
surface_finish: 相同
delivery_time: 接近
die_cut_method: 可兼容
special_finish: 可兼容
quantity_level: 不应差异过大
```

不建议合版的情况：

- 一个覆膜，一个不覆膜
- 一个有专色，一个没有专色
- 一个烫金，一个不烫金
- 一个普通直角，一个复杂异形
- 一个颜色要求极高，一个普通要求
- 数量差距过大
- 纸纹方向冲突
- 交货期差异大

---

# 4. 包装手提袋类拼板逻辑

## 4.1 手提袋类产品范围

手提袋类产品包括：

- 普通纸手提袋
- 牛皮纸袋
- 白卡纸手提袋
- 铜版纸覆膜手提袋
- 服装手提袋
- 礼品手提袋
- 展会手提袋
- 艺术馆手提袋
- 酒类手提袋
- 高端手工袋
- 带绳手提袋
- 带扣手提袋
- 不打扣穿绳手提袋

---

## 4.2 手提袋与纸卡的区别

纸卡通常是：

```text
单片平面产品
尺寸固定
拼数越多越省
后道相对简单
```

手提袋则是：

```text
展开结构产品
包含正面、背面、侧边、底部、糊口
有压痕线
有袋口折边
有穿绳孔
有糊袋方向
纸纹方向重要
后道成本高
人工成本高
```

因此手提袋拼板必须先计算展开尺寸，而不是直接用成品尺寸。

---

## 4.3 手提袋结构参数

设定：

```yaml
W: 袋宽
H: 袋高
D: 袋侧宽 / 袋厚
G: 糊口宽度
T: 顶部折口
B: 袋底折边
```

常规展开尺寸：

```text
展开宽度 = W × 2 + D × 2 + G

展开高度 = H + T + B
```

示例：

```text
成品袋尺寸：250 × 320 × 100 mm

W = 250
H = 320
D = 100
G = 20
T = 40
B = 80

展开宽度 = 250 × 2 + 100 × 2 + 20 = 720 mm
展开高度 = 320 + 40 + 80 = 440 mm
```

注意：不同袋型的袋底折边 B 可能不同，实际生产中需要根据工厂刀版结构确认。

---

## 4.4 手提袋拼板占位尺寸

```text
拼板占位宽度 = 展开宽度 + 左右出血 + 刀线安全边
拼板占位高度 = 展开高度 + 上下出血 + 刀线安全边
```

常见出血建议：

```yaml
普通手提袋: 3 mm
覆膜手提袋: 3-5 mm
深色满版手提袋: 5 mm
异形袋 / 特殊结构: 5 mm
```

还需要预留：

- 咬口
- 拉规
- 色标
- 套准线
- 压痕安全位
- 模切安全边
- 糊口安全边

---

## 4.5 手提袋纸纹方向

手提袋的纸纹方向非常重要。

纸纹会影响：

- 袋身挺度
- 袋口承重
- 折线效果
- 压痕是否爆线
- 糊袋稳定性
- 成品站立效果

规则：

```text
如果有纸纹方向要求，则拼板方向不能随意旋转。

如果旋转后更省纸，但纸纹方向错误，需要提示人工确认。

如果纸张较厚、深色满版、覆膜或大尺寸袋，应优先保证纸纹和压痕稳定性。
```

---

## 4.6 手提袋拼板计算

手提袋展开尺寸一般较大，常见拼法：

```text
一张纸拼 1 个袋
一张纸拼 2 个袋
小尺寸袋可能拼 3 个或更多
```

计算方式：

```text
横向可排数量 = floor((有效印刷宽度 + 间距) / (袋展开占位宽度 + 间距))

纵向可排数量 = floor((有效印刷高度 + 间距) / (袋展开占位高度 + 间距))

每张拼数 = 横向可排数量 × 纵向可排数量
```

同时必须检查：

- 印刷机是否能印
- 模切机是否能切
- 覆膜机是否能覆
- 糊袋机是否能糊
- 纸纹是否正确
- 压痕方向是否合理
- 是否方便清废
- 是否方便穿绳 / 打扣

---

## 4.7 手提袋旋转判断

```text
如果无纸纹要求：
    可以比较 0° 和 90° 两种排法

如果有纸纹要求：
    只能使用符合纸纹的排法

如果旋转后影响压痕、糊袋或承重：
    不允许使用该排法
```

---

## 4.8 手提袋后道流程

常见流程：

```text
印刷
→ 覆膜
→ 模切
→ 压痕
→ 糊袋
→ 打孔
→ 穿绳
→ 打扣
→ 加固
→ 包装
```

并不是每个袋子都有所有工艺，需要根据客户要求选择。

---

## 4.9 手提袋糊袋方式

常见分类：

```yaml
single_piece_gluing: 单片糊
two_piece_gluing: 两片糊
machine_gluing: 机器糊袋
manual_gluing: 手工糊袋
```

影响糊袋费用的因素：

- 袋子尺寸
- 纸张厚度
- 是否覆膜
- 糊口结构
- 数量
- 是否能机器生产
- 是否需要人工辅助

---

## 4.10 手提袋穿绳 / 打扣方式

常见方式：

```yaml
rope_without_eyelet: 不打扣 + 包头绳
rope_with_eyelet: 打扣 + 包头绳
rope_with_eyelet_and_knot: 打扣 + 打结绳
direct_rope_threading: 不打扣穿绳
cotton_rope: 棉绳
pp_rope: PP 绳
ribbon: 丝带
special_rope: 特种绳
```

核价时需要拆分：

```text
绳子材料费用
+ 打孔费用
+ 打扣费用
+ 穿绳人工费用
+ 打结人工费用
+ 包头费用
```

---

## 4.11 加固卡 / 底卡

高端手提袋通常包含：

- 袋口加固卡
- 袋底卡

这部分可以按纸卡逻辑单独拼板。

计算时注意：

```text
底卡数量通常 = 手提袋数量

袋口加固卡数量通常 = 手提袋数量 × 2
```

需要单独判断：

- 加固卡尺寸
- 底卡尺寸
- 是否印刷
- 是否覆膜
- 是否模切
- 是否需要粘贴

---

# 5. 通用拼板算法

## 5.1 可用纸张尺寸数据库

系统需要维护纸张尺寸列表，例如：

```yaml
paper_sizes:
  - name: 正度全开
    width: 787
    height: 1092

  - name: 大度全开
    width: 889
    height: 1194

  - name: 特规纸
    width: 700
    height: 1000

  - name: 特规纸
    width: 720
    height: 1020
```

具体尺寸应该根据工厂实际采购纸张和印刷机上机尺寸维护。

---

## 5.2 有效印刷区域

```text
有效宽度 = 纸张宽度 - 左安全边 - 右安全边

有效高度 = 纸张高度 - 咬口 - 尾边安全边
```

如果需要色标或套准线，应额外扣除空间。

---

## 5.3 遍历逻辑

系统应该遍历：

```text
所有可用纸张尺寸
× 所有允许方向
× 所有可行拼法
```

每个方案都计算：

- 是否排得下
- 每张拼数
- 印刷张数
- 损耗张数
- 总用纸量
- 利用率
- 印刷成本
- 后道成本
- 综合成本
- 风险等级

---

## 5.4 每张拼数

```text
pieces_x = floor((printable_width + gap) / (item_width + gap))

pieces_y = floor((printable_height + gap) / (item_height + gap))

pieces_per_sheet = pieces_x × pieces_y
```

如果 `pieces_per_sheet <= 0`，该方案不可用。

---

## 5.5 印刷张数

```text
净印刷张数 = ceil(成品数量 / 每张拼数)

损耗张数 = ceil(净印刷张数 × 损耗率)

实际印刷张数 = 净印刷张数 + 损耗张数
```

---

## 5.6 纸张利用率

```text
产品占用面积 = 单个拼板占位宽度 × 单个拼板占位高度 × 每张拼数

纸张利用率 = 产品占用面积 ÷ 上机纸面积
```

---

# 6. 损耗逻辑

损耗应分工序计算，而不是只用一个固定比例。

## 6.1 常见损耗来源

```text
印刷损耗
覆膜损耗
模切损耗
压痕损耗
糊袋损耗
穿绳损耗
打扣损耗
包装损耗
运输损耗
```

---

## 6.2 建议基础损耗率

```yaml
普通纸卡: 3%-5%
复杂纸卡: 5%-8%
普通手提袋: 5%-8%
复杂手提袋: 8%-12%
高端手工袋: 10%-15%
```

---

## 6.3 增加损耗的因素

以下情况应增加损耗：

- 数量少
- 纸张贵
- 工艺复杂
- 深色满版
- 专色要求高
- 大面积烫金
- 大面积 UV
- 异形模切复杂
- 手工糊袋
- 手工穿绳
- 纸张较厚
- 覆膜后压痕
- 容易爆线
- 客户颜色要求高

---

# 7. 后道工艺影响

## 7.1 覆膜

覆膜成本通常按面积计算：

```text
覆膜面积 = 上机纸面积 × 实际印刷张数
```

需要判断：

- 单面覆膜还是双面覆膜
- 哑膜还是亮膜
- 是否防刮膜
- 是否触感膜
- 是否先覆膜后模切
- 是否覆膜后再压痕

---

## 7.2 模切 / 压痕

模切成本由以下部分组成：

```text
刀模费
+ 模切加工费
+ 清废人工费
```

影响因素：

- 刀线尺寸
- 刀线复杂度
- 模切张数
- 是否异形
- 是否需要压痕
- 是否容易清废
- 是否机器能做

---

## 7.3 烫金 / 局部 UV / 击凸

特殊工艺应判断：

- 工艺面积
- 位置是否靠边
- 是否需要单独版
- 是否可以共版
- 是否影响模切
- 是否影响压痕
- 是否需要二次定位

注意：面积小不一定便宜，如果位置分散或定位复杂，成本也可能高。

---

## 7.4 打孔 / 圆角

需要判断：

- 打孔数量
- 打孔位置
- 圆角半径
- 是否可用标准圆角刀
- 是否需要模切一起完成
- 是否需要单独人工处理

---

# 8. 综合成本判断

一个拼板方案的综合成本可以拆分为：

```text
综合成本 =
纸张成本
+ 印刷成本
+ 表面处理成本
+ 模切 / 压痕成本
+ 特殊工艺成本
+ 糊袋 / 糊盒成本
+ 穿绳 / 打扣成本
+ 包装成本
+ 损耗成本
+ 管理成本
```

推荐方案不一定是拼数最高的方案，而应该是：

```text
综合成本最低
且生产稳定
且风险最低
```

---

# 9. 输出字段

系统最终应该输出以下字段：

```yaml
recommended_paper_size: 推荐纸张尺寸
recommended_print_size: 推荐上机尺寸
product_flat_size: 产品展开尺寸
product_bleed_size: 出血后尺寸
pieces_per_sheet: 每张拼数
pieces_x: 横向拼数
pieces_y: 纵向拼数
orientation: 拼板方向
grain_direction_result: 纸纹判断结果

net_print_sheets: 净印刷张数
waste_sheets: 损耗张数
total_print_sheets: 总印刷张数
paper_utilization: 纸张利用率

printing_method: 印刷方式
surface_finish: 表面处理
die_cut_required: 是否模切
creasing_required: 是否压痕
gluing_required: 是否糊袋 / 糊盒
rope_required: 是否穿绳
eyelet_required: 是否打扣

paper_cost: 纸张成本
printing_cost: 印刷成本
finishing_cost: 后道成本
total_cost: 综合成本

risk_notes: 风险提示
production_notes: 生产说明
```

---

# 10. 伪代码

```python
def calculate_flat_size(product):
    if product.type == "paper_card":
        width = product.finished_width + product.bleed * 2
        height = product.finished_height + product.bleed * 2
        return width, height

    if product.type == "folded_card":
        width = product.unfolded_width + product.bleed * 2
        height = product.unfolded_height + product.bleed * 2
        return width, height

    if product.type == "paper_bag":
        flat_width = product.bag_width * 2 + product.gusset * 2 + product.glue_flap
        flat_height = product.bag_height + product.top_fold + product.bottom_fold

        width = flat_width + product.bleed * 2
        height = flat_height + product.bleed * 2

        return width, height


def calculate_printable_area(paper, margins):
    printable_width = paper.width - margins.left - margins.right
    printable_height = paper.height - margins.gripper - margins.tail
    return printable_width, printable_height


def calculate_pieces_per_sheet(item_width, item_height, printable_width, printable_height, gap):
    pieces_x = int((printable_width + gap) // (item_width + gap))
    pieces_y = int((printable_height + gap) // (item_height + gap))
    return pieces_x, pieces_y, pieces_x * pieces_y


def check_machine_feasibility(option, machines):
    if option.print_size_width > machines.print_max_width:
        return False

    if option.print_size_height > machines.print_max_height:
        return False

    if option.die_cut_required:
        if option.print_size_width > machines.die_cut_max_width:
            return False
        if option.print_size_height > machines.die_cut_max_height:
            return False

    if option.lamination_required:
        if option.print_size_width > machines.lamination_max_width:
            return False
        if option.print_size_height > machines.lamination_max_height:
            return False

    return True


def calculate_imposition_options(product, paper_sizes, machines, margins):
    options = []

    item_width, item_height = calculate_flat_size(product)

    for paper in paper_sizes:
        printable_width, printable_height = calculate_printable_area(paper, margins)

        orientations = [
            ("normal", item_width, item_height),
            ("rotated", item_height, item_width)
        ]

        for orientation, width, height in orientations:
            if product.grain_direction_required:
                if not grain_direction_matches(product, paper, orientation):
                    continue

            pieces_x, pieces_y, pieces_per_sheet = calculate_pieces_per_sheet(
                width,
                height,
                printable_width,
                printable_height,
                product.gap_between_items
            )

            if pieces_per_sheet <= 0:
                continue

            net_sheets = ceil(product.quantity / pieces_per_sheet)
            waste_rate = estimate_waste_rate(product)
            waste_sheets = ceil(net_sheets * waste_rate)
            total_sheets = net_sheets + waste_sheets

            utilization = (width * height * pieces_per_sheet) / (paper.width * paper.height)

            option = {
                "paper": paper,
                "orientation": orientation,
                "pieces_x": pieces_x,
                "pieces_y": pieces_y,
                "pieces_per_sheet": pieces_per_sheet,
                "net_sheets": net_sheets,
                "waste_sheets": waste_sheets,
                "total_sheets": total_sheets,
                "utilization": utilization,
            }

            if not check_machine_feasibility(option, machines):
                continue

            option["paper_cost"] = calculate_paper_cost(product, paper, total_sheets)
            option["printing_cost"] = calculate_printing_cost(product, total_sheets)
            option["finishing_cost"] = calculate_finishing_cost(product, total_sheets)
            option["total_cost"] = (
                option["paper_cost"]
                + option["printing_cost"]
                + option["finishing_cost"]
            )

            options.append(option)

    return sorted(options, key=lambda x: x["total_cost"])
```

---

# 11. 示例

## 11.1 纸卡示例

```yaml
product_type: paper_card
product_name: 吊牌
finished_width: 50
finished_height: 90
bleed: 3
paper_type: 白卡
paper_weight: 350gsm
printing: 4C + 4C
surface_finish: matte_lamination
die_cut: true
hole_punching: true
round_corner: true
quantity: 20000
```

系统逻辑：

```text
1. 计算出血后尺寸：
   56 × 96 mm

2. 遍历可用纸张尺寸。

3. 比较正常方向和旋转方向。

4. 计算每张拼数。

5. 计算印刷张数。

6. 加入覆膜、模切、打孔、圆角成本。

7. 判断是否有纸纹要求。

8. 输出综合成本最低方案。
```

---

## 11.2 手提袋示例

```yaml
product_type: paper_bag
product_name: 纸手提袋
bag_width: 250
bag_height: 320
gusset: 100
glue_flap: 20
top_fold: 40
bottom_fold: 80
bleed: 3
paper_type: 白卡
paper_weight: 250gsm
printing: 4C + 0
surface_finish: matte_lamination
die_cut: true
creasing: true
gluing: true
hole_punching: true
rope_required: true
eyelet_required: false
quantity: 5000
```

展开尺寸：

```text
展开宽度 = 250 × 2 + 100 × 2 + 20 = 720 mm

展开高度 = 320 + 40 + 80 = 440 mm

出血后尺寸 = 726 × 446 mm
```

系统逻辑：

```text
1. 计算展开尺寸。
2. 加出血。
3. 遍历可用纸张尺寸。
4. 判断是否排得下。
5. 判断纸纹方向。
6. 判断印刷机、覆膜机、模切机是否能处理。
7. 计算每张拼数。
8. 计算印刷张数和损耗张数。
9. 加入覆膜、模切、压痕、糊袋、穿绳、打扣成本。
10. 输出综合成本最低方案。
```

---

# 12. Codex 学习重点总结

Codex 在处理纸卡和手提袋拼板时，应遵循以下原则：

```text
1. 先计算产品展开尺寸。
2. 再加入出血和安全边。
3. 遍历所有纸张尺寸。
4. 同时测试正常方向和旋转方向。
5. 如果有纸纹要求，禁止错误方向。
6. 计算每张拼数。
7. 计算印刷张数和损耗。
8. 检查印刷机和后道设备限制。
9. 计算纸张利用率。
10. 计算纸张、印刷、后道和人工成本。
11. 不只选择拼数最高，而是选择综合成本最低且生产风险最低的方案。
```

---

# 13. 一句话总结

```text
纸卡拼板 = 平面尺寸优化 + 工艺匹配 + 合版判断

手提袋拼板 = 展开结构计算 + 纸纹限制 + 后道可生产性判断 + 人工成本判断
```
