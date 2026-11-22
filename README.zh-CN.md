粤港澳大湾区旅游 POI 数据集
[English](README.md) | [简体中文](README.zh-CN.md)

本数据集收集粤港澳大湾区 11 个城市的旅游相关 POI（Point of Interest，兴趣点），包含景点名称、地址、经纬度、分类、采集来源信息，以及保留在 JSON 中的景点介绍、图片链接和部分票务属性。

重要说明： 本数据集是通过特定采集策略得到的样本，不代表大湾区全部景点。当前数据存在已确认的城市归属误判，且包含第三方平台返回的内容。使用或公开发布前，请阅读数据质量与已知问题及数据来源与公开许可。
数据集概览
项目	统计结果
数据文件	[`data/pois.csv`](data/pois.csv)
POI 记录数	3,597
唯一 poi_key 数量	3,597
覆盖城市	11
CSV 字段	12
非空分类标签数	35
category 为空	1,101 条（30.61%）
first_seen_at 范围	2026-09-30 至 2026-10-16（UTC+08:00）
文件大小	4,335,888 字节（约 4.34 MB）
编码	UTF-8 with BOM（utf-8-sig）


first_seen_at 表示该记录首次写入采集数据库的时间，不等于景点创建时间、实地核查时间或来源平台最近更新时间。
各城市数据量
城市	POI 数量
广州	859
深圳	445
中山	368
惠州	342
东莞	304
江门	291
佛山	259
珠海	254
肇庆	219
香港	151
澳门	105
合计	3,597


以上数量是 CSV 记录数，并非各城市实际景点总数。其中，中山的数据包含已确认的跨地区误采集记录，不能直接用于比较真实景点密度。
字段说明
字段	含义	示例或补充说明
poi_key	数据集内部唯一标识，由来源前缀及平台 ID 组合	flyai:67058
platform_id	来源平台的 POI ID	67058
city	采集或导入时赋予的城市标签，可能有误	广州
name	景点或兴趣点名称	流溪河国家森林公园
address	来源平台返回的地点地址，入库时会去除首尾空白	中文地址
latitude	纬度	23.74822
longitude	经度	113.782528
category	来源平台的旅游 POI 分类，可能为空	森林丛林
location_status	历史采集流程生成的城市归属校验标签	city_confirmed、`mismatch`、`unknown`
first_source	首次入库的采集途径或历史导入来源	search-poi、`v7_baseline`
first_seen_at	该 POI 首次进入采集数据库的时间	带时区的 ISO 8601 时间
raw_json	保留原始单条 POI 信息的 JSON 字符串	可解析为 JSON 对象


坐标系注意事项： 当前尚未独立核实来源坐标使用 WGS84、GCJ-02 或其他坐标参考系统。进行精确距离计算、地图叠加或导航前，应先验证坐标系。
raw_json 的扩展属性
JSON 对象可能包含 id、`name`、`address`、`latitude`、`longitude`、`category`、`description`（景点介绍）、`mainPic`（主图 URL）、`jumpUrl`（详情链接）、`freePoiStatus`（免费状态）、`poiLevel`（景点等级字段）、`listRank`（榜单描述）和 ticketInfo（部分票务信息）。历史导入记录还可能保留 district、`source`、`sourceTool`、`sourceEndpoint`、`occurrenceCount`、`matchedQueries` 和 retrievedAt。

上述属性并非每条记录都有有效值；景点排名、票价、状态及链接可能已发生变化。图片只以 URL 形式提供，不包含图片文件本身。
适用场景
该数据集可用于旅游景点检索与推荐、旅游规划 Agent 的候选 POI 知识库、RAG 语义检索、地图可视化、基于地理空间数据库的探索性分析，以及 POI 去重、实体消歧、异常位置检测等数据质量研究。

目前不适合直接用作高精度导航、实时票价查询或完整景点覆盖率评估的真值数据，相关任务需要额外验证或接入实时服务。
Python 读取示例
import json
import pandas as pd

pois = pd.read_csv("data/pois.csv", encoding="utf-8-sig")

# 示例：查看标记为城市归属已确认的广州博物馆。
subset = pois.loc[
    (pois["city"] == "广州")
    & (pois["category"] == "博物馆")
    & (pois["location_status"] == "city_confirmed"),
    ["poi_key", "name", "address", "latitude", "longitude"],
]
print(subset.head())

# 从原始 JSON 中提取景点介绍。
pois["description"] = pois["raw_json"].apply(
    lambda value: json.loads(value).get("description")
)
注意：`city_confirmed` 并不代表地理位置绝对正确。地图或区域查询任务仍需额外核验。
数据质量与已知问题
当前 CSV 中历史位置校验状态的统计结果如下：
location_status	记录数	占比
city_confirmed	3,541	98.44%
mismatch	44	1.22%
unknown	12	0.33%


已知问题包括：
1. 城市校验存在误判。 至少 34 条标记为 中山 的记录，其坐标实际位于远离大湾区的大连一带，却被标记为 city_confirmed。例如，`大连港` 的地址位于辽宁省大连市中山区，但 CSV 中 city=中山；另有 1 条同类地区之外的记录被标记为 mismatch。中山市与中山区的地名相似性可能导致搜索结果混入，但具体校验规则成因尚未确认。
2. 部分分类缺失。 有 1,101 条记录的 category 为空，不应将空值视为人工确认的景点类别。
3. 历史质量标签不等于真值。 city_confirmed 只能反映当时规则的判断结果，不能保证行政归属、坐标和实体身份正确。
4. 去重粒度有限。 poi_key 均不重复，但不同平台 ID 仍可能指向同一实际地点，尚未完成名称与空间实体的全量消歧。
5. 样本覆盖存在偏差。 城市和类别分布受采集查询策略、接口返回及配额限制影响，不代表真实景点分布。
6. 信息具有时效性。 景点介绍、图片地址、排名、票价及服务状态均为历史记录，不保证实时有效。
用于空间分析时，建议增加城市边界多边形或坐标范围检查、行政地址解析和异常样本人工复核。清洗后应保留原始数据及修改记录，避免覆盖来源字段。
数据来源与复现说明
本数据集来自与 FlyAI / 飞猪相关的旅游 POI 采集流程。当前文件中：
first_source	数量	含义
search-poi	2,914	通过结构化 POI 搜索流程入库
v7_baseline	683	从先前采集的基线数据导入


配套 crawler.py 的主要逻辑包括：构建查询任务、调用 FlyAI CLI 的 search-poi、读取 data.itemList、规范化 POI 字段、写入 SQLite、按照平台 ID 去重、调用质量检查模块，以及导出 CSV。

需要说明的是：单独一份 crawler.py 不能独立复现完整数据。 运行依赖 config.json、多个同级 Python 模块、查询输入、具备调用权限的 FlyAI 环境，以及历史基线数据。上游接口的返回内容可能变化，即使环境齐全，也不保证重新获得完全一致的 3,597 条记录。

官方参考资料：
- 飞猪 AI 开放平台文档
- FlyAI search-poi 命令参数文档
- FlyAI 官方 GitHub 仓库
数据来源与公开许可
本数据集包含第三方平台返回的景点介绍、链接和元数据，部分记录涉及商业图片和票务链接。FlyAI 工具代码开源，并不意味着其返回的 POI 内容也获得了公开再分发许可。 当前 README 并未确认该 CSV 可被公开复制、分发或用于商业训练。

在将完整 pois.csv 上传至公开 GitHub 仓库之前，应确认来源平台/API 的使用条款和数据再分发授权。如无法确认，建议仅公开经过授权的数据或不包含受限原始内容的派生数据，并将完整原始文件保存在私有仓库。这里不对 CSV 数据声明任何开放数据许可证。
引用说明
如将该数据用于论文或研究报告，建议注明数据属于基于 FlyAI 相关采集流程整理的粤港澳大湾区旅游 POI 历史样本，写明使用版本/访问日期，并披露预处理中的过滤、纠错和去重操作。目前尚未为本数据集分配 DOI 或正式引用标识。
