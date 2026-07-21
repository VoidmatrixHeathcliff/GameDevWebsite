---
title: 程序化地形生成与侵蚀模拟
date: 2025-11-28
updated: 2025-11-28
permalink: articles/InH/GodotTerrainErosionStory/
categories: InH
tags: ["Godot", "GDScript", "FastNoiseLite", "SurfaceTool", "GridMap", "程序化地形", "侵蚀模拟"]
---

<div align="center">

![方块地形填充示例](/articles/InH/GodotTerrainErosionStory/1.png)

</div>


本文我将着重介绍根据柏林噪声程序化生成地图并实现类似我的世界地图的方块填充策略以及通过侵蚀模拟对噪声地形进行更细致的雕刻。

最开始我想学习地形生成的理由很简单，就是复刻一个简易的《我的世界》，真正开始探索后问题很快变成了数据组织。地形可以用连续网格展示，也可以离散成方块，还可以交给侵蚀算法改写，每个阶段的表现形式都在变，但万变不离其宗的就是高度数据。

<!-- More -->

## 什么是柏林噪声

<div align="center">

![柏林噪声示例图](/articles/InH/GodotTerrainErosionStory/2.png)

</div>

柏林噪声更准确地说是 Perlin Noise，它是一种连续的梯度噪声。普通随机数每个点之间没有关系，直接拿来生成地形会变成一堆杂乱的尖峰；柏林噪声的特点是相邻位置的值会平滑过渡，所以更适合用来描述山坡、丘陵、云雾这类自然形态。

在地形生成里，可以把柏林噪声理解成一个函数：输入二维坐标 `(x, z)`，输出一个大致位于 `-1` 到 `1` 之间的值。这个值本身还不是地形，只是高度的原始来源。把它乘上高度系数，就能得到某个顶点或某一列方块的高度。

```gdscript
var noise_val = noise.get_noise_2d(x, z)
var height = noise_val * noise_strength
```

这种方式能保证地形是连续起伏的，而不是每个格子完全独立随机。



## 从柏林噪声到 fBM

<div align="center">

![fBM示例图](/articles/InH/GodotTerrainErosionStory/6.png)

</div>

单层柏林噪声已经可以生成连续起伏的高度，但尺度比较单一。自然地形往往是多层尺度共同作用的结果，大轮廓决定山脉走向，小尺度噪声补充坡面细节。fBM 的作用就是把多层不同频率、不同强度的噪声叠在一起。

在 Godot 的 `FastNoiseLite` 里，不需要自己手写多次采样，可以直接打开分形噪声：

```gdscript
noise.noise_type = FastNoiseLite.TYPE_PERLIN
noise.frequency = noise_frequency
noise.fractal_type = FastNoiseLite.FRACTAL_FBM
noise.fractal_octaves = 4
noise.fractal_lacunarity = 2.0
noise.fractal_gain = 0.5
```

这里可以把 `fractal_octaves` 理解成叠加层数，层数越多细节越丰富；`fractal_lacunarity` 控制每一层频率放大的速度，决定细节尺度缩小得有多快；`fractal_gain` 控制每一层振幅衰减的比例，决定后续细节对最终高度的影响有多明显。

所以 fBM 是在柏林噪声的基础上补充层，后面无论是连续网格、方块填充还是侵蚀模拟，入口仍然都是高度数据；只是这个高度不再来自一层噪声，而是来自多层噪声叠加后的结果。



## 核心数据：地形高度数据

首先，噪声使用 `FastNoiseLite`，平面网格用 `PlaneMesh` 创建，随后通过 `SurfaceTool` 提取顶点和索引。但如果每次生成都重新理解一遍网格拓扑，后面做动画和材质同步会很乱。后面我把基础网格和目标高度拆开保存，`base_vertices` 表示原始平面，`target_heights` 表示每个顶点最终应该到达的高度。

```gdscript
var base_vertices: PackedVector3Array
var base_indices: PackedInt32Array
var target_heights: PackedFloat32Array

func prepare_mesh_data():
    # 首次生成时缓存基础网格拓扑，后面只更新高度。
    if base_vertices.is_empty():
        var arrs = _extract_plane_arrays()
        base_vertices = arrs[ArrayMesh.ARRAY_VERTEX]
        base_indices = arrs[ArrayMesh.ARRAY_INDEX]

    target_heights.clear()
    for v in base_vertices:
        var h = noise.get_noise_2d(v.x, v.z) * noise_strength
        target_heights.append(h)
```

这个结构让生长动画只需要从 0 插值到目标高度，Shader 分层只需要读取最新的 `min_height` 和 `max_height`，重建网格时索引也可以复用。

```gdscript
func apply_growth(t: float):
    var st = SurfaceTool.new()
    st.begin(Mesh.PRIMITIVE_TRIANGLES)

    for i in range(base_vertices.size()):
        var v = base_vertices[i]
        v.y = target_heights[i] * t
        st.add_vertex(v)

    for i in base_indices:
        st.add_index(i)

    st.generate_normals()
    self.mesh = st.commit()

    if shader_mat and self.mesh.get_surface_count() > 0:
        self.set_surface_override_material(0, shader_mat)
```

只需不断刷新表现层，数据层保持不变，后面加方块、侵蚀、材质都能接在高度数据后面。



## 从网格地形到方块地形

<div align="center">

![裸方块地形示例](/articles/InH/GodotTerrainErosionStory/3.png)

</div>

连续网格适合看地形起伏，方块地形更适合看规则分层。所以我把噪声高度转成了整型高度，再用 `Dictionary` 保存方块坐标。

这里的关键点是 `block_data`。它用 `Vector3i` 作为 key，代表某个方块是否存在。生成阶段先算每一列高度，再把从 `min_height` 到 `column_height` 的方块填进去。

```gdscript
var block_data: Dictionary = {}

func generate_new_terrain():
    block_data.clear()

    var height_map: Dictionary = {}
    for x in range(int(model_size.x)):
        for z in range(int(model_size.y)):
            var height_noise = noise.get_noise_2d(x, z)
            var normalized_height = (height_noise + 1.0) / 2.0
            var h = int(round(normalized_height * noise_strength))

            height_map[Vector2i(x, z)] = h
            min_height = min(min_height, h)
            max_height = max(max_height, h)

    for pos_2d in height_map:
        var column_height = height_map[pos_2d]
        for y in range(min_height, column_height):
            block_data[Vector3i(pos_2d.x, y, pos_2d.y)] = 1
```

这里会出现一个性能问题，方块地形如果把每个方块六个面都画出来，内部面会占掉大量三角形。实际能看到的只有外露面，所以我在生成 mesh 时检查六个方向的邻居，只给空邻居或者还没有长出来的邻居生成面。

```gdscript
func apply_growth(t: float):
    var current_max_y = lerp(min_height, max_height, t)
    var st = SurfaceTool.new()
    var vertex_idx := 0
    var directions = [
        Vector3i.UP, Vector3i.DOWN, Vector3i.LEFT,
        Vector3i.RIGHT, Vector3i.FORWARD, Vector3i.BACK
    ]

    for pos in block_data.keys():
        if pos.y > current_max_y:
            continue

        for dir in directions:
            var neighbor_pos = pos + dir
            if not block_data.has(neighbor_pos) or neighbor_pos.y > current_max_y:
                add_cube_face(st, pos, dir, vertex_idx)
                vertex_idx += 4
```

方块世界的规模上来以后，这个朴素的处理会带来很明显的性能提升，效果显著。



## GridMap 把地形变成可编辑世界

<div align="center">

![方块地形填充示例](/articles/InH/GodotTerrainErosionStory/1.png)

</div>

我生成了多套 `MeshLibrary`，分别给地形、水、树、结构物使用。这样做以后，地形生成从一张表面扩展成可以直接摆草地、沙地、石块、水面、树和房屋的世界生成流程。

```gdscript
func _ready() -> void:
    var height_map := _generate_height_map(noise)

    _populate_terrain(height_map, grid_terrain, grid_water)
    _place_trees(height_map, grid_terrain, grid_trees)
    _place_village(height_map, grid_terrain, grid_trees, grid_structures)
    _place_boats(height_map, grid_structures)
```

这里的思路是把世界分层。地形层负责基础块，水层负责补齐低洼区域，树和结构物各用自己的 `GridMap`。这样清理和重新生成会简单很多，调试时也能直接隐藏某一层观察问题。

高度图仍然是入口。`_generate_height_map()` 把噪声采样转换成以地图中心为原点的坐标，后面的摆放逻辑只看高度和顶层材质。

```gdscript
func _generate_height_map(noise: FastNoiseLite) -> Dictionary:
    var height_map: Dictionary = {}
    var offset: Vector2i = map_size / 2

    for x in range(map_size.x):
        for z in range(map_size.y):
            var noise_val: float = noise.get_noise_2d(x, z)
            var normalized_height: float = (noise_val + 1.0) / 2.0
            var centered_pos := Vector2i(x - offset.x, z - offset.y)
            height_map[centered_pos] = int(round(normalized_height * noise_strength))

    return height_map
```

这里遇到的主要问题就是“地形生成”和“地物生成”会互相影响。比如树先生成，村庄区域又要清树；水面低洼位置可以放船，普通地面就不该放。最后我把规则集中到放置函数里，让每个函数只处理一种资源。

```gdscript
var h: int = height_map[pos_centered]
var terrain_pos := Vector3i(pos_centered.x, h, pos_centered.y)
var top_terrain_id: int = grid_terrain.get_cell_item(terrain_pos)

if top_terrain_id >= Terrain.GRASS_1 and top_terrain_id <= Terrain.GRASS_6:
    var structure_pos := terrain_pos + Vector3i.UP
    grid_trees.set_cell_item(structure_pos, GridMap.INVALID_CELL_ITEM)
    grid_structures.set_cell_item(structure_pos, Structure.HOUSE)
```

为了避免生成规则分散，拆成高度图、地形、水、植被、结构物以后，问题容易定位。



## 热力侵蚀修饰地形

<div align="center">

![热力侵蚀示例](/articles/InH/GodotTerrainErosionStory/4.png)

</div>

噪声地形的问题是它有高度变化，但很多坡面缺少自然搬运的痕迹。热力侵蚀的目标是让过陡的坡面缓一点：相邻点高度差超过临界坡度，就从高处向低处搬一点高度。

我当时最在意的是迭代稳定性。直接边遍历边改 `height_map` 会让前面改过的点影响后面的点，同一轮迭代里会出现明显方向性。现在的做法是每轮先写 `delta_map`，整轮算完再统一应用。

```gdscript
func apply_thermal_erosion(height_map: Array[PackedFloat32Array]):
    for i in range(thermal_iterations):
        var delta_map := create_empty_delta_map()

        for z in range(depth):
            for x in range(width):
                for neighbor in [Vector2i(x + 1, z), Vector2i(x, z + 1)]:
                    if not in_height_map(neighbor):
                        continue

                    var diff = height_map[z][x] - height_map[neighbor.y][neighbor.x]
                    if abs(diff) <= thermal_talus_angle:
                        continue

                    var amount = (abs(diff) - thermal_talus_angle) * 0.5 * thermal_strength
                    var flow = sign(diff) * amount

                    delta_map[z][x] -= flow
                    delta_map[neighbor.y][neighbor.x] += flow

        for z in range(depth):
            for x in range(width):
                height_map[z][x] += delta_map[z][x]
```

每一轮计算基于旧高度，这个细节处理好以后，迭代出来的坡面更均匀，调 `thermal_iterations`、`thermal_talus_angle`、`thermal_strength` 时也更可预期。



## 水力侵蚀处理沟壑和沉积

<div align="center">

![水力侵蚀示例](/articles/InH/GodotTerrainErosionStory/5.png)

</div>

水力侵蚀比热力侵蚀更像一个小模拟。代码里每个雨滴都有位置、方向、速度、水量和泥沙量。雨滴按地形梯度移动，低处会获得携沙能力，高处或者容量不足时发生沉积。我这里用了 `erosion_iterations = 80000`，单个雨滴最多走 `max_steps = 64`。

```gdscript
if border_size > 0:
    for z in depth:
        for x in width:
            var dist_x = min(x, width - 1 - x)
            var dist_z = min(z, depth - 1 - z)
            var dist_to_edge = min(dist_x, dist_z)

            if dist_to_edge < border_size:
                height_map[z][x] *= pow(float(dist_to_edge) / border_size, border_falloff_strength)
```

雨滴移动时，雨滴在高度图上是连续坐标，所以采样高度和回写侵蚀量都需要双线性处理。否则地表会出现颗粒感很强的格点坑。

```gdscript
func apply_erosion(height_map: Array[PackedFloat32Array]):
    for i in range(erosion_iterations):
        var pos = Vector2(rng.randf_range(0, width - 1), rng.randf_range(0, depth - 1))
        var dir = Vector2.ZERO
        var speed = 1.0
        var water = 1.0
        var sediment = 0.0

        for step in range(max_steps):
            if not in_height_map(pos):
                break

            var current_height = get_interpolated_height(pos, height_map)
            var gradient = get_gradient(pos, height_map)

            dir = (dir * inertia - gradient * (1.0 - inertia)).normalized()

            var old_pos = pos
            pos += dir

            var new_height = get_interpolated_height(pos, height_map)
            var delta_height = new_height - current_height
            var sediment_capacity = max(-delta_height, 0.01) * speed * water * sediment_capacity_factor

            if sediment > sediment_capacity or delta_height > 0:
                var amount_to_deposit = (sediment - sediment_capacity) * deposition_speed
                if delta_height > 0:
                    amount_to_deposit = min(delta_height, sediment)
                sediment -= amount_to_deposit
                deposit_at(old_pos, amount_to_deposit, height_map)
            else:
                var amount_to_erode = min((sediment_capacity - sediment), -delta_height) * erosion_strength
                sediment += amount_to_erode
                erode_at(old_pos, -amount_to_erode, height_map)

            speed = sqrt(max(0.0, speed * speed + delta_height * -4.0))
            water *= 1.0 - evaporation_rate
```

回写阶段把一次沉积或侵蚀分摊给周围四个格点。这是水力侵蚀里比较容易被忽略的部分，视觉上差别很大。

```gdscript
func deposit_at(pos: Vector2, amount: float, height_map: Array[PackedFloat32Array]):
    var cell_x = int(pos.x)
    var cell_z = int(pos.y)
    var offset = Vector2(pos.x - cell_x, pos.y - cell_z)

    height_map[cell_z][cell_x] += amount * (1 - offset.x) * (1 - offset.y)
    height_map[cell_z][cell_x + 1] += amount * offset.x * (1 - offset.y)
    height_map[cell_z + 1][cell_x] += amount * (1 - offset.x) * offset.y
    height_map[cell_z + 1][cell_x + 1] += amount * offset.x * offset.y
```

最后就会得到山峰沟壑分明的水力侵蚀模拟地形

## 总结

这个专题展示了 `FastNoiseLite`、`SurfaceTool`、`GridMap`、`MeshLibrary`、热力侵蚀、水力侵蚀，看起来分散，实际都围绕高度数据展开。

程序化生成不能只追求随机。随机只提供输入，后面必须有稳定的数据结构、可调参数、可观察的中间状态和可以复用的输出层。这样地形从连续网格切到方块世界，或者从普通噪声切到侵蚀结果时，代码还能保持清楚。
